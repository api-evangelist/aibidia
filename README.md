# Aibidia

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Aibidia is a Helsinki-based transfer pricing technology company. Its cloud platform lets multinational
enterprises manage transfer pricing policy, intercompany transaction data, documentation and country-by-country
reporting across a set of solution modules — TPDoc, CbCR, OTP Management, Strategic TP Management, Value Chain
Analysis, Data Studio, Horizon and TP AI — behind a single Azure AD B2C sign-in at platform.aibidia.com.

**What this profile found (enrichment pass 2026-09-13).** Aibidia publishes no developer portal and no API
reference, and its marketing site's full 149-URL sitemap contains nothing developer-facing. The machine-readable
contracts were recovered from the provider's own published platform runtime configuration
(`https://platform.aibidia.com/env.js` and the per-solution `env.js` files), which names every backend service
host:

- **Public OTP Management API** — `https://otpm-api.aibidia.com/swagger/public/swagger.json`, OpenAPI 3.1.1, a
  deliberately-public two-operation ERP data-ingestion surface authenticated with an `X-DATAINGESTION-API-KEY`
  header. This is a real integration API, not a leak: the Swagger UI declares exactly one document group and
  names it "Public".
- **Aibidia TP AI API** — `https://tpai-api.aibidia.com/openapi.json`, OpenAPI 3.1.0, the session service behind
  the TP AI solution.
- A third published document, the **Public Horizon API** at `https://hzn-api.aibidia.com/swagger/public/swagger.json`,
  is real but declares zero operations. It is saved under `openapi/_original/` as evidence and deliberately not
  registered as an API.

No MCP server, no A2A agent card, no client SDKs in any registry, no public pricing, and no status page. Aibidia
does publish an `llms.txt`, an OpenID Provider Metadata document on its own `auth.aibidia.com` custom domain, and
ISO 27001 / SOC 2 Type 2 claims on its site. Every probe and its HTTP status is recorded in the artifacts.

- https://www.aibidia.com/
- https://equityzen.com/company/aibidia
