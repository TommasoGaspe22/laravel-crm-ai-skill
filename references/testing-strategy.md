# Testing strategy (Laravel)

**Scope:** test strategy for the CRM. Source: **Proposed for Laravel**. Reuses the existing `tests/Feature` patterns (the pipeline already has tests for access/CRUD/import/export/bulk).

## Levels & what to cover
- **Unit:** accessors (`computed_status`, `is_closed/is_won`), enum/config, `CommercialPipelineActionService` (bucket/next-action per fixture), `FilterQueryBuilder` (operator→SQL translation), `TimelineBuilder`.
- **Feature (HTTP):**
  - **List view:** every view/filter, `q` search, sort (whitelist rejects unknown columns — anti-injection), query-string pagination, empty state, missing table → no-500.
  - **Record page:** authorized show, inline edit (partial `PATCH`), related counts.
  - **Form/validation:** inline required, types, `in:` picklist, email/date; 422/redirect-errors; no partial write.
  - **Conversion:** new account · existing account (dedup) · without an opportunity · minimal data · rollback on error (mock failure) · lead status=converted · relation transfer · idempotency.
  - **Stage transition:** valid/invalid transitions, won/lost closing + loss reason, audit written.
  - **Activities/timeline:** logging task/call/note → shows up in the timeline; completion; upcoming/overdue scopes.
  - **Bulk:** applies only to the given ids, transaction, per-record authorization, partial outcome.
  - **Saved views/filters:** serialization, field whitelist.
  - **Policy/permissions:** guest→login; sales vs admin (delete/bulk/assign 403 for sales); "assigned to me" scope; convert.
- **Data integrity/migration:** up/down on SQLite; non-destructive additive changes; FK/indexes; backfill; soft delete + restore.
- **Regression:** existing tests (Today/pipeline/import/export) stay green after the service extraction and additive migrations.
- **Browser/E2E (V2):** critical flows (create lead → convert → opportunity → change stage → activity) with Playwright/Dusk.
- **Accessibility:** aria/focus/contrast on key components (markup assertions + manual audit).
- **Performance:** lists with N records (seed factory), assert on query count (no N+1, e.g. `assertDatabaseQueryCount`), indexes used; target: list < X queries, kanban < Y.

## Factories & seeds
`Crm*Factory` + a pipeline-entry factory; demo seeder kept **separate** from real data (fake data only, never real PII). Test seeds for every scenario (statuses/stages/activities).

## Commands
```bash
composer test        # php artisan test (config:clear first)
./vendor/bin/pint    # style
npm run build        # Vite/Tailwind
php artisan test --filter=Crm
```

## Definition of tested-enough (per feature)
Happy path + validation + authorization + edge cases (empty/missing) covered; existing regression green; `pint` clean; build ok; manual flow verification (`verification-before-completion`).

## Priority
- **V1:** unit service/accessors, feature (views/validation/conversion/stage/policy/bulk), migrations, regression. **V2:** E2E, performance, automated accessibility. **V3:** load testing, API contract tests.
