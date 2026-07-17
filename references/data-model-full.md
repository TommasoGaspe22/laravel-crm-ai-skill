# Relational sales-flow data model (new tables)

Extends `data-model.md` (which covers the Lead/pipeline additive columns). Here we add the **downstream relational objects**. All are **new tables** — additive, non-destructive; existing `commercial_pipeline_entries`, `leads`, and their tests are untouched. Prefix new CRM tables `crm_` to keep the namespace obvious.

Build these **in dependency order** (accounts first). Each migration is reversible. SQLite: keep FKs nullable and use `foreignId()->nullable()->constrained()->nullOnDelete()`.

## Tables

### `crm_accounts`  (Account / Company — the hub)
`id, name*, website, type (customer|prospect|partner|other), description, parent_id(self-FK nullable), phone, billing_address, shipping_address, business_area, owner_id(FK users), source_entry_id(FK commercial_pipeline_entries nullable — provenance), timestamps, softDeletes`.
Model `CrmAccount`: `hasMany contacts`, `hasMany opportunities`, `hasMany activities (morph)`, `belongsTo parent`.

### `crm_contacts`  (Contact)
`id, account_id(FK crm_accounts nullable), first_name*, last_name*, title, reports_to_id(self-FK nullable), email, phone, mailing_address, description, owner_id, source_entry_id, timestamps, softDeletes`.
Model `CrmContact`: `belongsTo account`, `belongsTo reportsTo`, `hasMany opportunities (as primary contact)`, morph activities.

### `crm_opportunities`  (Opportunity — value pipeline)
`id, account_id(FK crm_accounts)*, primary_contact_id(FK crm_contacts nullable), name*, stage*(enum), amount(decimal nullable), close_date(date)*, probability(unsigned tinyint nullable), forecast_category(nullable), next_step(nullable), description, owner_id, source_entry_id, timestamps, softDeletes`.
**Stage enum keys → labels** (config `crm_opportunity_stages`, order preserved, mirrors the org):
`qualify=Qualify, present=Present, propose=Propose, negotiate=Negotiate, won=Won, lost=Lost`.
`won`/`lost` are terminal (closed). Add `is_closed` / `is_won` accessors derived from stage. Default probability per stage is optional.
Model `CrmOpportunity`: `belongsTo account`, `belongsTo primaryContact`, `hasMany items`, morph activities.

### `crm_activities`  (Task/Call/Email/Event; replaces derived timeline)
`id, type(task|call|email|event|note)*, subject*, due_at(nullable), start_at(nullable), end_at(nullable), status(open|done|nullable), priority(nullable), owner_id, subject_type+subject_id (morphTo: lead/account/contact/opportunity), body/notes, timestamps`.
Model `CrmActivity`: `morphTo subject`, `belongsTo owner`. Query helpers: `upcoming()`, `overdue()`, `forSubject()`.

### Catalog (later phase — build only when a real SKU list exists)
- `crm_products`: `name*, product_class, code, description, family, active`.
- `crm_price_books`: `name*, description, valid_from, valid_to, active`.
- `crm_price_book_entries`: `product_id, price_book_id, unit_price, active`.
- `crm_opportunity_items`: `opportunity_id, price_book_entry_id, quantity, unit_price, total`.
V1 of the sales flow may skip these and let an Opportunity carry a free `amount`.

## Lead conversion flow (reference "Convert")

Quick action `POST /crm/leads/{entry}/convert` on a `commercial_pipeline_entry`:
1. Find-or-create `crm_accounts` by `company` (case-insensitive) → `account`.
2. Create `crm_contacts` from `lead_name`/`role`/`email`/`phone` linked to `account` (skip if a matching contact exists).
3. Optionally create `crm_opportunities` (name default `"{company} — {today}"`, `stage=qualify`, `close_date` = +30 days default, `amount` optional) linked to account + contact.
4. Copy `owner_id`; set the pipeline entry `status = converted` (add to `crm_lead_statuses`) and stamp a `crm_activities` "convert" note.
5. Wrap in a transaction; authorize; return to the new opportunity (or account).
All target fields are optional-friendly; never fail on missing lead data.

## Access & policy
One policy per object (`CrmAccountPolicy`, `CrmContactPolicy`, `CrmOpportunityPolicy`, `CrmActivityPolicy`) following the Lead pattern in `data-model.md`: `admin` + `sales` can view/create/update; `delete` admin-only; ownership scoping for "Mine/Assigned to me" views. Register all in the policy provider.

## Routing (mirror the Lead module)
Under the existing `auth` + `dashboard.` group, add `crm.` resources:
`/dashboard/crm/accounts`, `/contacts`, `/opportunities`, `/activities` (+ `/products`,`/price-books` later) — each `index` (reference-style list view) + `show` (detail) + quick-action `PATCH` endpoints. Bind route models to the `Crm*` models. Add a **Kanban by stage** for opportunities (SortableJS, deferred dependency) once stages are persisted — until then a "by stage" grouped list.

## Build order (whole flow)
1. Lead module (modules.md) + additive pipeline columns (data-model.md).
2. `crm_accounts` + `crm_contacts` (list + detail + relations).
3. `crm_opportunities` (list + detail + **stage path**; "by stage" view).
4. Lead **Convert** action wiring the three together.
5. `crm_activities` + real timeline on every detail page (replace derived timeline).
6. Catalog (products/price books/line items) — only if needed.
7. Policies + tests for each (see testing.md), then `pint` / `composer test` / `npm run build`.
