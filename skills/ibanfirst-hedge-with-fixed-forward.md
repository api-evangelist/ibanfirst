---
name: Hedge currency risk with an iBanFirst fixed forward payment contract
description: >-
  Quote and book a fixed forward payment contract to lock a rate for a future settlement date,
  respecting the quote expiry and the published initial-collateral schedule.
api: openapi/ibanfirst-clientapi-openapi.yml
api_version: 1.6.0
base_url: https://api.ibanfirst.com/api
test_base_url: https://api-demo.ibanfirst.com/api
operations:
  - 'GET /wallets'
  - 'POST /fixed-forwards/quote'
  - 'POST /fixed-forwards'
  - 'GET /fixed-forwards'
  - 'GET /fixed-forwards/{fixedForwardId}'
generated: '2026-08-17'
method: generated
source: >-
  openapi/ibanfirst-clientapi-openapi.yml, plans/ibanfirst-plans-pricing.yml,
  data-model/ibanfirst-data-model.yml
---

# Hedge currency risk with an iBanFirst fixed forward payment contract

Fixed forward payment contracts arrived in API **1.6.0** (2026-03-19). They lock an FX rate for a
future delivery date. The API declares **no `operationId`s**; each step names the verbatim
`METHOD /path`.

Two things make this flow different from a spot trade: the quote **expires at a stated instant**,
and booking one can require **collateral**.

## Preconditions

- `X-WSSE` on every request, recomputed each time. See `authentication/ibanfirst-authentication.yml`.
- The module is not on by default. Per iBanFirst's published Pricing Conditions, risk management for
  deliverable forward payment contracts is a "Module activated on request" — confirm it is enabled
  on the account before building against it.
- Identify the two wallets: `GET /wallets`, then match on `currency`.

## Step 1 — quote

`POST /fixed-forwards/quote`

Supply the `currencyPair`, the amount (`sourceAmount` or `deliveredAmount`) and the future delivery
date.

The response is a `FixedForwardQuote`:

| Field | Notes |
|---|---|
| `quoteId` | pass this to the booking call — note it is `quoteId`, not `id` |
| `currencyPair` | |
| `sourceAmount` / `deliveredAmount` | |
| `appliedRate` | the locked rate on offer |
| `expiresAt` | **fractional-second timestamp** — the only explicit expiry in the whole contract |

**Respect `expiresAt`.** Parse it as a fractional-second datetime (`DatetimeFractionnal` in the
spec — the misspelling is the provider's). If it has passed, re-quote; do not attempt to book.

## Step 2 — check the collateral consequence before booking

iBanFirst may require an initial deposit when the contract maturity exceeds 7 days. The published
schedule:

| Maturity | Initial collateral |
|---|---|
| `< 7 days` | 0% |
| `>= 7 days and <= 1 month` | 3% |
| `> 1 month and <= 6 months` | 5% |
| `> 6 months and <= 2 years` | 10% |

Collateral is a percentage of the amount to be paid, must be credited to the collateral account
**within 48 hours** of the forward payments being validated, and is deducted from the final settled
amount. If the market moves against the position, iBanFirst may request additional collateral.

None of that is expressed in the OpenAPI. Compute the expected collateral from the delivery date
before booking so the treasury impact is not a surprise, and confirm it with a human when it is
non-zero.

## Step 3 — book

`POST /fixed-forwards`

Reference the `quoteId` from step 1, the `side` (`S` to sell, `B` to buy), and the two legs.

Note the naming: this entity uses `sourceAccountId` and `deliveryAccountId`, while `Payment` and
`TradeReconciliation` use `sourceWalletId` / `deliveryWalletId` for the same relationship. They all
point at wallets. Do not assume the field names carry over between resources.

**No idempotency key exists.** On a timeout, reconcile before retrying: `GET /fixed-forwards`
(supports pagination) and match on `currencyPair`, amounts and `deliveryDate`. Booking twice creates
two hedges and two collateral obligations.

## Step 4 — track

- `GET /fixed-forwards/{fixedForwardId}` — note the path parameter is `fixedForwardId`, and the
  response body's id field is also `fixedForwardId`, not `id`.
- `GET /fixed-forwards` — list by status, paginated with `page` / `per_page` / `sort`. An empty list
  returns **HTTP 204**.
- `FixedForwardTrade` carries `status`, `appliedRate`, `createdDate` and `deliveryDate`.

There are **no forward-specific webhook events**. The `events` enum covers `PAYMENT_*` and `TRADE_*`
only, so a contract's progress toward settlement must be polled.

## Errors

One `default` response per operation carrying `Error` (`errorCode`, `errorType`, `errorMessage`,
`link`). No enumerated status codes, no published error-code registry. A lapsed quote and an
unfunded collateral account will both surface through this same untyped envelope, so branch on the
HTTP status and log the body.

## Agent guidance

A fixed forward is a multi-month financial commitment with a collateral call attached. Quoting
(`POST /fixed-forwards/quote`) is safe to automate. Booking (`POST /fixed-forwards`) must not be
autonomous — and iBanFirst's own MCP connector agrees: it exposes `get_fix_forwards` and
`get_fix_forward` and no booking tool.
