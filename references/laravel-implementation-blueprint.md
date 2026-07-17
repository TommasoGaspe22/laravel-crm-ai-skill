# Laravel implementation blueprint

**Scope:** blueprint tecnico concreto per implementare il CRM. Fonte: **Proposto per Laravel** (fondato su tutto l'osservato/testato). Non implementare finché la skill non è pronta (DoD). Stack: Laravel 13 · Blade · Alpine · Tailwind · SQLite (Aruba).

## Architettura (strati) — vedi `crm-architecture-overview.md`
Presentazione (Blade `x-crm.*` + Alpine) · HTTP (controller sottili + FormRequest + Policy) · Applicazione (Action classes) · Dominio (Eloquent + enum/config) · Persistenza (migration/indici) · Eventi/Audit (Observer → activities/audit) · Infra (storage/notifiche/jobs).

## Modelli & relazioni
`CommercialPipelineEntry`(Lead) · `CrmAccount` · `CrmContact` · `CrmOpportunity` · `CrmActivity` · `AuditEvent` · (V2) `SavedView` · (V3) catalogo. Relazioni in `data-model-and-migrations.md`. Accessor: `computed_status`(lead), `is_closed/is_won`(opp), badge classes (score/priority esistenti).

## Action classes (casi d'uso)
`ConvertLead` (`lead-conversion-system.md`) · `ChangeStage`/`ChangeLeadStatus` (`stage-transition-system.md`) · `LogActivity` · `AssignOwner` · `BulkUpdate` · `CloseOpportunity` (won/lost+motivo). Ognuna: valida (o riceve FormRequest validato), autorizza (policy), esegue in transazione, scrive audit/activity, ritorna risultato + toast.

## Controller & route
Sotto `auth` + prefix `dashboard` + `crm.` (vedi `modules.md`). Per oggetto: `index` (list+kanban), `show`, `store/update/destroy`, quick-action `PATCH`, `export`, `bulk`. Route-model-binding ai `Crm*`. `authorize()` ovunque.

**Verificato (Day 0, prima di scrivere il primo controller):** dal skeleton Laravel 11+ il `Controller` base (`app/Http/Controllers/Controller.php`) **non** include più `AuthorizesRequests`/`ValidatesRequests` di serie. Se il progetto è recente (o creato da zero seguendo questa skill), `$this->authorize(...)` fallisce con "Call to undefined method" finché non si aggiunge `use Illuminate\Foundation\Auth\Access\AuthorizesRequests;` + `use AuthorizesRequests;` al `Controller` astratto. Controllare questo per primo se `authorize()` non è già usato altrove nel progetto esistente.

## FormRequest & validazione
Una per oggetto/azione (`Store/UpdateXRequest`, `ConvertLeadRequest`, `ChangeStageRequest`…). Regole da `field-catalog.md`/`validation-and-error-system.md`. Messaggi IT. Inline errors.

## Policy / Gate
Una per oggetto (`permissions-and-roles.md`). `role` middleware per admin-only.

## Eventi / Observer / Audit
Model Observers (Opportunity/Lead/…) → su `updated`(stage/owner/status) e azioni → scrivono `crm_activities`(o `audit_events`) per timeline+audit. Generator `TimelineEvent` centralizza la creazione.

## Query, filtri, viste, paginazione
`filteredQuery(Request)` per oggetto (riusa pattern `CommercialPipelineController`): view(`?view=`) + `q` + sort(whitelist) + filtri (`filter-builder-system.md`) + paginazione (`withQueryString`). `App\Support\{Obj}ListViews`. **N+1 prevention:** eager-load owner/account; `withCount` per related badges.

## Servizi condivisi
`CommercialPipelineActionService` (estratto — next-action/bucket, riuso Oggi+CRM) · `TimelineBuilder` · `FilterQueryBuilder` · `CsvExporter`(esistente).

## Componenti UI
`component-catalog.md` (equivalente al riferimento) in `resources/views/components/crm/` + `public/css/crm.css` (token) + Alpine behaviors (dropdown/modal/inlineEdit/bulkSelect/drawer). Nessuna dipendenza nuova V1; SortableJS (Kanban) e FullCalendar (eventi) in V2.

## File storage / search / cache / jobs
File: `storage` (related "File"). Search globale: query server-side cross-oggetto (V1); Scout (V2). Cache: conteggi/kanban (V2). Jobs/queue: bulk grandi, notifiche, reminder (V2).

## Naming & convenzioni
Modelli `Crm{Object}`, tabelle `crm_{objects}`, route `dashboard.crm.{objects}.*`, componenti `x-crm.*`, config `crm.crm_*`. Timezone `config('app.timezone')`. Coerenza con codice esistente.

## Error handling / stati
No-500 (degrada come `pipelineUnavailable`); empty/loading/error states (`ux-design-system.md`, `pagination-and-loading-system.md`); toast; 404 record mancante.

## Accessibilità & responsive
`ux-design-system.md`: aria, focus, contrasto, tastiera; desktop-first, tablet scroll, mobile essenziale.

## Ordine di implementazione (fasi)
Vedi `SKILL.md` build order (Fase A Lead layer, Fase B relazionale) + `v1-v2-v3-roadmap.md`. Ogni fase: TDD → `pint` → `composer test` → `npm run build` → verifica manuale (`testing.md`, `testing-strategy.md`).

## Rischi implementativi (sintesi)
N+1; concorrenza (optimistic lock); polimorfiche (activities who/what); dedup conversione; cascata eliminazioni; sicurezza query filtri (whitelist/binding); PII; timezone; performance SQLite/Aruba (indici, paginazione, no query pesanti).
