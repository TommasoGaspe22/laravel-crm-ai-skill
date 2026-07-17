# Data model & migrations (consolidato)

**Scope:** specifica dati completa e migration-ready per il CRM Laravel. Consolida `objects-catalog.md` + `data-model.md` + `data-model-full.md` + `field-catalog.md`. Fonte: **Osservato/Testato** (campi/picklist reali) + **Proposto per Laravel**. Tutte le nuove tabelle `crm_*` sono **additive e reversibili**; non toccano `commercial_pipeline_entries`/`leads` esistenti.

## Convenzioni
- Prefisso `crm_`; PK `id` bigint; timestamps + `softDeletes` dove ha senso; FK nullable `->constrained()->nullOnDelete()` (SQLite-friendly); indici su FK + campi filtrati/ordinati; enum/picklist come `string` + validazione via `config('crm.*')`.

## Oggetti (business → tabella → campi chiave → relazioni)

### Lead → `commercial_pipeline_entries` (esistente) + additive
Vedi `data-model.md`. Add: `email, phone, status, owner_id, lead_source, website, linkedin_url` (nullable). Status: `nuovo/contattato/nurturing/non_qualificato/convertito` (**Osservato** scala reale) + accessor `computed_status`. Campi anagrafici già presenti (lead_name, company, role, business_area, address, priority, lead_score, follow_up_*, sales_outcome…). Indici: `status, owner_id, follow_up_date`.

### `crm_accounts` (Account)
`name*, website, type(cliente|potenziale_cliente|partner|altro), description, parent_id(self FK), phone, billing_address, shipping_address, business_area, owner_id, source_entry_id, timestamps, softDeletes`. Rel: `hasMany contacts/opportunities/activities(morph)`, `belongsTo parent`. Indici: `name, owner_id, parent_id`. (**Osservato** campi record + Tipo picklist.)

### `crm_contacts` (Referente)
`account_id(FK), first_name*, last_name*, title, reports_to_id(self FK), email, phone, mailing_address, description, owner_id, source_entry_id, timestamps, softDeletes`. Rel: `belongsTo account/reportsTo`, `hasMany opportunities(primary)`, `belongsToMany opportunities` via `crm_opportunity_contact_roles`(V2). Indici: `account_id, email, owner_id`.

### `crm_opportunities` (Opportunità)
`account_id(FK)*, primary_contact_id(FK), name*, stage*(qualifica|presentazione|proposta|negoziazione|vinta|persa), amount(decimal 15,2), close_date(date)*, probability(tinyint), forecast_category, next_step, loss_reason, description, owner_id, source_entry_id, closed_at, timestamps, softDeletes`. Accessor `is_closed/is_won`. Rel: `belongsTo account/primaryContact`, `hasMany items`(V3), morph activities. Indici: `account_id, stage, owner_id, close_date`. (**Osservato/Testato** fasi + campi.)

### `crm_activities` (Task/Call/Event/Email/Note) — vedi `activity-and-timeline-system.md`
`type*, subject, body, status, priority, due_at, start_at, end_at, who_type+who_id(morph), what_type+what_id(morph), owner_id, created_by, timestamps`. Indici: `who_*, what_*, owner_id, due_at, type`.

### `audit_events` (audit/timeline di sistema)
`subject_type+subject_id(morph), event(convert|stage_change|owner_change|status_change|create|delete…), from_json, to_json, user_id, created_at`. Indici: morph + `created_at`. (Alternativa: `crm_activities` type=system.)

### `saved_views` (V2) — `saved-views-system.md`
`object, name, owner_id, is_shared, filters_json, columns_json, sort_json, is_default`. Indici: `object, owner_id`.

### Catalogo (V3) — `crm_products, crm_price_books, crm_price_book_entries, crm_opportunity_items`
Vedi `data-model-full.md` §Catalog. Solo se serve un vero listino SKU.

### Config/enum (in `config/crm.php`, non tabelle in V1)
`crm_lead_statuses`, `crm_lead_ratings`(caldo/tiepido/freddo), `crm_account_types`, `crm_lead_sources`, `crm_industries`, `crm_opportunity_stages`, `crm_forecast_categories`, `crm_activity_types`, `crm_loss_reasons`. (Valori picklist secondari = **Da verificare** aprendo le picklist reali.)

## Relazioni (riepilogo)
`Account 1—* Contact` · `Account 1—* Opportunity` · `Contact 1—* Opportunity`(primary) + `* —* `(ruoli) · `Lead —(convert)→ Account+Contact+Opportunity` · `Activity *—morph→ (Lead|Account|Contact|Opportunity)` who/what · `AuditEvent *—morph→ any`.

## Migration order (additivo)
1. additive su `commercial_pipeline_entries` (Lead layer). 2. `crm_accounts`. 3. `crm_contacts`. 4. `crm_opportunities`. 5. `crm_activities`. 6. `audit_events`. 7. (V2) `saved_views`, `crm_opportunity_contact_roles`. 8. (V3) catalogo.
Ogni migration: `up()` additiva + `down()` che rimuove FK/indici/tabella. Backfill soft (status derivato, owner match nome).

## Indici & performance (Rischio implementativo)
Indicizzare: tutti gli FK, `stage`, `status`, `owner_id`, date (`close_date`, `follow_up_date`, `due_at`). Query liste con `select` mirati + eager-load (owner/account) per evitare N+1. Timezone `config('app.timezone')` su tutte le date. SQLite: ALTER semplici; FK nullable.

## Dati sensibili & soft delete
Email/telefono = PII → non loggare in chiaro; soft delete su Lead/Account/Contact/Opportunity (cestino/ripristino, `record-page-system.md` eliminazione). Audit degli accessi (`permissions-and-roles.md`).

## Open questions
Valori picklist secondari (Tipo account, Fonte lead, Settore, Categoria previsione, Motivo persa) — **Da verificare**; ruoli referenti M:N; catalogo prodotti (se necessario).
