---
name: Subscribe to iBanFirst payment and trade events
description: >-
  Create a webhook subscription, verify the HMAC-SHA256 signature correctly, survive the short
  three-attempt retry envelope, rotate the secret, and read the failed-notification log.
api: openapi/ibanfirst-clientapi-openapi.yml
api_version: 1.6.0
base_url: https://api.ibanfirst.com/api
test_base_url: https://api-demo.ibanfirst.com/api
operations:
  - 'POST /webhooks'
  - 'GET /webhooks'
  - 'GET /webhooks/{webhookId}'
  - 'PATCH /webhooks/{webhookId}'
  - 'DELETE /webhooks/{webhookId}'
  - 'POST /webhooks/{webhookId}/rotate-secret'
  - 'GET /webhooks/{webhookId}/failed-notifications'
  - 'GET /payments/{id}'
  - 'GET /trades/{id}'
generated: '2026-08-17'
method: generated
source: >-
  openapi/ibanfirst-clientapi-openapi.yml, asyncapi/ibanfirst-webhooks.yml,
  https://docs.ibanfirst.com/api/clientapi/webhook-subscriptions
---

# Subscribe to iBanFirst payment and trade events

Webhook subscriptions arrived in API **1.4.0** for payment events; trade events, secret rotation,
subscription detail retrieval and the failed-notification log arrived in **1.6.0**. iBanFirst
publishes **no AsyncAPI document** — the event surface exists only in prose and in these REST
operations. The API declares **no `operationId`s**; each step names the verbatim `METHOD /path`.

## Step 1 — create the subscription

`POST /webhooks`

Body: your HTTPS `url` and the `events` array. Valid values, verbatim from the `events` enum in the
spec:

**Payment (9):** `PAYMENT_CREATED`, `PAYMENT_PLANIFIED`, `PAYMENT_FINALIZED`,
`PAYMENT_WAITING_SIGNATURE`, `PAYMENT_AWAITING_CONFIRMATION`, `PAYMENT_CANCELED`,
`PAYMENT_BLOCKED`, `PAYMENT_WAITING_JUSTIFICATION`, `PAYMENT_INCOMING`

**Trade (4):** `TRADE_PLANIFIED`, `TRADE_FINALIZED`, `TRADE_CANCELED`, `TRADE_BLOCKED`

There are no forward-contract events.

The response carries the `webhookId` and the **subscription secret** (32–64 alphanumeric
characters). Store the secret in your secret manager immediately — it is the only thing that proves
a notification came from iBanFirst.

**Limit: 10 active subscriptions.** Budget them by event family, not one per event type.

## Step 2 — verify every notification

Two headers arrive with each delivery:

- `x-ibanfirst-timestamp`
- `x-ibanfirst-signature`

Verification:

1. Take `x-ibanfirst-timestamp` **exactly as received**.
2. Concatenate: `{x-ibanfirst-timestamp}.{raw request body}` — a literal `.` between them.
3. Compute `HMAC-SHA256` over that string using the subscription secret.
4. Compare against `x-ibanfirst-signature` using a constant-time comparison.
5. **Reject the notification if the signatures do not match.**

Two ways this goes wrong in practice:

- **Sign the raw body.** If your framework parses JSON and you re-serialise it, key order and
  whitespace change and the signature will never match. Capture the raw bytes before parsing.
- **Bound the timestamp.** The timestamp is inside the signed message specifically so you can
  reject replays. iBanFirst does not publish a tolerance window, so pick your own — a few minutes —
  and reject anything older.

## Step 3 — read the payload

```json
{
  "event": "PAYMENT_FINALIZED",
  "payload": { },
  "webhookId": "..."
}
```

`payload` is the same body you would get from `GET /payments/{id}` or `GET /trades/{id}` for the
corresponding resource. There is no separate per-event schema in the contract — the payload is
defined by reference to those detail operations.

**Notifications may arrive out of order.** Do not derive state from arrival sequence. Treat each
notification as a hint to refresh, and let `GET /payments/{id}` be your source of truth on status.

## Step 4 — respond fast, and know the retry envelope

Failed deliveries — HTTP **400** or **500** from your endpoint — are retried **twice, 60 seconds
apart**: three attempts total, inside roughly two minutes.

That is a very short envelope for payment events. Consequences to design for:

- Acknowledge with a 2xx as soon as you have durably queued the notification. Never do downstream
  work before acknowledging.
- If your receiver is down for more than about two minutes, the event is **lost**. You must
  reconcile by polling `GET /payments/{status}` or `GET /trades/_{status}`.
- There is no replay or redelivery operation. Failures can be listed, but not resent.

## Step 5 — monitor failures

`GET /webhooks/{webhookId}/failed-notifications`

Returns `webhookFailedNotification` rows: `id`, `notificationContent`, `errorMessage`,
`httpStatusCode`, `failedAt`, `retryCount`. Note `httpStatusCode` is the status **your** endpoint
returned — this log records your failures, not iBanFirst's.

Poll it on a schedule and alert on any row. Since there is no redelivery, a row here is a
reconciliation task.

## Step 6 — rotate and maintain

- `POST /webhooks/{webhookId}/rotate-secret` — rotate the signing secret. Accept both the old and
  new secret during the cutover window, then drop the old one.
- `PATCH /webhooks/{webhookId}` — change the URL or the subscribed events.
- `GET /webhooks` / `GET /webhooks/{webhookId}` — list and inspect.
- `DELETE /webhooks/{webhookId}` — cancel.

## Errors

One `default` response per operation carrying `Error` (`errorCode`, `errorType`, `errorMessage`,
`link`). No enumerated status codes; the `errorCode` values are not published. An empty list returns
**HTTP 204**, not `200 []`.

## Agent guidance

Every operation in this skill is safe for an agent except two: `PATCH /webhooks/{webhookId}` and
`POST /webhooks/{webhookId}/rotate-secret` change or invalidate a security-critical credential, and
`DELETE /webhooks/{webhookId}` silently stops the event stream. Gate those three behind a human.
