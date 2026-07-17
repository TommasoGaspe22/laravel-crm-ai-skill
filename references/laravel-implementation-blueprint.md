# Laravel implementation blueprint

**Scope:** concrete technical blueprint for implementing the CRM. Source: **Proposed for Laravel** (grounded in everything observed/tested). Don't implement until the skill is ready (DoD). Stack: Laravel 13 · Blade · Alpine · Tailwind · SQLite.

## Architecture (layers) — see `crm-architecture-overview.md`
Presentation (Blade `x-crm.*` + Alpine) · HTTP (thin controllers + FormRequest + Policy) · Application (Action classes) · Domain (Eloquent + enum/config) · Persistence (migrations/indexes) · Events/Audit (Observer → activities/audit) · Infra (storage/notifications/jobs).

## Models & relations
`CommercialPipelineEntry`(Lead) · `CrmAccount` · `CrmContact` · `CrmOpportunity` · `CrmActivity` · `AuditEvent` · (V2) `SavedView` · (V3) catalog. Relations in `data-model-and-migrations.md`. Accessors: `computed_status`(lead), `is_closed/is_won`(opp), badge classes (existing score/priority).

## Action classes (use cases)
`ConvertLead` (`lead-conversion-system.md`) · `ChangeStage`/`ChangeLeadStatus` (`stage-transition-system.md`) · `LogActivity` · `AssignOwner` · `BulkUpdate` · `CloseOpportunity` (won/lost+reason). Each: validates (or receives a validated FormRequest), authorizes (policy), runs in a transaction, writes audit/activity, returns a result + toast.

## Controllers & routes
Under `auth` + prefix `dashboard` + `crm.` (see `modules.md`). Per object: `index` (list+kanban), `show`, `store/update/destroy`, quick-action `PATCH`, `export`, `bulk`. Route-model-binding to `Crm*`. `authorize()` everywhere.

**Verified (Day 0, before writing the first controller):** as of the Laravel 11+ skeleton, the base `Controller` (`app/Http/Controllers/Controller.php`) **no longer** ships with `AuthorizesRequests`/`ValidatesRequests` by default. If the project is recent (or created from scratch following this skill), `$this->authorize(...)` fails with "Call to undefined method" until you add `use Illuminate\Foundation\Auth\Access\AuthorizesRequests;` + `use AuthorizesRequests;` to the abstract `Controller`. Check this first if `authorize()` isn't already used elsewhere in the existing project.

## FormRequest & validation
One per object/action (`Store/UpdateXRequest`, `ConvertLeadRequest`, `ChangeStageRequest`…). Rules from `field-catalog.md`/`validation-and-error-system.md`. Localized messages. Inline errors.

## Policy / Gate
One per object (`permissions-and-roles.md`). `role` middleware for admin-only.

## Events / Observer / Audit
Model Observers (Opportunity/Lead/…) → on `updated` (stage/owner/status) and actions → write `crm_activities` (or `audit_events`) for timeline+audit. A `TimelineEvent` generator centralizes creation.

## Query, filters, views, pagination
`filteredQuery(Request)` per object (reuses the `CommercialPipelineController` pattern): view (`?view=`) + `q` + sort (whitelist) + filters (`filter-builder-system.md`) + pagination (`withQueryString`). `App\Support\{Obj}ListViews`. **N+1 prevention:** eager-load owner/account; `withCount` for related badges.

## Shared services
`CommercialPipelineActionService` (extracted — next-action/bucket, reused by Today+CRM) · `TimelineBuilder` · `FilterQueryBuilder` · `CsvExporter` (existing).

## UI components
`component-catalog.md` (reference-equivalent) in `resources/views/components/crm/` + `public/css/crm.css` (tokens) + Alpine behaviors (dropdown/modal/inlineEdit/bulkSelect/drawer). No new dependency for V1; SortableJS (Kanban) and FullCalendar (events) in V2.

## File storage / search / cache / jobs
Files: `storage` (related "Files"). Global search: server-side cross-object query (V1); Scout (V2). Cache: counts/kanban (V2). Jobs/queue: large bulk operations, notifications, reminders (V2).

## Naming & conventions
Models `Crm{Object}`, tables `crm_{objects}`, routes `dashboard.crm.{objects}.*`, components `x-crm.*`, config `crm.crm_*`. Timezone `config('app.timezone')`. Consistency with existing code.

## Error handling / states
No 500s (degrades like `pipelineUnavailable`); empty/loading/error states (`ux-design-system.md`, `pagination-and-loading-system.md`); toast; 404 for a missing record.

## Accessibility & responsive
`ux-design-system.md`: aria, focus, contrast, keyboard; desktop-first, tablet scroll, essential mobile.

## Implementation order (phases)
See `SKILL.md` build order (Phase A Lead layer, Phase B relational) + `v1-v2-v3-roadmap.md`. Every phase: TDD → `pint` → `composer test` → `npm run build` → manual verification (`testing.md`, `testing-strategy.md`).

## Implementation risks (summary)
N+1; concurrency (optimistic lock); polymorphic relations (activities who/what); conversion dedup; deletion cascades; filter-query security (whitelist/binding); PII; timezone; SQLite performance (indexes, pagination, no heavy queries).
