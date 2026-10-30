# iBanFirst

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

iBanFirst is a Brussels-headquartered cross-border payment and foreign-exchange platform for businesses, licensed as a payment institution by the National Bank of Belgium (company number 0849.872.824) and operating across Belgium, France, the Netherlands, Germany, Spain, Romania, Bulgaria, Italy, Hungary and the United Kingdom. It provides multi-currency "augmented currency accounts" with dedicated IBANs, SWIFT and SEPA payments with payment tracking, spot FX in 30+ currencies, and fixed forward payment contracts for currency-risk hedging. Its developer surface is a public REST API (iBanFirst API 1.6.0, 38 operations over accounts, financial movements, beneficiaries, payments, spot trades, fixed forwards, documents and webhook subscriptions), X-WSSE token authentication, HMAC-SHA256-signed webhooks for payment and trade events, a separate eIDAS/QWAC-gated PSD2 XS2A API for AIS and PIS third-party providers, and a hosted first-party remote MCP server that gives agents read access to treasury data.

**Areas:** Company · Fintech Insurtech · Cross-Border Payments · Foreign Exchange · B2B Payments · Treasury · multi-currency-accounts · SEPA · PSD2 · Open Banking · currency-risk-management · Webhook

## iBanFirst on APIs.io

Everything profiled for iBanFirst is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/ibanfirst/](https://apis.io/providers/ibanfirst/)** — or go straight to one of the APIs or properties below.

### APIs (11)

- [iBanFirst MCP Connector](https://apis.io/apis/ibanfirst/ibanfirst-mcp-connector/) — First-party hosted remote MCP server, launched with API 1.7.0 on 2026-06-05, exposing 16 read tools over wallets, balances, financial movements, payments,…
- [iBanFirst PSD2 XS2A API](https://apis.io/apis/ibanfirst/ibanfirst-psd2-xs2a-api/) — PSD2 access-to-account API for registered third-party providers, covering account information services (AIS), payment initiation services (PIS) and strong…
- [iBanFirst Accounts API](https://apis.io/apis/ibanfirst/ibanfirst-accounts-api/) — Each of your accounts has its own specific currency and IBAN. The API allows you to get details and balances about each account in real time. **Note :**…
- [iBanFirst Beneficiaries API](https://apis.io/apis/ibanfirst/ibanfirst-beneficiaries-api/) — A beneficiary can be either your own account in another bank or a third party recipient account. Beneficiaries can be created or deleted through the API.…
- [iBanFirst Documents API](https://apis.io/apis/ibanfirst/ibanfirst-documents-api/) — The API allows you to access your documents stored on the iBanFirst platform through a one-time access link. Documents must be generated on the platform…
- [iBanFirst Financial movements API](https://apis.io/apis/ibanfirst/ibanfirst-financial-movements-api/) — The API allows you to retrieve all financial movements from your accounts.
- [iBanFirst Fixed forward payment contract API](https://apis.io/apis/ibanfirst/ibanfirst-fixed-forward-payment-contract-api/) — Book a fixed forward payment contracts instantly on iBanFirst without manual intervention. - Available currency pairs: - **EUR/USD** - **EUR/GBP** -…
- [iBanFirst Logs API](https://apis.io/apis/ibanfirst/ibanfirst-logs-api/) — The Logs API from iBanFirst — 2 operation(s) for logs.
- [iBanFirst Payments API](https://apis.io/apis/ibanfirst/ibanfirst-payments-api/) — Sending funds from one of your iBanFirst accounts to your own external bank account or a third-party recipient involves two steps: 1. Generate the payment…
- [iBanFirst Spot trades API](https://apis.io/apis/ibanfirst/ibanfirst-spot-trades-api/) — The API provides a deliverable FX facility and deliverable FX liquidity. You will become counterparty to iBanFirst and can market and sell deliverable FX…
- [iBanFirst Webhook subscriptions API](https://apis.io/apis/ibanfirst/ibanfirst-webhook-subscriptions-api/) — **1. WHAT IS A WEBHOOK ?** - Webhooks are events based real-time notifications providing updates on transactions and removing the need for periodic polling.…

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### AsyncAPI (1)

Event-driven and webhook contracts.

- [Ibanfirst Webhooks](https://apis.io/asyncapis/ibanfirst/ibanfirst-webhooks/)

#### MCP servers (1)

Model Context Protocol servers that expose the API to agents.

- [iBanFirst MCP](https://apis.io/servers/ibanfirst/ibanfirst-mcp/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Ibanfirst Agentic Access](https://apis.io/agentic-access/ibanfirst/ibanfirst-agentic-access/)

#### JSON Schema (30)

The data models the API reads and writes.

- [Amount](https://apis.io/schemas/ibanfirst/ibanfirst-amount/)
- [Balance](https://apis.io/schemas/ibanfirst/ibanfirst-balance/)
- [Currency](https://apis.io/schemas/ibanfirst/ibanfirst-currency/)
- [CurrencyPair](https://apis.io/schemas/ibanfirst/ibanfirst-currency-pair/)
- [Date](https://apis.io/schemas/ibanfirst/ibanfirst-date/)
- [Datetime](https://apis.io/schemas/ibanfirst/ibanfirst-datetime/)
- [Document](https://apis.io/schemas/ibanfirst/ibanfirst-document/)
- [events](https://apis.io/schemas/ibanfirst/ibanfirst-events/)
- [ExternalBankAccount](https://apis.io/schemas/ibanfirst/ibanfirst-external-bank-account/)
- [ExternalBankAccountVOP](https://apis.io/schemas/ibanfirst/ibanfirst-external-bank-account-vop/)
- [FinancialMovement](https://apis.io/schemas/ibanfirst/ibanfirst-financial-movement/)
- [FixedForwardQuote](https://apis.io/schemas/ibanfirst/ibanfirst-fixed-forward-quote/)
- [FixedForwardTrade](https://apis.io/schemas/ibanfirst/ibanfirst-fixed-forward-trade/)
- [Holder](https://apis.io/schemas/ibanfirst/ibanfirst-holder/)
- [HolderBank](https://apis.io/schemas/ibanfirst/ibanfirst-holder-bank/)
- [ID](https://apis.io/schemas/ibanfirst/ibanfirst-id/)
- [Log](https://apis.io/schemas/ibanfirst/ibanfirst-log/)
- [Payment](https://apis.io/schemas/ibanfirst/ibanfirst-payment/)
- [PaymentOption](https://apis.io/schemas/ibanfirst/ibanfirst-payment-option/)
- [PaymentVOP](https://apis.io/schemas/ibanfirst/ibanfirst-payment-vop/)
- [ProcessResult](https://apis.io/schemas/ibanfirst/ibanfirst-process-result/)
- [Quote](https://apis.io/schemas/ibanfirst/ibanfirst-quote/)
- [quoteRef](https://apis.io/schemas/ibanfirst/ibanfirst-quote-ref/)
- [Trade](https://apis.io/schemas/ibanfirst/ibanfirst-trade/)
- [TradeReconciliation](https://apis.io/schemas/ibanfirst/ibanfirst-trade-reconciliation/)
- …and 5 more published on APIs.io

#### JSON-LD (1)

Linked-data contexts for the provider's vocabulary.

- [Ibanfirst Context](https://apis.io/jsonld/ibanfirst/ibanfirst-context/)

#### Overlays (1)

OpenAPI overlays, including conversational phrasing for each operation.

- [Ibanfirst Clientapi Overlay](https://apis.io/overlays/ibanfirst/ibanfirst-clientapi-overlay/)

#### Governance rules (1)

Spectral rulesets for the provider's contracts.

- [iBanFirst API Rules](https://apis.io/rules/ibanfirst/ibanfirst-rules/)

#### Security (3)

Authentication, trust centre, and vulnerability disclosure.

- [Ibanfirst Authentication](https://apis.io/security/ibanfirst/ibanfirst-authentication/)
- [Ibanfirst Domain Security](https://apis.io/security/ibanfirst/ibanfirst-domain-security/)
- [Ibanfirst Vulnerability Disclosure](https://apis.io/security/ibanfirst/ibanfirst-vulnerability-disclosure/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Ibanfirst Plans Pricing](https://apis.io/plans/ibanfirst/ibanfirst-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Ibanfirst Rate Limits](https://apis.io/rate-limits/ibanfirst/ibanfirst-rate-limits/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://www.ibanfirst.com/)
- [Documentation](https://docs.ibanfirst.com/)
- [Getting started](https://docs.ibanfirst.com/guides/quickstart)
- [API reference](https://docs.ibanfirst.com/api/clientapi)
- [Pricing](https://ibanfirst.com/fees)
- [Sign up](https://info.ibanfirst.com/en/request-an-account)
- [Support](https://support.ibanfirst.com/en/)
- [Changelog](https://docs.ibanfirst.com/changelog)
- [Blog](https://blog.ibanfirst.com/en)
- [Security](https://ibanfirst.com/security-policy.html)
- [Compliance](https://ibanfirst.com/psd2-api)
- [Terms of service](https://ibanfirst.com/terms)
- [Privacy policy](https://ibanfirst.com/privacy-policy)
- [GitHub](https://github.com/iBanFirst)

## API Evangelist rating

**Kin Score 63.8** — strong · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 76.7 |
| Contract quality | 68.5 |
| Contract governance | 35.6 |
| Operational transparency | 44.7 |
| Developer ergonomics | 68.5 |
| Access clarity | 73.7 |

**Agent readiness 55.5** — agent-ready (gated down from agent-native)

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | documented (partial credit) |
| MCP Server | verified |
| Auth Clarity | served (partial credit) |
| Idempotency | — |
| Error Semantics | derived (partial credit) |
| OpenAPI Examples | partial |
| Rate Limit Signal | documented (partial credit) |
| Event Surface Described | yes |
| Agent Skills | derived (partial credit) |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | served (partial credit) |
| Protected Resource Metadata | verified |
| Dynamic Client Registration | yes |
| Agentic Commerce | — |
| Pagination Ratio | partial |
| Operation Distinctiveness | distinct |

**Regulatory alignment 28.3** — Banking & Open Finance

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/ibanfirst/](https://apis.io/providers/ibanfirst/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for iBanFirst: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/ibanfirst/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
