---
name: Quote and execute an FX conversion
description: Get an indicative FX quote and execute a currency conversion with Augustus.
api: openapi/augustus-openapi-original.yml
operations:
- QuotesController_getIndicativeQuote
- ConversionsController_create
- ConversionsController_retrieve
- ConversionsController_list
---

# Quote and execute an FX conversion

## Auth
`Authorization: Bearer {api_key}` with `quotes:read` and `conversions:write`
(plus `conversions:read` to read results).

## Steps
1. **Get an indicative quote** — `QuotesController_getIndicativeQuote`
   (`GET /v1/quotes/indicative`) for a currency pair. It is indicative only —
   not persisted or holdable.
2. **Execute the conversion** — `ConversionsController_create`
   (`POST /v1/conversions`). Send an `Idempotency-Key` header so a retry doesn't
   convert twice.
3. **Read it** — `ConversionsController_retrieve` (`GET /v1/conversions/{id}`),
   or subscribe to `conversion.created` / `fx.succeeded` / `fx.failed` webhooks.
4. **Reconcile** — `ConversionsController_list` (`GET /v1/conversions`) with
   cursor pagination.

## Error handling
Branch on error `code`. Rates are not guaranteed for subsequent transactions;
re-quote if a conversion is rejected. Capture `correlation_id`.
