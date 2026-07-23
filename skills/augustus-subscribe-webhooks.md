---
name: Subscribe to and verify webhooks
description: Register an Augustus webhook subscription, send a test event, and replay failed deliveries.
api: openapi/augustus-openapi-original.yml
operations:
- WebhookSubscriptionsController_create
- WebhookSubscriptionsController_sendTestEvent
- WebhookDeliveriesController_list
- WebhookDeliveriesController_redeliver
- EventsController_list
---

# Subscribe to and verify webhooks

## Auth
`Authorization: Bearer {api_key}` with `webhook_subscriptions:write`,
`events:read`, and `webhook_deliveries:read` / `webhook_deliveries:write`.

## Steps
1. **Create a subscription** — `WebhookSubscriptionsController_create`
   (`POST /v1/webhook_subscriptions`) with an HTTPS `url` and an `events` array
   (e.g. `["payout.paid","payout.failed","deposit.received"]`, or `["*"]` for
   all). Send an `Idempotency-Key` header.
2. **Send a test event** — `WebhookSubscriptionsController_sendTestEvent`
   (`POST /v1/webhook_subscriptions/{id}/send_test_event`) to emit `ping.test`
   and confirm your endpoint is wired up.
3. **Verify signatures** — Augustus signs each delivery with your subscription's
   secret key; verify the signature before trusting the payload.
4. **Inspect deliveries** — `WebhookDeliveriesController_list`
   (`GET /v1/webhook_deliveries`); replay a failure with
   `WebhookDeliveriesController_redeliver`
   (`POST /v1/webhook_deliveries/{id}/redeliver`).
5. **Backfill events** — `EventsController_list` (`GET /v1/events`); events are
   retained for 30 days.

## Handling payloads
Each delivery envelope carries `id`, `type`, `api_version`, `payload`, `date`.
Branch on `type`. Ignore `ping.test` in business logic.
