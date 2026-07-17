---
name: laravel-crm-ai-skill
description: Use when building, extending, or reviewing a Laravel CRM that needs an enterprise-style sales flow (Lead, Account, Contact, Opportunity with stages, Products/Price Books, Activities/Calendar) rebranded for your own app. Triggers on requests about /crm, list views, saved views, record detail pages, opportunity stages/pipeline, lead conversion, activities timeline, next-action queue, bulk/row actions, status/priority badges, or making a CRM feel like a polished enterprise product.
---

# CRM build framework (Laravel, enterprise-style sales flow)

## Overview

This skill is the implementation playbook for turning your Laravel app into an operational CRM that replicates a reference enterprise **sales flow**, rebranded for your own app. Scope (captured read-only from the live reference org): **Lead → (Convert) → Account + Contact + Opportunity → Products/Price Books → Activities/Calendar.** It captures the real reference UX, the data mapping, the reuse points in the existing code, and an incremental plan.

**Core principle:** Do NOT clone the reference platform itself (no Flows, report builder, admin, AI). Replicate the *sales objects and their relationships*. Start from the *existing* `commercial_pipeline_entries` + next-action engine for the Lead layer, then add the downstream relational objects as **new, additive** tables. Reuse before you add. Every change is additive and reversible; existing pipeline pages, imports, exports, and tests must keep working.

Stack (fixed — match it, don't introduce new frameworks): **Laravel 13, PHP 8.3, Blade, Alpine.js 3, Tailwind 4 (Vite), SQLite, session auth with `admin`/`sales` roles.** No Livewire, no Inertia, no React, no new JS libraries.

## When to use

- Building any `/crm` list view, saved views, filters drawer, bulk/row actions.
- Building any record detail page, timeline, or quick actions.
- Building Account / Contact / Opportunity (with stages) / Activities, or the Lead **Convert** flow.
- Adding CRM fields, status/stage/priority badges, or the opportunity stage path/Kanban.
- Any task described as "make it feel like a polished enterprise CRM" for your sales team.

**When NOT to use:** inbound funnel/assessment lead work lives in `App\Models\Lead` + `/dashboard/leads` (different table, keep separate). Enterprise-platform-only features (Flows, reports, permissions, AI assistant) — out of scope. Invoices — out of scope for V1.

## Read these in order (the framework)

> **Full file map + method + evidence legend:** `references/README.md`. Progress & honest %: `references/definition-of-done.md`. Deep reverse-engineering docs (per-system) live in `references/*-system.md` and per-object record pages. The list below is the core reading path; the README indexes everything.

1. `references/source-crm-reference.md` — the real "All Open Leads" list-view anatomy + reference→app concept/field mapping + record-page anatomy. **Read first** for the UX target.
2. `references/objects-catalog.md` — the **full sales flow** captured from the org: every object (Account, Contact, Opportunity + real stages, Products/Pricebooks, Activities), its real columns/fields/actions, relationships, and how they coexist with the existing pipeline. Read for the *what*.
3. `references/data-model.md` — Lead layer: pipeline additive columns, `status` enum + derivation, `owner_id` backfill, policy.
4. `references/data-model-full.md` — the downstream relational schema (`crm_accounts/contacts/opportunities/activities`…), stage enum, the **Convert** flow, per-object policies, and the whole-flow build order. Read for the *how* (data).
5. `references/screen-anatomy.md` — **pixel-level anatomy** of every screen archetype (list view, record page, create/edit modal, Convert modal, Kanban) captured from the org, region-by-region, with the real section/related-list/control names. Read for exact fidelity.
6. `references/component-catalog.md` — the **reference-equivalent component library** to build in Blade/Alpine/Tailwind (page-header, path, data-table, detail-section, activity-panel, related-list, modals, form fields, badges, kanban, toasts) + design tokens. This is what makes it *look* like a polished enterprise CRM.
7. `references/modules.md` — the Lead module build spec (routes, controllers, saved views, quick actions, validation). Every other object mirrors this pattern.
8. `references/ux-design-system.md` — the CRM's UI rules: layout, table density, status/stage/priority badges, empty/loading/error states, accessibility, responsive, Alpine patterns.
9. `references/testing.md` — test checklist, commands, done-criteria. Run before claiming done.

## Prerequisites (Day 0)

This skill is a **build framework**, not a scaffolder: the Reuse map below assumes your app *already has* an operational lead/pipeline table with owner, priority, and follow-up tracking, a dashboard shell with auth, and `admin`/`sales` roles — the names in the table are illustrative examples of that shape, not a hard requirement to match exactly. If your app doesn't have an equivalent yet, build (or stub) that layer first — Phase A step 1 assumes it exists and will reference undefined classes otherwise. Also verify before writing the first controller: on Laravel 11+, the base `app/Http/Controllers/Controller.php` no longer includes `AuthorizesRequests` by default, which this skill's `$this->authorize()` pattern needs everywhere.

## Reuse map (existing code — do not duplicate)

| Need | Reuse this | Location |
|---|---|---|
| Lead data (rich: owner, priority, score, follow-ups, outcomes) | `CommercialPipelineEntry` | `app/Models/CommercialPipelineEntry.php` |
| Next-action / bucket / urgency logic | `actionForEntry()` + helpers | `app/Http/Controllers/Dashboard/CommercialPipelineTodayController.php` — **extract to `App\Support\CommercialPipelineActionService`**, then both `/dashboard/oggi` and `/crm` consume it |
| Filtered query, columns, summary, CSV | `filteredQuery()`, `columns()`, `export()` | `app/Http/Controllers/Dashboard/CommercialPipelineController.php` |
| CSV output | `App\Support\CsvExporter` | `app/Support/CsvExporter.php` |
| Follow-up outcome vocab + slug badges | `FOLLOW_UP_OUTCOMES`, `*OutcomeSlug` accessors | model |
| Dashboard shell, sidebar, auth | `layouts/dashboard.blade.php`, `auth` + `role:admin` middleware | routes group `prefix('dashboard')->as('dashboard.')` |
| Config vocab | `config('crm.lead_statuses')`, `user_roles`, `dashboard.page_sizes` | `config/crm.php` |

## Non-negotiable guardrails

- **Additive migrations only.** New columns are nullable + have a `down()`. Never rename/drop existing pipeline columns.
- **Don't break existing pipeline pages/tests.** `/dashboard/pipeline-commerciale`, import/export, bulk update stay green.
- **Single source of truth for next-action:** the extracted service. Do not re-implement bucket logic in CRM controllers/views.
- **No real source-org data** in code, seeds, docs, or commits — no real names/emails/phones. Demo seeders use fake data only.
- **Server-side first:** filtering, sorting, pagination in SQL (guard against N+1). Alpine handles only in-page interactions (drawer, column toggle, bulk-select, quick-action menus).
- **Auth + policy on every route and action.** `sales` and `admin` both use the CRM; "Assigned to me" and ownership scoping go through the policy.
- The app's own localized UI and design system, no external assets/CDNs beyond what the project already loads.

## Build order (incremental — implementation is TDD)

**Phase A — Lead layer** (details in `modules.md` + `data-model.md`)
1. Extract `CommercialPipelineActionService` (pure refactor; `/dashboard/oggi` tests stay green).
2. Additive pipeline migration (email, phone, status, owner_id, optional lead_source/website/linkedin) + casts/accessors + backfill.
3. Lead list view `/crm/leads` (table, saved views, search, sort, pagination, empty/loading/error).
4. Filters drawer + column toggle + bulk actions + row actions + quick-action endpoints.
5. Lead detail `/crm/leads/{entry}` (highlights, next-action, cards, timeline, notes, quick links).

**Phase B — Relational sales flow** (details in `objects-catalog.md` + `data-model-full.md`)
6. `crm_accounts` + `crm_contacts` (list + detail + relations), new tables.
7. `crm_opportunities` with the real **stage path** (`Qualify→Present→Propose→Negotiate→Won/Lost`) + "by stage" view.
8. Lead **Convert** action → creates Account + Contact + Opportunity (transactional).
9. `crm_activities` + real timeline on every detail page (replaces the derived timeline).
10. Catalog (products/price books/line items) — only if a real SKU list is needed.

**Every phase:** policy per object + tests (`testing.md`), then `pint`, `composer test`, `npm run build`. Each object mirrors the Lead module pattern.

**REQUIRED SUB-SKILL:** use superpowers:test-driven-development for every controller/service/policy change. **REQUIRED:** use superpowers:verification-before-completion before claiming any phase done.
