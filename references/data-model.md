# Data model, migration, status, ownership, policy

## 1. Current `commercial_pipeline_entries` (as-is)

Fillable (see `app/Models/CommercialPipelineEntry.php`):
`source_row, contact_method, contact_owner, contacted(bool), lead_name, address, company, business_area, role, priority(0–5 int), lead_notes, analysis_appointment_set(bool), follow_up_done(bool), follow_up_date(date), follow_up_method, follow_up_outcome, second_follow_up_done(bool), second_follow_up_date(date), second_follow_up_method, second_follow_up_outcome, analysis_feedback, lead_score(1–10 int), sales_appointment_set(bool), sales_outcome, contract_value(decimal), is_flagged(bool)`.

Follow-up outcome constants (already defined): `ANALYSIS APPT SET`, `NO RESPONSE`, `NOT INTERESTED`, `NOT YET ASSESSED`, `ANALYSIS APPT PENDING` (`FOLLOW_UP_OUTCOMES`). Sales outcome values in use: `WON`, `LOST` (else pending).

**Gaps for the reference columns:** no `email`, no `phone`, no explicit `status`, owner is free-text `contact_owner` (no FK), no `lead_source`/`website`/`linkedin_url`.

## 2. Additive migration (nullable + reversible)

One migration `add_crm_fields_to_commercial_pipeline_entries_table`:

```php
Schema::table('commercial_pipeline_entries', function (Blueprint $t) {
    $t->string('email', 190)->nullable()->after('address');
    $t->string('phone', 40)->nullable()->after('email');
    $t->string('status', 30)->nullable()->after('phone');        // explicit app-level status
    $t->foreignId('owner_id')->nullable()->after('contact_owner')
        ->constrained('users')->nullOnDelete();
    // optional (guard behind Schema::hasColumn in views if you skip them):
    $t->string('lead_source', 60)->nullable()->after('contact_method');
    $t->string('website', 190)->nullable()->after('email');
    $t->string('linkedin_url', 190)->nullable()->after('website');
    $t->index('status');
    $t->index('owner_id');
});
```

`down()` drops the FK, indexes, and columns. Add all new columns to `$fillable` and cast nothing new except keep strings; `owner_id` stays int. **Do not** change existing casts.

Because SQLite is the driver, keep each `ALTER` simple (SQLite supports add-column; `foreignId(...)->constrained()` works via Laravel's grammar). Add the FK column without `constrained()` if a batch alter causes SQLite trouble — a nullable `unsignedBigInteger('owner_id')->nullable()` + separate `index` is an acceptable fallback (document it).

**Verified pitfall (`down()` on SQLite):** SQLite has no native `ALTER TABLE DROP COLUMN`/`DROP INDEX` — Laravel's grammar rebuilds the whole table per `Schema::table()` call. If a single call both drops an indexed column (`dropConstrainedForeignId`/`dropColumn`) *and* a separate index that references it, or if `dropColumn` runs while a plain `$table->index(...)` on that same column is still in place, the rebuild throws `no such column` referencing the just-dropped column. Fix confirmed by testing: (1) don't add a redundant `$t->index('owner_id')` — `constrained()` already indexes the FK column; (2) in `down()`, drop any remaining named index (`dropIndex(['status'])`) in its **own** `Schema::table()` call, *before* the call that drops the column itself. Always exercise `migrate` → `migrate:rollback` → `migrate` in a real test, not just `up()`, before calling a migration done.

## 3. Lead status — enum + derivation

Define the vocabulary once, in `config/crm.php` (extend the existing `lead_statuses` idea; the current keys are `new/contacted/qualified/converted/lost`). Use this CRM-specific set (superset):

```php
'crm_lead_statuses' => [
    'new'            => 'New',              // not contacted
    'contacted'      => 'Contacted',        // working
    'nurturing'      => 'Nurturing',
    'appointment'    => 'Appointment',      // analysis/sales appointment set
    'won'            => 'Won',              // sales_outcome WON
    'lost'           => 'Lost',             // sales_outcome LOST
    'not_interested' => 'Not interested',   // follow_up_outcome NOT INTERESTED
],
```

**Derivation (fallback when `status` is null)** — add an accessor `getComputedStatusAttribute()` on the model, priority order:
1. `sales_outcome === 'WON'` → `won`
2. `sales_outcome === 'LOST'` → `lost`
3. `follow_up_outcome` or `second_follow_up_outcome === 'NOT INTERESTED'` → `not_interested`
4. `analysis_appointment_set || sales_appointment_set` → `appointment`
5. `follow_up_outcome === 'ANALYSIS APPT PENDING'` or any follow-up done → `nurturing`
6. `contacted === true` → `contacted`
7. else → `new`

Rule: **explicit `status` wins; otherwise show computed.** The "change status" quick action writes the explicit `status`. This keeps old rows meaningful with zero backfill while letting the team override.

An "open lead" (for the default view) = `computed_status ∉ {won, lost, not_interested}` — equivalently the existing Today-view predicate: `sales_outcome` null or not in `{WON, LOST}` **and** not marked not-interested.

**Verified pitfall (SQL `NOT IN` vs. `NULL`):** because `status` is nullable and most rows will have `status = NULL` until an explicit "change status" action runs, a saved-view filter written as `whereNotIn('status', ['won','lost','not_interested'])` silently excludes every `NULL`-status row too — SQL's `NULL NOT IN (...)` evaluates to unknown, not true. That means a brand-new, never-touched lead disappears from its own default "Open leads" view. Confirmed by testing. Always pair it with an explicit `whereNull`: `$q->where(fn ($w) => $w->whereNull('status')->orWhereNotIn('status', [...]))`. Apply the same care anywhere else a saved view or filter is built against `status` directly instead of `computed_status` (see `saved-views-system.md`, `list-view-system.md`).

## 4. Ownership & backfill

- Keep legacy `contact_owner` (text) for display/CSV compatibility.
- New `owner_id` FK → `users`. Backfill in the migration (or a one-off command): for each row, match `contact_owner` (trimmed, case-insensitive) against `users.name`; set `owner_id` when unambiguous, else leave null. Never guess.
- "Assigned to me" view = `where('owner_id', auth()->id())`.

## 5. Access policy

Create `App\Policies\CommercialPipelineEntryPolicy` and register it.

- `viewAny`, `view`, `update`, `create`: allowed for `admin` and `sales` (`User::isSales()` — returns true for role in `{admin, sales}`).
- `delete`: `admin` only (sales should archive/close, not hard-delete). Confirm with product; default to admin-only to be safe.
- `assign` (change owner) / `bulk`: `admin` only, OR sales limited to assigning to self — pick admin-only for V1 simplicity.
- The controller must call `$this->authorize(...)` on show/update/quick-actions/bulk. List index filters by policy scope (all rows for staff; "assigned to me" is a user-chosen filter, not a hard restriction, unless product wants sales to see only their own — if so, scope in `viewAny`).

Routes already sit behind `auth`. CRM routes additionally go through the policy; no new middleware needed beyond the existing `role:admin` for admin-only endpoints if you prefer route-level gating.

## 6. Timezone & dates

Use the app timezone (`config('app.timezone')`) consistently. Follow-up buckets compare `startOfDay()` in that timezone (as the Today controller already does). Guard against null/invalid dates: null follow-up date → "no follow-up" bucket, never a fatal error.
