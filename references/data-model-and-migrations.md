# Data model & migrations (consolidated)

**Scope:** complete, migration-ready data spec for the Laravel CRM. Consolidates `objects-catalog.md` + `data-model.md` + `data-model-full.md` + `field-catalog.md`. Source: **Observed/Tested** (real fields/picklists) + **Proposed for Laravel**. All new `crm_*` tables are **additive and reversible**; they don't touch the existing `commercial_pipeline_entries`/`leads`.

## Conventions
- Prefix `crm_`; PK `id` bigint; timestamps + `softDeletes` where it makes sense; nullable FK `->constrained()->nullOnDelete()` (SQLite-friendly); indexes on FKs + filtered/sorted fields; enum/picklist as `string` + validation via `config('crm.*')`.

## Objects (business → table → key fields → relations)

### Lead → `commercial_pipeline_entries` (existing) + additive
See `data-model.md`. Add: `email, phone, status, owner_id, lead_source, website, linkedin_url` (nullable). Status: `new/contacted/nurturing/unqualified/converted` (**Observed** real ladder) + accessor `computed_status`. Core fields already present (lead_name, company, role, business_area, address, priority, lead_score, follow_up_*, sales_outcome…). Indexes: `status, owner_id, follow_up_date`.

### `crm_accounts` (Account)
`name*, website, type(customer|prospect|partner|other), description, parent_id(self FK), phone, billing_address, shipping_address, business_area, owner_id, source_entry_id, timestamps, softDeletes`. Rel: `hasMany contacts/opportunities/activities(morph)`, `belongsTo parent`. Indexes: `name, owner_id, parent_id`. (**Observed** record fields + Type picklist.)

### `crm_contacts` (Contact)
`account_id(FK), first_name*, last_name*, title, reports_to_id(self FK), email, phone, mailing_address, description, owner_id, source_entry_id, timestamps, softDeletes`. Rel: `belongsTo account/reportsTo`, `hasMany opportunities(primary)`, `belongsToMany opportunities` via `crm_opportunity_contact_roles`(V2). Indexes: `account_id, email, owner_id`.

### `crm_opportunities` (Opportunity)
`account_id(FK)*, primary_contact_id(FK), name*, stage*(qualify|present|propose|negotiate|won|lost), amount(decimal 15,2), close_date(date)*, probability(tinyint), forecast_category, next_step, loss_reason, description, owner_id, source_entry_id, closed_at, timestamps, softDeletes`. Accessor `is_closed/is_won`. Rel: `belongsTo account/primaryContact`, `hasMany items`(V3), morph activities. Indexes: `account_id, stage, owner_id, close_date`. (**Observed/Tested** stages + fields.)

### `crm_activities` (Task/Call/Event/Email/Note) — see `activity-and-timeline-system.md`
`type*, subject, body, status, priority, due_at, start_at, end_at, who_type+who_id(morph), what_type+what_id(morph), owner_id, created_by, timestamps`. Indexes: `who_*, what_*, owner_id, due_at, type`.

### `audit_events` (system audit/timeline)
`subject_type+subject_id(morph), event(convert|stage_change|owner_change|status_change|create|delete…), from_json, to_json, user_id, created_at`. Indexes: morph + `created_at`. (Alternative: `crm_activities` type=system.)

### `saved_views` (V2) — `saved-views-system.md`
`object, name, owner_id, is_shared, filters_json, columns_json, sort_json, is_default`. Indexes: `object, owner_id`.

### Catalog (V3) — `crm_products, crm_price_books, crm_price_book_entries, crm_opportunity_items`
See `data-model-full.md` §Catalog. Only if a real SKU list is needed.

## Relations (summary)
`Account 1—* Contact` · `Account 1—* Opportunity` · `Contact 1—* Opportunity`(primary) + `* —* `(roles) · `Lead —(convert)→ Account+Contact+Opportunity` · `Activity *—morph→ (Lead|Account|Contact|Opportunity)` who/what · `AuditEvent *—morph→ any`.

## Migration order (additive)
1. additive on `commercial_pipeline_entries` (Lead layer). 2. `crm_accounts`. 3. `crm_contacts`. 4. `crm_opportunities`. 5. `crm_activities`. 6. `audit_events`. 7. (V2) `saved_views`, `crm_opportunity_contact_roles`. 8. (V3) catalog.
Every migration: additive `up()` + `down()` that removes the FK/indexes/table. Soft backfill (derived status, owner name matching).

## Indexes & performance (Implementation risk)
Index: all FKs, `stage`, `status`, `owner_id`, dates (`close_date`, `follow_up_date`, `due_at`). List queries with targeted `select` + eager-load (owner/account) to avoid N+1. `config('app.timezone')` on all dates. SQLite: simple ALTERs; nullable FKs.

## Sensitive data & soft delete
Email/phone = PII → don't log in plaintext; soft delete on Lead/Account/Contact/Opportunity (trash/restore, see `record-page-system.md` deletion). Access audit (`permissions-and-roles.md`).

## Open questions
Secondary picklist values (Account type, Lead source, Industry, Forecast category, Loss reason) — **to verify**; M:N contact roles; product catalog (if needed).
