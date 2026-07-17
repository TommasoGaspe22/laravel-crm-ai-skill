# CRM architecture overview (reference CRM → Laravel)

**Goal:** give a bird's-eye view of the CRM architecture being replicated, as a map for all the detail files. **Scope:** concepts and layers, not line-by-line implementation (that's in `laravel-implementation-blueprint.md`).

## 1. Conceptual model (B2B CRM domain)

```
                 ┌─────────┐   convert     ┌──────────┐ 1  ┌────────────┐
   inbound ─────▶│  Lead   │──────────────▶│  Account │───▶│  Contact   │
                 └─────────┘               └────┬─────┘ *  └─────┬──────┘
                      │                         │ 1              │ *
                      │                         ▼ *              │
                      │                   ┌─────────────┐        │
                      └──────────────────▶│ Opportunity │◀───────┘  (primary contact)
                                          └──────┬──────┘
                                                 │ * (stages/pipeline)
     ┌────────────────────────────────────────┴──────────────┐
     ▼ (polymorphic "related to")                             ▼
┌──────────┐   ┌────────┐   ┌───────┐                    ┌──────────┐
│ Activity │   │  Note  │   │ File  │  ── on Lead/Account/Contact/Opportunity
└──────────┘   └────────┘   └───────┘
```

- **Lead** = unqualified top-of-funnel. **Observed**: states `New→Contacted→Nurturing→Unqualified→Converted`.
- **Account** = a company (hub). **Contact** = a person at a company. **Opportunity** = a value deal with **stages** (`Qualify→…→Closed Won/Lost` — Observed).
- **Activity/Note/File** = polymorphically related to any object → feed the **timeline** and **related lists**.
- **Conversion**: Lead → creates Account+Contact+Opportunity in a transaction (**Observed** modal).

## 2. Laravel application layers (Proposed for Laravel)

| Layer | Responsibility | Key components |
|---|---|---|
| **Presentation** | enterprise-like UI | Blade components (`x-crm.*`), Alpine for interactions, Tailwind + `crm.css` tokens |
| **HTTP** | routing, requests, authorization | Thin controllers, Form Requests (validation), Policies |
| **Application** | use cases / actions | Action classes (e.g. `ConvertLead`, `ChangeStage`, `LogActivity`), Services where needed |
| **Domain** | entities + rules | Eloquent models (`CrmAccount`, `CrmContact`, `CrmOpportunity`, `CrmActivity`, …), enums/config (statuses, stages) |
| **Persistence** | schema + queries | Additive migrations, indexes, query-builder filters, soft delete |
| **Events/Audit** | timeline + audit | Model events/Observers → `crm_activities`/`audit_events` (timeline generator) |
| **Infrastructure** | files, notifications, jobs | Storage (files), Notifications (reminders/follow-ups), Jobs (async) |

**Guiding principle:** server-rendered + progressive enhancement (Alpine). No SPA/React. Server-side queries (filters/sort/pagination) with attention to N+1 and indexes. See `laravel-implementation-blueprint.md`.

## 3. Coexistence with existing code (your Laravel project)
- Keep `commercial_pipeline_entries` + `/crm/leads` (operational Lead V1, already specced in `modules.md`) as the **Lead layer**.
- Add the relational objects as **new `crm_*` tables** (additive, non-destructive): `crm_accounts/contacts/opportunities/activities/…`.
- Bridge = **Convert** action. Details in `data-model-full.md` + `lead-conversion-system.md`.
- **Implementation risk:** temporary duplication between the flat pipeline and the relational objects; govern it with conversion and clear views. Don't normalize destructively.

## 4. Module map → detail files
- App shell/navigation → `app-shell-system.md`
- Lists → `list-view-system.md`, `table-system.md`, `saved-views-system.md`, `column-management-system.md`, `bulk-actions-system.md`, `pagination-and-loading-system.md`, `split-view-system.md`, `filter-builder-system.md`
- Record → `record-page-system.md` + per-object + `inline-edit-system.md`, `related-list-system.md`
- Flows → `activity-and-timeline-system.md`, `lead-conversion-system.md`, `pipeline-system.md`, `kanban-system.md`, `stage-transition-system.md`
- Forms/data → `form-and-modal-system.md`, `field-catalog.md`, `validation-and-error-system.md`, `unsaved-changes-system.md`, `data-model-and-migrations.md`
- Design → `component-catalog.md`, `screen-anatomy.md`
- Cross-cutting → `permissions-and-roles.md`, `testing-strategy.md`, `laravel-implementation-blueprint.md`, `v1-v2-v3-roadmap.md`

## 5. Architectural priority (summary — detail in the roadmap)
- **V1:** Lead + Account + Contact + Opportunity (stages + Path), list view (table, sort, search, basic filters, basic saved views, essential row/bulk actions), record page (sections, inline edit, related, basic timeline), Lead conversion, basic activities, owner/staff policy, seed+tests.
- **V2:** Kanban drag&drop, advanced filter builder (AND/OR/groups), configurable+resizable columns, split view, rich notes/files, basic forecast, reminders.
- **V3:** products/price-books/line items, teams, advanced forecasting, granular audit, public API.
- **Do not replicate:** Flow/automation builder, report builder, built-in AI, complex sharing rules, advanced multi-currency.
