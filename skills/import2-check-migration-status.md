---
name: Check Import2 migration status
description: List a customer's imports and report the status and record counts of a migration using the Import2 API v2.1.
api: openapi/import2-openapi.yml
operations: [listImports, getImport]
---

# Check Import2 migration status

## Auth
HTTP Basic with your API token as the username (`curl -u <api_token>:X`).

## Steps

1. **List the customer's imports** — call `listImports` (`GET /imports`) with the
   required `destination_username` query parameter. You get an array of
   `{id, token, status, items, users}`.
2. **Inspect one import** — call `getImport` (`GET /imports/{id}`) for the full
   record `{id, token, status, family, items, users}`.
3. **Report** — surface `status` (e.g. "Migration in progress", "Migration completed"),
   `items` (record count) and `users` (user count).

## Rules
- Respect the global **10 req/sec** limit (`429` on exceed).
- `404` means the import was not found, archived, or cancelled.
