---
name: Quote and book a spot FX trade with iBanFirst
description: >-
  Request a spot FX quote, book the trade against it, and reconcile the settlement legs — including
  how to read the client margin out of the Rate object and why a quote must not be retried blindly.
api: openapi/ibanfirst-clientapi-openapi.yml
api_version: 1.6.0
base_url: https://api.ibanfirst.com/api
test_base_url: https://api-demo.ibanfirst.com/api
operations:
  - 'GET /wallets'
  - 'POST /quotes'
  - 'POST /trades'
  - 'GET /trades/{id}'
  - 'GET /trades/_{status}'
  - 'GET /logs/{nonce}'
generated: '2026-08-17'
method: generated
source: >-
  openapi/ibanfirst-clientapi-openapi.yml, data-model/ibanfirst-data-model.yml,
  conventions/ibanfirst-conventions.yml
---

# Quote and book a spot FX trade with iBanFirst

Spot trades settle instantly and convert funds between two of the caller's own wallets. The API
declares **no `operationId`s**, so every step names the verbatim `METHOD /path`.

## Preconditions

- `X-WSSE` header on every request, recomputed each time (~5 minute validity). See
  `authentication/ibanfirst-authentication.yml`.
- Assert your base URL: `https://api-demo.ibanfirst.com/api` for demo,
  `https://api.ibanfirst.com/api` for live. Credentials look identical on both.
- Both legs of the trade must be wallets you hold. Enumerate them with `GET /wallets` and match on
  `currency`.

## Step 1 — request a quote

`POST /quotes`

Supply the `currencyPair` and the amount you are fixing (`sourceAmount` or `deliveredAmount`) plus
the `deliveryDate`.

The response is a `Quote`: `id`, `appliedRate`, `currencyPair`, `sourceAmount`, `deliveredAmount`,
`createdDate`, `deliveryDate`.

**Read the margin.** Where the response carries a `Rate` object it exposes both sides of the book:

| Field | Meaning |
|---|---|
| `midMarket` | mid-market reference |
| `coreBid` / `coreAsk` | the raw bid/ask |
| `appliedBid` / `appliedAsk` | the bid/ask actually applied to you |

The difference between `core*` and `applied*` is your margin. iBanFirst does not publish its FX
margin — the Pricing Conditions say the rate is "a personalised foreign exchange rate quote" — so
this object is the only place the cost of the trade is visible. Record it.

A quote is a price at a moment. Treat it as short-lived; re-quote rather than reuse a stale one.

## Step 2 — book the trade

`POST /trades`

Book against the quote from step 1 rather than re-pricing at book time.

**No idempotency key exists on this API.** A retried `POST /trades` can create a second trade. On a
timeout:

1. Do not resend.
2. Call `GET /logs/{nonce}` with the nonce from the `X-WSSE` header of the failed request to see
   whether it was received and what it returned.
3. Otherwise list with `GET /trades/_{status}` and match on your `tag`, `currencyPair` and amounts
   before considering a second attempt.

Set a `tag` — "A custom wording for the trade" — on every trade so this reconciliation is possible.

Note the path spelling on the list operation: it is literally `/trades/_{status}`, with an
underscore before the template. That is not a typo in this document.

## Step 3 — confirm and reconcile

`GET /trades/{id}` returns the trade. Where the API returns the richer `TradeReconciliation`
projection you also get:

- `status`, `side` (`S` to sell, `B` to buy)
- `sourceWalletId` and `deliveryWalletId` — both legs by id
- `accountSourceNumber` and `accountTargetNumber` — both legs by IBAN
- `rate` — the full `Rate` object

Reconcile the delivered amount into your ledger against `deliveryWalletId`, then confirm the cash
landed with `GET /wallets/{id}/balance/{date}`.

## Step 4 — watch for the terminal states

Trade events are available over webhooks (added in API 1.6.0): `TRADE_PLANIFIED`,
`TRADE_FINALIZED`, `TRADE_CANCELED`, `TRADE_BLOCKED`. Subscribe rather than poll — see
`skills/ibanfirst-subscribe-to-payment-events.md`.

`TRADE_BLOCKED` means a human needs to look at it. Do not auto-retry a blocked trade.

## Errors

All non-2xx outcomes bind to one `default` response carrying `Error`
(`errorCode`, `errorType`, `errorMessage`, `link`). No 4xx/5xx status codes are enumerated and the
`errorCode` values are not published. Log them verbatim and surface `link`.

Requesting a list with no matching rows returns **HTTP 204**, not `200 []`.

## Agent guidance

`POST /trades` commits the caller to an FX rate and is not reversible by an API call — cancellation
is an operational request. `POST /quotes` is safe to call autonomously; `POST /trades` should not be.
iBanFirst's own MCP connector exposes `get_fx_rates`, `get_trades` and `get_trade_details` and
deliberately exposes **no** booking tool.
