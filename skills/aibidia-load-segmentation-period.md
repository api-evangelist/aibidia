---
name: aibidia-load-segmentation-period
description: >-
  Upload one reporting period (a year and a month) of multi-level segmentation data from an ERP extract into
  Aibidia's OTP Management solution via the Public OTP Management API, then confirm it landed.
api: Aibidia Public OTP Management API
base_url: https://otpm-api.aibidia.com
operations:
  - POST /api/datainjections/automated
  - GET /api/extract-types/integration
generated: '2026-09-13'
method: generated
source: >-
  openapi/aibidia-otp-management-public-openapi.yml, fetched verbatim from
  https://otpm-api.aibidia.com/swagger/public/swagger.json. The served document declares no operationIds, so the
  two operations are referenced here by method and path — the only stable names the provider publishes.
  overlays/aibidia-otp-management-public-overlay.yaml proposes injectAutomatedSegmentationData and
  getExtractTypeIntegration as stable ids.
---

# Load a segmentation period into Aibidia OTP Management

## Before you start

Read these first — they change how you must sequence the work:

- **There is no idempotency mechanism.** No `Idempotency-Key`, no client request id, no conditional header.
  If a request times out you cannot tell whether it landed.
- **There is no reversal operation.** Nothing cancels, deletes or voids an injected extract. Get it right or
  re-upload the whole period.
- **Pages must arrive in consecutive order**, and Aibidia's own instruction on any failure is to
  **restart from `currentPage: 0`** — the entire submission, not the failed page.
- **The recommended rate is 1 request per second.** At the maximum 2000 pages that is roughly 33 minutes for a
  full-size period, all of which is lost on a mid-run failure.

Because of the first three points, do not hand this flow to an unsupervised agent without an external replay
guard that records which periods have already been submitted.

## Authentication

Every request carries a single header:

```
X-DATAINGESTION-API-KEY: <key issued by Aibidia>
```

The key is bound to **one Extract Type**. That binding is the whole scope model — you never name an Extract Type
in a request, so using the wrong key silently targets the wrong configuration (or returns `403`). Keys are
issued by Aibidia to an existing customer; there is no self-serve issuance endpoint.

## Step 1 — Read the target configuration

```
GET https://otpm-api.aibidia.com/api/extract-types/integration
X-DATAINGESTION-API-KEY: <key>
```

Returns `ExtractTypeIntegrationDto`: the Extract Type's `id` and `name`, its `columnMappings[]`, and its
existing `extracts[]`.

Do two things with the response before you send anything:

1. **Map your source columns.** Each `columnMappings[]` entry gives your customer's own header
   (`originalName`), the Aibidia canonical column it maps to (`extractTypeColumn`, one of 48 values),
   `columnType`, `defaultMaxLength`, `isAlwaysRequired`, and `requiredOnlyIfTheseColsAreNull[]` — the
   conditional-requirement rule. Honour `defaultMaxLength`: over-length strings are a `400`.
2. **Check for an existing extract for this period.** Scan `extracts[]` for a matching `reportingYear` and
   `month`. This is your only defence against a duplicate upload, since the write itself is not idempotent.
   Note `uploadState` (`Created`, `Initiated`, `Completed`, `Failed`) and `validationStatus` (`Created`,
   `Pending`, `Processing`, `Completed`, `Failed`) — they are independent state machines.

## Step 2 — Page the data

Split your rows into pages of **at most 5000 items**, and at most **2000 pages** (10,000,000 rows per period).

Each request body:

```json
{
  "year": 2026,
  "month": 9,
  "pages": 12,
  "currentPage": 0,
  "data": [ { /* InputAutomatedMultiLevelSegmentationInjectionItemDto */ } ]
}
```

- `pages` is the **total** page count for this submission (1–2000) and must be identical on every page.
- `currentPage` starts at `0` and increments by one; pages must be sent consecutively.
- `month` accepts **1–13**. `13` is the adjustment/closing period an ERP posts after month 12, not a calendar
  month — send it only when your ledger actually has one.

Each item in `data` requires all of: `account`, `accountDescription`, `legalEntity`, `counterpartyLegalEntity`,
`groupValue`, `localValue`, `taxJurisdiction`, `division`, `businessUnit`, `functionalArea`, `department`,
`profitCenter`, `costCenter`, `productLevel1` through `productLevel5`, and `projectCode`. Most are nullable —
required means the key must be present, not that it must be non-null. `account` (max 255) and `legalEntity`
(max 50) are the two that must be non-empty. `groupValue` and `localValue` are doubles: the group-currency and
local-currency amounts for the line.

`legalEntity` and `counterpartyLegalEntity` are the customer's own entity codes, not Aibidia identifiers, and
there is no lookup operation to validate them against before sending.

Every schema sets `additionalProperties: false` — an extra field is a `400`, not a silently ignored one.

## Step 3 — Send

```
POST https://otpm-api.aibidia.com/api/datainjections/automated
Content-Type: application/json
X-DATAINGESTION-API-KEY: <key>
```

Pace at roughly **1 request per second**. A `200` returns `AutomatedDataInjectionResponse` with `extractId`;
record it — it is the only handle you get on what you just created.

## Step 4 — Handle failures

Errors are RFC 7807 `ProblemDetails` (`type`, `title`, `status`, `detail`, `instance`), served as
`application/json` rather than `application/problem+json`, so match on HTTP status, not on media type. No
problem `type` URIs are published.

| Status | Meaning | What to do |
|---|---|---|
| `400` | Payload failed validation — out-of-range `pages`/`month`, over 5000 items, a missing required key, an over-length string, or an unknown property | Fix the payload, then **restart the submission from `currentPage: 0`** |
| `401` | Missing or invalid `X-DATAINGESTION-API-KEY` | Fix the header; do not retry blind |
| `403` | Key is valid but not permitted for this target | You are using the key for a different Extract Type |
| `500` | Server-side failure (no body schema declared) | Retry with backoff — but see the warning below |

**The `500` retry is the dangerous one.** With no idempotency key you cannot distinguish "the page never landed"
from "the page landed and the response was lost". Aibidia's published recovery is to restart from
`currentPage: 0`, which means re-sending every page of the period.

No `429` is declared, so do not expect a rate-limit signal — there are no `X-RateLimit-*` or `Retry-After`
headers on this API. The 1 req/s figure is a recommendation you obey blind.

## Step 5 — Confirm

Call `GET /api/extract-types/integration` again and find the extract whose `id` matches the `extractId` you
recorded. Upload is done when `uploadState` is `Completed`; the data is *accepted* only when
`validationStatus` reaches `Completed`. `validationStatus: Failed` means the platform rejected the content after
ingesting it — and there is no operation to withdraw it. Poll rather than wait on a callback: this API publishes
no webhook and no event contract.
