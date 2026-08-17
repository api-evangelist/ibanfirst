---
name: Read an iBanFirst treasury position
description: >-
  Pull the full multi-currency cash position, statement lines and pending payments — over REST or
  over iBanFirst's own hosted MCP connector — and reconcile the two surfaces where they disagree.
api: openapi/ibanfirst-clientapi-openapi.yml
api_version: 1.6.0
base_url: https://api.ibanfirst.com/api
test_base_url: https://api-demo.ibanfirst.com/api
mcp: https://mcp.ibanfirst.com/mcp
operations:
  - 'GET /wallets'
  - 'GET /wallets/{id}'
  - 'GET /wallets/{id}/balance/{date}'
  - 'GET /financialMovements'
  - 'GET /financialMovements/{id}'
  - 'GET /payments/{status}'
  - 'GET /externalBankAccounts'
  - 'GET /documents/RIB'
generated: '2026-08-17'
method: generated
source: >-
  openapi/ibanfirst-clientapi-openapi.yml, mcp/ibanfirst-mcp.yml,
  mcp/ibanfirst-tool-crosswalk.yml, data-model/ibanfirst-data-model.yml
---

# Read an iBanFirst treasury position

This is the one iBanFirst flow that is fully read-only, and therefore the one an agent can run end
to end. It is also the flow iBanFirst itself exposed as an MCP connector, so there are two ways to
do it. The API declares **no `operationId`s**; each step names the verbatim `METHOD /path`, with the
matching MCP tool alongside.

## Choose a surface

| | REST | MCP |
|---|---|---|
| Endpoint | `https://api.ibanfirst.com/api` | `https://mcp.ibanfirst.com/mcp` |
| Auth | `X-WSSE` digest, recomputed per request (~5 min) | OAuth 2.0, authorization_code + PKCE (S256) |
| Credentials | issued by iBanFirst support, per method | "OAuth Client ID" + "OAuth Client Password" |
| Writes | yes | **no — every tool is a read** |
| History cap | `fromDate` / `toDate`, no documented cap | `get_financial_movements` is capped at **12 months** |

The MCP surface covers 15 of the 38 REST operations. Everything in this skill is available on both;
everything else in this repo's skills is REST-only. See `mcp/ibanfirst-tool-crosswalk.yml`.

## Step 1 — enumerate the accounts

REST: `GET /wallets` — optionally narrowed by currency.
MCP: `get_wallets` — "List all iBanFirst wallets. Optionally filter by 3-letter currency code."

Each `Wallet` carries `id`, `currency`, `status`, `accountNumber` (the IBAN) and your own `tag`. In
the product these are "accounts"; in the contract and in the tool names they are "wallets".

Detail, including holder and bank information: `GET /wallets/{id}` / `get_wallet_details`.

An empty list returns **HTTP 204 No Content**, not `200 []`.

## Step 2 — get balances as of a date

REST: `GET /wallets/{id}/balance/{date}` (date as `YYYY-MM-DD`).
MCP: `get_wallet_balance`.

`Balance` returns `closingDate`, `bookingAmount` and `valueAmount`. **Use the right one:**
`bookingAmount` is what has been booked; `valueAmount` is what is available at value date. For a
cash-position report, `valueAmount` is usually the figure you want.

Note that `Balance` is not addressable on its own — it exists only under a wallet and a date. There
is no "all balances" call, so a multi-currency position is N calls, one per wallet. Fan them out.

## Step 3 — pull the statement lines

REST: `GET /financialMovements` with `walletId`, `fromDate`, `toDate`, `page`, `per_page`, `sort`.
MCP: `get_financial_movements` — **capped at the last 12 months** by the tool's own description,
even though the REST operation documents no cap. If you need older history, use REST.

`FinancialMovement` is deliberately flat: `bookingDate`, `valueDate`, ordering and beneficiary
customer/institution as **free-form strings**, `orderingAmount`, `beneficiaryAmount`,
`remittanceInformation`, `chargesDetails`, `exchangeRate`, `typeLabel`, `internalReference`.

**The join you want does not exist.** Movements reference counterparties by *account number*, not by
entity id, so you cannot join a movement back to a `Wallet` or an `ExternalBankAccount` by id. Match
on `accountNumber` from step 1 and on `beneficiaryAccountNumber` / `orderingAccountNumber` here, and
expect fuzzy matches on the free-text institution fields.

Detail: `GET /financialMovements/{id}` / `get_financial_movement_details`.

### Paginate correctly

`page` / `per_page` / `sort` are accepted, but responses are **bare arrays with no total count and
no next/prev links**. You cannot tell from a response whether more pages exist. Page until you get a
short page or a **204**, and cap your loop.

## Step 4 — add what is in flight

REST: `GET /payments/{status}` — statuses documented for the MCP equivalent include `all`,
`planified`, `finalized` and `rejected`.
MCP: `get_payments_by_status`, then `get_payment_details`.

Pending outbound payments are the difference between a balance and a true position. Include
`planified` and anything awaiting confirmation or signature
(`PAYMENT_AWAITING_CONFIRMATION`, `PAYMENT_WAITING_SIGNATURE`) as committed-but-unsettled.

## Step 5 — context you may need

- `GET /externalBankAccounts` / `get_external_bank_accounts` — the beneficiary register, to label
  outbound flows.
- `GET /documents/RIB` — the account's RIB document, for onboarding a counterparty.
- `get_fx_rates` (MCP) / `POST /quotes` (REST) — to revalue non-base-currency balances. Be aware
  that `POST /quotes` is a **POST that requests a price**; it does not commit anything, but it is the
  only rate-producing operation in the REST contract, which is why the read-only MCP connector maps
  its `get_fx_rates` tool onto it.

## Reconciliation checklist

1. Sum `valueAmount` per currency from step 2 → gross position.
2. Subtract unsettled outbound payments from step 4 → net position.
3. Tie the movement lines from step 3 to the change in `bookingAmount` between two dates.
4. Where a movement will not tie, check `chargesDetails` and `exchangeRate` — fees and conversion
   are on the movement, not on a separate fee object.

## Limits and caveats

- **No rate limits are published.** No `RateLimit-*` headers, no `Retry-After`, no documented `429`.
  When fanning out balance calls across many wallets, throttle yourself conservatively; nothing tells
  you the budget. See `rate-limits/ibanfirst-rate-limits.yml`.
- **REST tokens expire in ~5 minutes.** For a long fan-out, recompute the `X-WSSE` digest per
  request rather than once per run.
- **Errors are untyped.** One `default` response per operation carrying `Error` (`errorCode`,
  `errorType`, `errorMessage`, `link`); no status codes are enumerated and no error-code registry is
  published.
- **MCP `tools/list` is OAuth-gated** (HTTP 401 `invalid_token` anonymously), so a client discovers
  the tool schemas only after authenticating.

## Agent guidance

Every operation in this skill is a read, and iBanFirst's own connector is read-only by construction,
so a treasury-reporting agent can be given this whole flow. Keep it on the MCP surface if you can:
the OAuth credential there cannot be used to move money, whereas an `X-WSSE` credential granted the
write methods can.
