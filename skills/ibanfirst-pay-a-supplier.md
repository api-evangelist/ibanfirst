---
name: Pay a supplier abroad with iBanFirst
description: >-
  Register a beneficiary, price the payment, create it, and confirm it — the core money-movement
  flow of the iBanFirst REST API, including verification-of-payee handling and the retry hazard
  that comes from the API having no idempotency key.
api: openapi/ibanfirst-clientapi-openapi.yml
api_version: 1.6.0
base_url: https://api.ibanfirst.com/api
test_base_url: https://api-demo.ibanfirst.com/api
operations:
  - 'GET /wallets'
  - 'POST /externalBankAccounts'
  - 'GET /externalBankAccounts'
  - 'GET /payments/options/{walletId}/{externalBankAccountId}'
  - 'POST /payments'
  - 'PUT /payments/{id}/confirm'
  - 'GET /payments/{id}'
  - 'DELETE /payments/{id}'
  - 'GET /logs/{nonce}'
generated: '2026-08-17'
method: generated
source: >-
  openapi/ibanfirst-clientapi-openapi.yml, conventions/ibanfirst-conventions.yml,
  errors/ibanfirst-problem-types.yml
---

# Pay a supplier abroad with iBanFirst

The iBanFirst OpenAPI declares **no `operationId` on any operation**, so every step below names the
verbatim `METHOD /path` from the contract. Do not invent an operationId; there is none to use.

## Before you start

1. **Authentication.** Every request carries an `X-WSSE` header and there is no bearer token
   option. The header is a WS-Security UsernameToken and it **expires in about 5 minutes**, so
   compute it fresh for every single request:

   ```
   X-WSSE: UsernameToken Username="<username>", PasswordDigest="<digest>", Nonce="<nonce_b64>", Created="<timestamp>"
   ```

   - `nonce` — at least 32 lowercase hex characters
   - `created` — UTC ISO 8601, `YYYY-MM-DDTHH:MM:SSZ`
   - `PasswordDigest` = `Base64( SHA-1( nonce_bytes + created_bytes + secret_bytes ) )`, over the
     **raw binary** SHA-1 digest
   - `Nonce` = `Base64( nonce_utf8_bytes )`

   **Persist the nonce you generated for every write.** The API returns no request-id header, and
   `GET /logs/{nonce}` is the only way to find out afterwards whether a request was received.
   See `authentication/ibanfirst-authentication.yml`.

2. **Environment.** The only thing separating test from live is the hostname —
   `https://api-demo.ibanfirst.com/api` versus `https://api.ibanfirst.com/api`. Credentials carry
   no test/live prefix, so a wrong base URL sends real payment instructions. Assert the host before
   any write.

3. **Forbidden characters.** `&` `<` `>` `%` `?` `\` `/` `|` are rejected in route parameters, query
   parameters and JSON bodies. Transliterate supplier names, addresses and payment references
   before submitting.

## Step 1 — find the source account

`GET /wallets`

Returns the caller's wallets. Each carries `id`, `currency`, `status` and `accountNumber` (the
IBAN). Pick the wallet whose `currency` you intend to debit.

If the account list is empty the API returns **HTTP 204 No Content**, not `200 []`. Treat 204 as
zero rows.

## Step 2 — register the beneficiary

Check whether the beneficiary already exists with `GET /externalBankAccounts` (supports `page`,
`per_page`, `sort`). If not:

`POST /externalBankAccounts`

Supply the account number/IBAN, currency, holder (name, type, address), holder bank (BIC or local
clearing code) and optionally `contactEmail` and a `tag` for your own reference.

**Handle verification of payee.** This operation can return the `ErrorVOP` envelope instead of the
plain `Error` envelope:

```json
{
  "errorCode": 0,
  "errorType": "...",
  "errorMessage": "...",
  "payeeVerification": {
    "status": "PARTIAL",
    "message": "...",
    "corrections": {
      "account_holder_name": "...",
      "account_holder_type": "Corporate"
    }
  }
}
```

- `status: PARTIAL` — **actionable.** Resubmit with `corrections.account_holder_name` and
  `corrections.account_holder_type`. Do not treat it as a terminal failure.
- `status: FAILED` — stop and escalate to a human. Do not retry with the same details.

## Step 3 — price the payment before creating it

`GET /payments/options/{walletId}/{externalBankAccountId}`

Returns the available payment options and fee estimates for that wallet/beneficiary pair. Both path
parameters are required.

Note the contract limitation: `PaymentOption` is declared as a bare untyped `object` in the spec, so
read the response defensively — its fields are not described in the machine-readable contract.

Use it to choose:
- `speedOption` (`paymentSpeedOption`, added in API 1.6.0)
- `priorityPaymentOption` (`paymentPriorityOption`)
- `feePaymentOption` — the charge-bearer code. On SWIFT this is the SHARE/OUR/BEN choice, and per
  the published Pricing Conditions it costs EUR 5 / EUR 25 / EUR 0 respectively.

## Step 4 — create the payment

`POST /payments`

Reference `sourceWalletId` (from step 1) and `externalBankAccountId` (from step 2), plus `amount`,
`desiredExecutionDate`, `communication` (the wording on the transfer), the option fields from
step 3, and a `tag` you control.

**This is the dangerous step, and the reason this skill exists.** The API has **no
`Idempotency-Key`** — no such parameter appears on any operation. If the request times out you
cannot safely retry it.

Do this instead:

1. Set a unique `tag` on the payment before sending.
2. On a timeout or ambiguous error, **do not resend.** Call `GET /logs/{nonce}` with the nonce from
   your `X-WSSE` header to see whether the request reached iBanFirst and what it returned.
3. If the log is inconclusive, list payments with `GET /payments/{status}` and look for your `tag`.
4. Only create a second payment once you have positively established that the first does not exist.

Payments are created **unconfirmed**, which is your safety net: a duplicate that has not been
confirmed can be removed with `DELETE /payments/{id}`.

## Step 5 — confirm the payment

`PUT /payments/{id}/confirm`

Nothing moves until this call succeeds. Operations that only report success return
`ProcessResult { "result": true }`.

Confirm exactly once, and only after step 4 has been reconciled.

## Step 6 — track and evidence

- `GET /payments/{id}` — current state. The `tracker` field carries a payment-tracker link for
  SWIFT payments (added in 1.3.0); it is not populated for every payment.
- `PUT /payments/{id}/proofOfTransaction` — upload proof of transaction.
- `GET /payments/{status}` — list by status. Statuses seen in the MCP tool documentation include
  `all`, `planified`, `finalized` and `rejected`.
- Prefer webhooks over polling: subscribe to `PAYMENT_CREATED`, `PAYMENT_PLANIFIED`,
  `PAYMENT_FINALIZED`, `PAYMENT_WAITING_SIGNATURE`, `PAYMENT_AWAITING_CONFIRMATION`,
  `PAYMENT_CANCELED`, `PAYMENT_BLOCKED`, `PAYMENT_WAITING_JUSTIFICATION`. See
  `skills/ibanfirst-subscribe-to-payment-events.md`.

## Cancelling

`DELETE /payments/{id}` deletes a payment. Per the published Pricing Conditions, a payment
cancellation or amendment after the fact costs EUR 20, so cancel before confirmation where you can.

## Error handling

Every operation declares exactly one `default` error response bound to the `Error` schema —
`errorCode`, `errorType`, `errorMessage`, `link`. **No 4xx or 5xx status is enumerated anywhere in
the contract, and the `errorCode` values are not published**, so:

- Branch on the HTTP status you actually receive, not on a documented list.
- Log `errorCode` and `errorType` verbatim; surface `link` to the operator.
- Never infer that an error means the payment was not created — see step 4.
- There is no `429` contract and no `Retry-After` header. Back off conservatively on your own
  schedule; nothing tells you the budget.

## What an agent must not do autonomously

`POST /payments` and `PUT /payments/{id}/confirm` move money, there is no idempotency key, and there
is no scope model that could restrict a credential to reads. Require a human confirmation before
step 4 and step 5. Note that iBanFirst's own MCP connector exposes **no** payment-creation tool for
exactly this reason — its 16 tools are all reads.
