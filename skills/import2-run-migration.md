---
name: Run an Import2 data migration
description: Create a data migration job from a source SaaS tool to a destination account and drive it to completion using the Import2 API v2.1.
api: openapi/import2-openapi.yml
operations: [listTools, createImport, getImport, acceptImportPayment]
---

# Run an Import2 data migration

Use the Import2 API v2.1 to migrate data between SaaS tools.

## Auth
HTTP Basic. Supply your API token as the username and any value as the password
(`curl -u <api_token>:X`). Request a token from partners@import2.com. Test against
the sandbox base URL `https://www.import2.com/api/sandbox` before production
(`https://www.import2.com/api/v2.1`).

## Steps

1. **Discover source tools** — call `listTools` (`GET /tools`), optionally filtered
   by `family` (e.g. `crm`, `helpdesk`). Pick the `name` of the source tool.
2. **Create the import** — call `createImport` (`POST /imports`) with `source_tool`
   (required) plus optional `destination_instance_url`, `destination_username`,
   `destination_token`, `family`, and `start_full`. A `201` returns `{id, token, redirect_url}`.
   Send the user to `redirect_url` to finish connecting credentials.
3. **Poll status** — call `getImport` (`GET /imports/{id}`) and read `status`. It moves
   through: Collecting credentials -> Waiting to start sample migration -> Sample
   migration in progress -> Sample migration completed -> Migration in progress ->
   Migration completed.
4. **Accept a sponsored migration** — if the migration is vendor-sponsored, call
   `acceptImportPayment` (`POST /imports/{id}/payment/accept`) to accept, or
   `rejectImportPayment` to decline.

## Rules
- Stay within **10 requests/second** globally or you get `429`; back off and retry.
- No idempotency key exists — do not blindly retry `createImport`; check `listImports`
  (`GET /imports?destination_username=...`) first to avoid duplicate jobs.
- `422` on create returns a `field -> [messages]` validation map; fix the flagged fields.
- `404` on `getImport` means the import was not found, archived, or cancelled.
