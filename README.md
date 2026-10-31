# Quandela

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

Quandela is a Paris-region (Massy, France) photonic quantum computing company building modular, scalable, energy-efficient quantum systems driven by deterministic single-photon sources. Its hardware lineup includes Ascella (its first cloud-accessible system), Belenos (a 12-qubit second-generation machine), Canopus, and Mosaiq (its flagship prototyping platform), alongside Prometheus single-photon sources and the Entropy quantum random number generator. Quandela develops Perceval, the open-source Python framework for programming photonic quantum computers, distributed on PyPI as perceval-quandela with the perceval-interop bridges to Qiskit, QuTiP, cQASM and myQLM and the exqalibur native optimisation kernel. It operates Quandela Cloud, which — contrary to an earlier reading of this profile — DOES publish a real, anonymously readable REST contract: OpenAPI 3.0.3 at https://api.cloud.quandela.com/openapi.json, version v2.8.0-rc4, with 55 paths, 59 operations and 54 component schemas, plus a Quantum Toolbox sub-specification at /qt-openapi.json. The surface covers Perceval job submission (submit-then-poll, with the circuit carried as an SDK-serialised opaque payload), a self-service Cloud Job Token lifecycle with a per-platform credit ledger, five typed Quantum Toolbox primitives (Chemistry VQE, Custom VQE, CVaR VQE, Graph DSI, Graph Isomorphism) each with a paired cost estimator, and the Entropy QRNG, whose draws return CHSH and min-entropy certification. Authentication is a single declared bearer scheme covering two distinct credentials — an account token from account.quandela.com and a Cloud Job Token minted over the API. Quandela also runs a Quantum Acceleration Program for enterprise pilots in cybersecurity, pharma, chemistry, logistics, finance, energy and aerospace, and publishes a 2024-2030 roadmap to fault-tolerant quantum computing.

**Areas:** Quantum Computing · Photonic Quantum · Photonics · Single Photon Sources · Quantum Hardware · Quantum Cloud · QPU · Perceval · Python SDK · Quantum Random Number Generation · Quantum Simulation · Variational Quantum Algorithms

## Quandela on APIs.io

Everything profiled for Quandela is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/quandela/](https://apis.io/providers/quandela/)** — or go straight to one of the APIs or properties below.

### APIs (9)

- [Quandela Api - Job Token API](https://apis.io/apis/quandela/quandela-api-job-token-api/) — APIs related to job token
- [Quandela Api - Perceval Job API](https://apis.io/apis/quandela/quandela-api-perceval-job-api/) — Operations about job - Authenticate by `Cloud Job Token`
- [Quandela Api - Perceval Job Token API](https://apis.io/apis/quandela/quandela-api-perceval-job-token-api/) — Operations about job token - Authenticate by `Cloud Job Token`
- [Quandela QRNG API](https://apis.io/apis/quandela/quandela-qrng-api/) — Quantum Random Number generation
- [Quandela Quantum Toolbox API](https://apis.io/apis/quandela/quandela-quantumtoolbox-api/) — Quantum Toolbox entrypoints
- [Quandela Health API](https://apis.io/apis/quandela/quandela-health-api/) — The Health API from Quandela — 1 operation(s) for health.
- [Quandela Openapi.json API](https://apis.io/apis/quandela/quandela-openapi-json-api/) — The Openapi.json API from Quandela — 1 operation(s) for openapi.json.
- [Quandela Qt Openapi.json API](https://apis.io/apis/quandela/quandela-qt-openapi-json-api/) — The Qt Openapi.json API from Quandela — 1 operation(s) for qt openapi.json.
- [Quandela Quandela Cloud API](https://apis.io/apis/quandela/quandela-quandela-cloud-api-api/) — The Quandela Cloud Api API from Quandela — 1 operation(s) for quandela cloud api.

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### Arazzo workflows (3)

Multi-step workflows that chain operations into a business task.

- [Estimate then run a Chemistry VQE workload on Quandela Quantum Toolbox](https://apis.io/arazzos/quandela/quandela-estimate-and-run-chemistry-vqe/)
- [Rotate and audit a Quandela Cloud Job Token](https://apis.io/arazzos/quandela/quandela-rotate-and-audit-job-token/)
- [Submit a Perceval job to Quandela Cloud and collect the result](https://apis.io/arazzos/quandela/quandela-submit-and-collect-perceval-job/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Quandela Agentic Access](https://apis.io/agentic-access/quandela/quandela-agentic-access/)

#### Overlays (1)

OpenAPI overlays, including conversational phrasing for each operation.

- [Quandela Cloud Overlay](https://apis.io/overlays/quandela/quandela-cloud-overlay/)

#### Security (2)

Authentication, trust centre, and vulnerability disclosure.

- [Quandela Authentication](https://apis.io/security/quandela/quandela-authentication/)
- [Quandela Domain Security](https://apis.io/security/quandela/quandela-domain-security/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Quandela Plans Pricing](https://apis.io/plans/quandela/quandela-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Quandela Rate Limits](https://apis.io/rate-limits/quandela/quandela-rate-limits/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://www.quandela.com)
- [Documentation](https://perceval.quandela.net/docs)
- [Getting started](https://perceval.quandela.net/docs/v1.2/getting_started.html)
- [Pricing](https://cloud.quandela.com/pricing)
- [Support](https://community.quandela.com)
- [Changelog](https://github.com/Quandela/Perceval/releases)
- [Blog](https://www.quandela.com/resources/blog/)
- [Terms of service](https://www.quandela.com/legal-terms/)
- [Privacy policy](https://www.quandela.com/privacy-policy/)
- [GitHub](https://github.com/Quandela)
- [Status](https://api.cloud.quandela.com/api/platforms/public)

## API Evangelist rating

**Kin Score 47.2** — developing · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 71.4 |
| Contract quality | 47.3 |
| Contract governance | 4.5 |
| Operational transparency | 71.1 |
| Developer ergonomics | 61.3 |
| Access clarity | 27.6 |

**Agent readiness 29.4** — agent-ready

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | documented (partial credit) |
| MCP Server | — |
| Auth Clarity | bearer (partial credit) |
| Idempotency | — |
| Error Semantics | verified |
| OpenAPI Examples | — |
| Rate Limit Signal | documented (partial credit) |
| Event Surface Described | — |
| Agent Skills | derived (partial credit) |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | — |
| Protected Resource Metadata | — |
| Dynamic Client Registration | — |
| Agentic Commerce | — |
| Pagination Ratio | partial |
| Operation Distinctiveness | mostly |

**Regulatory alignment 24.5** — Horizontal (data, software, accessibility, platform)

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/quandela/](https://apis.io/providers/quandela/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for Quandela: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/quandela/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
