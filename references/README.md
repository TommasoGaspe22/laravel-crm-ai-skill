# CRM skill — index & method

Reverse-engineering di un **CRM enterprise di riferimento** (org demo, read+write su record demo) → **specifica implementabile** per un CRM **Laravel** enterprise, riusabile in qualunque progetto. Nessun branding/asset/codice/dato proprietario dell'org di riferimento: si replicano **pattern, UX e logica**.

## Come usare questa skill
1. Parti da `../SKILL.md` (entry + reuse map + guardrail + build order).
2. Consulta `definition-of-done.md` per lo **stato reale** di ogni area.
3. Per un'area specifica, apri il relativo `*-system.md`.
4. Prima di implementare in Laravel, leggi `laravel-implementation-blueprint.md` + `data-model-and-migrations.md` + `component-catalog.md`.
5. Rispetta la **priorità** (`v1-v2-v3-roadmap.md`): non tutto è V1.

## Legenda evidenza (in ogni file)
**Osservato** · **Testato dinamicamente** · **Dedotto** · **Proposto per Laravel** · **Da verificare** · **Non replicare** · **Rischio implementativo**.

## Metodo (a spirale, sperimentale)
Per ogni oggetto/schermata/flusso: osserva stato iniziale → crea dati demo `CRM-SKILL-` → prova ogni azione/stato (completi/minimi/vuoti/errori/annulla) → verifica impatti (timeline, related, filtri, Kanban) → documenta comportamento reale → traduci in Laravel → classifica V1/V2/V3 → cerca flussi collegati e ricomincia. Mai "esiste un pulsante": descrivere trigger, stato, campi, regole, feedback, conseguenze.

## Mappa file
**Meta/tracking:** `README.md` · `definition-of-done.md` · `open-questions-and-assumptions.md` · `demo-data-register.md` · `crm-architecture-overview.md` · `v1-v2-v3-roadmap.md`
**Riferimento + UX:** `source-crm-reference.md` · `screen-anatomy.md` · `objects-catalog.md`
**List view & dintorni:** `list-view-system.md` · `saved-views-system.md` · `table-system.md` · `column-management-system.md` · `bulk-actions-system.md` · `pagination-and-loading-system.md` · `split-view-system.md` · `filter-builder-system.md`
**Record page:** `record-page-system.md` · `lead-record-page.md` · `account-record-page.md` · `contact-record-page.md` · `opportunity-record-page.md` · `inline-edit-system.md` · `related-list-system.md`
**Flussi:** `activity-and-timeline-system.md` · `lead-conversion-system.md` · `opportunity-pipeline-and-kanban.md` · `pipeline-system.md` · `kanban-system.md` · `stage-transition-system.md`
**Form/dati:** `form-and-modal-system.md` · `field-catalog.md` · `validation-and-error-system.md` · `unsaved-changes-system.md`
**Laravel:** `data-model.md` · `data-model-full.md` · `data-model-and-migrations.md` · `component-catalog.md` · `app-shell-system.md` · `permissions-and-roles.md` · `laravel-implementation-blueprint.md` · `testing-strategy.md` · `modules.md` · `ux-design-system.md` · `testing.md`

> Alcuni file sono in creazione progressiva: `definition-of-done.md` indica quali esistono e a che %. I file `data-model.md/data-model-full.md/modules.md/ux-design-system.md/testing.md` sono la V1 Lead-only e confluiscono nei nuovi documenti di sistema.

## Stack Laravel target
Laravel 13 · Blade + Alpine · Tailwind · SQLite (Aruba) — nessuna nuova dipendenza pesante salvo dove indicato (es. SortableJS per Kanban, in V2). Vedi `laravel-implementation-blueprint.md`.

## Vincoli etici/tecnici
Solo record demo `CRM-SKILL-` con dati fittizi; niente config/metadata/utenti/Flow modificati; niente comunicazioni esterne; cleanup finale obbligatorio (`demo-data-register.md`).
