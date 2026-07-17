# How to use the `laravel-crm-ai-skill` skill

Practical guide for using this skill. It's a **complete reverse-engineering of a reference enterprise CRM**, turned into an **implementable specification for a Laravel CRM** (built for a generic Laravel project, reusable in any app). No asset, branding, or proprietary data from the reference org: only patterns, UX, and logic were captured.

---

## 1. What it is (and isn't)

- **It is:** a product spec + UX + database + business logic + components + tests + roadmap, detailed enough that an agent can build the V1 of the CRM without reinventing the key decisions.
- **It is NOT:** already-implemented CRM code. Implementation happens by following the skill (it hasn't been written into an application repo yet).

## 2. Activation in Claude Code

- **Automatic:** working inside your Laravel project, the skill triggers on requests like *"build the CRM leads view"*, *"enterprise-style lead list"*, *"opportunity detail page"*, *"pipeline/kanban"*, *"lead conversion"*, etc. (see the `description` field in `SKILL.md`).
- **Manual:** open `SKILL.md` (entry point) → follow the pointers into `references/`.

## 3. Where to start (reading order)

1. **`SKILL.md`** — overview, guiding principle, reuse map, guardrails, build order.
2. **`references/README.md`** — full file index + method + evidence legend.
3. **`references/definition-of-done.md`** — honest current state/percentages + what's still "to verify".
4. Then whichever domain file you need (see §5).

## 4. Evidence legend (used in every file)

Every important claim is labeled: **Observed** (seen in the UI) · **Dynamically tested** (by creating/editing demo records) · **Deduced** · **Proposed for Laravel** · **To verify** · **Do not replicate** · **Implementation risk**. → Trust "Tested/Observed"; treat "Proposed for Laravel" as a recommendation; verify "To verify" items before assuming them.

## 5. Usage paths

### A) Understand the reference CRM you're replicating

`source-crm-reference.md` · `screen-anatomy.md` · `objects-catalog.md` · the various `*-system.md` files (list-view, filters, record-page, kanban, activities, conversion…).

### B) Implement V1 in Laravel (main path)

1. `laravel-implementation-blueprint.md` — architecture, layers, action/policy/observer, naming, risks.
2. `data-model-and-migrations.md` — `crm_*` tables, fields, indexes, relations (additive, non-destructive).
3. `component-catalog.md` + `ux-design-system.md` + `screen-anatomy.md` — enterprise-style Blade/Alpine components + tokens.
4. `modules.md` — the concrete Lead module (routes/controller/views/quick-actions) that every other object mirrors.
5. `permissions-and-roles.md` + `testing-strategy.md` — policy + tests.
6. `v1-v2-v3-roadmap.md` — what to build in V1 (NOT everything!) and what to defer.
7. **Build order** (in `SKILL.md`): Phase A (Lead layer: extract `CommercialPipelineActionService` + additive migration + `/crm/leads`) → Phase B (Account/Contact/Opportunity + conversion + activities).

### C) Build a single module/screen

Open the specific file: `list-view-system.md`, `record-page-system.md` (+ `lead/account/contact/opportunity-record-page.md`), `filter-builder-system.md`, `kanban-system.md`, `stage-transition-system.md`, `lead-conversion-system.md`, `activity-and-timeline-system.md`, `inline-edit-system.md`, `form-and-modal-system.md`, `validation-and-error-system.md`, `field-catalog.md`.

### D) Complete remaining tests against a reference demo org (optional)

See `definition-of-done.md` §remaining + `open-questions-and-assumptions.md`. Recreate a small demo dataset with the **`CRM-SKILL-`** prefix and fake data only (e.g. `@crm-skill.test` emails), then **clean it up** at the end (procedure in `demo-data-register.md`). Never use real accounts, real credentials, or real personal data for this — see the guardrails in `SKILL.md`.

## 6. Golden rules (from the guardrails)

- **Do NOT** clone the reference enterprise platform itself (Flow, report builder, AI, complex sharing) → `v1-v2-v3-roadmap.md` §Do not replicate.
- **Reuse** `commercial_pipeline_entries` + the existing next-action engine; add `crm_*` tables **additively and reversibly**; don't break the existing pipeline/today/import-export.
- **Server-side first** (filters/sort/pagination in SQL, no N+1); Alpine only for interactions.
- **Policy on every route/action**; server-side validation; **no real reference-CRM data** in the repo/commits/seeds.
- Fixed stack: Laravel 13 · Blade · Alpine · Tailwind · SQLite. New dependencies only in V2 (SortableJS for Kanban, FullCalendar for events).

## 7. Quick file map (by need)

| You need… | Open |
|---|---|
| Index + method | `references/README.md` |
| Status/progress | `references/definition-of-done.md` |
| The real reference anatomy | `references/source-crm-reference.md`, `screen-anatomy.md` |
| Objects + relationships | `references/objects-catalog.md`, `data-model-and-migrations.md` |
| UI components | `references/component-catalog.md`, `ux-design-system.md` |
| Lists/filters/kanban | `references/list-view-system.md`, `filter-builder-system.md`, `kanban-system.md` |
| Record/detail | `references/record-page-system.md` (+ per-object), `inline-edit-system.md` |
| Flows | `references/lead-conversion-system.md`, `stage-transition-system.md`, `activity-and-timeline-system.md` |
| Implementation | `references/laravel-implementation-blueprint.md`, `modules.md` |
| Permissions/testing | `references/permissions-and-roles.md`, `testing-strategy.md` |
| Priorities | `references/v1-v2-v3-roadmap.md` |

## 8. Suggested next step

Ask your AI assistant, inside your Laravel project: *"implement Phase A of the CRM following the laravel-crm-ai-skill skill"* → it will start with extracting the service + an additive migration + `/crm/leads`, in TDD, without touching existing functionality.
