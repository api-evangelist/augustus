---
name: Create and track a payout
description: Send money to a beneficiary with Augustus and follow it to a terminal status via webhooks.
api: openapi/augustus-openapi-original.yml
operations:
- PayoutsController_create
- PayoutsController_retrieve
- PayoutsController_list
---

# Create and track a payout

Use the Augustus Banking API (base `https://api.augustus.com`, sandbox
`https://api.sandbox.augustus.com`) to pay out to an IBAN/BIC, sort code, ABA,
or crypto wallet.

## Auth
Send `Authorization: Bearer {api_key}`. The key needs scope `payouts:write`
(and `payouts:read` to read status). Sandbox keys are prefixed `sandbox.`.

## Steps
1. **Create the payout** — `PayoutsController_create` (`POST /v1/payouts`) with
   `source_account_id`, amount, currency, and a `destination`. Always send an
   `Idempotency-Key` header (UUID v4) so a retry never double-pays. Include a
   `reference` and optional `metadata`.
2. **Read status** — `PayoutsController_retrieve` (`GET /v1/payouts/{id}`) or
   subscribe to `payout.initiated` / `payout.paid` / `payout.failed` webhooks
   rather than polling.
3. **Reconcile** — `PayoutsController_list` (`GET /v1/payouts`) with cursor
   pagination (`cursor`, `has_more`, `next_cursor`), newest first.

## Error handling
Branch on the error `code`, never the `message`. Common: `insufficient_scope`
(403, add the scope), `parameter_invalid` / `validation_error` (400), `conflict`
(409), `idempotency_key_already_used` (409 — reused key with different params).
Capture `correlation_id` for support.
