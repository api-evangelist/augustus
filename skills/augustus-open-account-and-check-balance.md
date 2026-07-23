---
name: Open an account and check its balance
description: Create an Augustus account and read its available balance, plus the aggregated account-program balance.
api: openapi/augustus-openapi-original.yml
operations:
- AccountsController_create
- AccountsController_retrieve
- AccountsController_balance
- AccountProgramsController_retrieveBalance
---

# Open an account and check its balance

Accounts hold fiat and stablecoin funds under an account program.

## Auth
`Authorization: Bearer {api_key}` with scopes `accounts:write` (create) and
`accounts:read` / `account_programs:read` (read).

## Steps
1. **Create the account** — `AccountsController_create` (`POST /v1/accounts`).
   Send an `Idempotency-Key` header so a retry doesn't create duplicates.
2. **Confirm it** — `AccountsController_retrieve` (`GET /v1/accounts/{id}`).
3. **Read the account balance** — `AccountsController_balance`
   (`GET /v1/accounts/{id}/balance`); the available balance is broken down by
   currency.
4. **Read the program balance** — `AccountProgramsController_retrieveBalance`
   (`GET /v1/account_programs/{id}/balance`) for the aggregated available
   balance across all virtual accounts in the program.

## Lifecycle
Accounts can be frozen (`AccountsController_freeze`), unfrozen
(`AccountsController_unfreeze`), and closed (`AccountsController_close`).

## Error handling
Branch on error `code`. `insufficient_scope` (403) means add the scope;
`resource_not_found` (404) means the id/environment is wrong (test and live are
isolated).
