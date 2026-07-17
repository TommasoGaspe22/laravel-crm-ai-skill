# Testing, verification & done-criteria

Follow superpowers:test-driven-development: write the failing test first for every controller/service/policy change, then implement. Feature tests use the existing `tests/Feature` patterns (the pipeline already has feature tests for access, CRUD, import/export, bulk — mirror them).

## Required feature tests

**Service refactor**
- `CommercialPipelineActionService` returns the same buckets/tone/action as the old inline logic for representative fixtures (overdue, today, upcoming, first_contact, analysis_pending, stalled, sales_pending, flagged, none).
- `/dashboard/oggi` still renders identically after the refactor (existing Today tests stay green — run them).

**Migration & model**
- Migration up/down runs clean on SQLite; new columns nullable; existing rows untouched.
- Don't just run `migrate` and call it done: actually exercise `migrate` → `migrate:rollback` → `migrate` (or `migrate:fresh` then rollback) for every new/edited migration. SQLite rebuilds the whole table per `Schema::table()` call, and dropping an indexed column in the wrong grouping/order throws on rollback even though `up()` looked fine — confirmed while building this skill (see `data-model.md` "Verified pitfall").
- `computed_status` accessor returns correct status for each derivation branch; explicit `status` overrides computed.
- Any filter/saved-view built against the raw `status` column (not `computed_status`) is tested with a `NULL`-status row too — `whereNotIn` silently drops `NULL` rows in SQL and will hide untouched leads from their own default view (see `data-model.md` "Verified pitfall").
- `owner()` relation loads; owner backfill matches exact names only (ambiguous/none → null).

**List view + saved views**
- Each view key filters correctly: `all, open(default), to_contact, follow_up_overdue, follow_up_today, contacted, closed, high_priority, assigned_to_me`.
- `assigned_to_me` shows only `auth` user's `owner_id` rows.
- Quick search `q` matches across name/company/role/email/phone/notes.
- Sorting whitelist rejects unknown columns (no SQL injection via `sort`), applies asc/desc.
- Pagination preserves query string; count reflects filter.
- Empty view renders the empty state; missing table renders graceful "unavailable", not 500.

**Quick actions (each: happy path + validation + authorization)**
- `markContacted`, `recordOutcome` (outcome must be in `FOLLOW_UP_OUTCOMES`), `scheduleFollowUp` (date ≥ today; +3/+7/+14 presets), `setPriority` (0–5), `setStatus` (key in `crm_lead_statuses`), `addNote` (prepends timestamped), `close` (maps reason → status + sales/follow-up outcome), `toggleFlag`.
- Invalid payloads return 422/redirect-with-errors; no partial writes.

**Bulk**
- `bulk` applies to only the given ids, inside a transaction; unknown ids rejected; authorized.

**Policy / access**
- Guest → redirected to login on all CRM routes.
- `sales` and `admin` can view list + detail + allowed quick actions.
- `delete`/admin-only actions forbidden for `sales` (403).
- Detail `show` authorizes the specific entry.

## Commands (run before claiming done)

```bash
composer test          # or: php artisan test  (config:clear runs first per composer script)
./vendor/bin/pint      # code style (Laravel Pint) — must be clean
npm run build          # Vite build must succeed (Tailwind 4)
```

Optionally target the CRM suite: `php artisan test --filter=Crm`.

## Manual verification (superpowers:verification-before-completion)

Drive the real flow, don't just trust green tests:
1. `php artisan migrate` then seed demo (fake data only) → open `/dashboard/crm/leads`.
2. Switch each saved view; confirm count + rows.
3. Open a lead detail; confirm next-action, timeline, quick links.
4. Run one quick action (e.g. schedule follow-up +7) → confirm it moves buckets/views.
5. Confirm `/dashboard/pipeline-commerciale` and `/dashboard/oggi` still work unchanged.
6. Confirm no real source-org data anywhere in the diff (grep the branch for stray names/emails before commit).

## Done-criteria checklist
- [ ] Service extracted; Today view + tests unchanged.
- [ ] Additive migration reversible; existing pipeline tests green.
- [ ] List view: views, search, sort, pagination, bulk, row actions, all states.
- [ ] Detail: highlights, next-action, timeline, cards, notes, quick links.
- [ ] Quick actions validated + authorized + tested.
- [ ] Policy enforced on every route/action.
- [ ] `pint` clean, `composer test` green, `npm run build` succeeds.
- [ ] No real source-org data in code/seeds/docs/commits.
