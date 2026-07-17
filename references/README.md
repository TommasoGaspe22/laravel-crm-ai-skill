# CRM skill — index & method

Reverse-engineering of a **reference enterprise CRM** (demo org, read+write on demo records) → **implementable specification** for an enterprise **Laravel** CRM, reusable in any project. No branding/asset/code/proprietary data from the reference org: only **patterns, UX, and logic** are replicated.

## How to use this skill
1. Start from `../SKILL.md` (entry + reuse map + guardrails + build order).
2. Check `definition-of-done.md` for the **real status** of each area.
3. For a specific area, open the matching `*-system.md`.
4. Before implementing in Laravel, read `laravel-implementation-blueprint.md` + `data-model-and-migrations.md` + `component-catalog.md`.
5. Respect the **priority order** (`v1-v2-v3-roadmap.md`): not everything is V1.

## Evidence legend (in every file)
**Observed** · **Dynamically tested** · **Deduced** · **Proposed for Laravel** · **To verify** · **Do not replicate** · **Implementation risk**.

## Method (spiral, experimental)
For every object/screen/flow: observe the initial state → create `CRM-SKILL-` demo data → try every action/state (full/minimal/empty/error/cancel) → verify impacts (timeline, related lists, filters, Kanban) → document the real behavior → translate into Laravel → classify V1/V2/V3 → look for connected flows and start again. Never just "there's a button": describe the trigger, state, fields, rules, feedback, and consequences.

## File map
**Meta/tracking:** `README.md` · `definition-of-done.md` · `open-questions-and-assumptions.md` · `demo-data-register.md` · `crm-architecture-overview.md` · `v1-v2-v3-roadmap.md`
**Reference + UX:** `source-crm-reference.md` · `screen-anatomy.md` · `objects-catalog.md`
**List view & related:** `list-view-system.md` · `saved-views-system.md` · `table-system.md` · `column-management-system.md` · `bulk-actions-system.md` · `pagination-and-loading-system.md` · `split-view-system.md` · `filter-builder-system.md`
**Record page:** `record-page-system.md` · `lead-record-page.md` · `account-record-page.md` · `contact-record-page.md` · `opportunity-record-page.md` · `inline-edit-system.md` · `related-list-system.md`
**Flows:** `activity-and-timeline-system.md` · `lead-conversion-system.md` · `opportunity-pipeline-and-kanban.md` · `pipeline-system.md` · `kanban-system.md` · `stage-transition-system.md`
**Forms/data:** `form-and-modal-system.md` · `field-catalog.md` · `validation-and-error-system.md` · `unsaved-changes-system.md`
**Laravel:** `data-model.md` · `data-model-full.md` · `data-model-and-migrations.md` · `component-catalog.md` · `app-shell-system.md` · `permissions-and-roles.md` · `laravel-implementation-blueprint.md` · `testing-strategy.md` · `modules.md` · `ux-design-system.md` · `testing.md`

> Some files were built progressively: `definition-of-done.md` shows which exist and at what %. The files `data-model.md`/`data-model-full.md`/`modules.md`/`ux-design-system.md`/`testing.md` are the Lead-only V1 pass and feed into the newer system-level documents.

## Target Laravel stack
Laravel 13 · Blade + Alpine · Tailwind · SQLite — no new heavy dependency except where noted (e.g. SortableJS for Kanban, in V2). See `laravel-implementation-blueprint.md`.

## Ethical/technical constraints
Only `CRM-SKILL-` demo records with fake data; no config/metadata/user/Flow changes; no external communications; mandatory final cleanup (`demo-data-register.md`).
