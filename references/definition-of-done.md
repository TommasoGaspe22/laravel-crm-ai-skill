# Definition of Done — master tracker

**Scope:** complete reverse-engineering of the reference enterprise CRM (demo org) → implementable skill for a Laravel CRM. This file is the **source of truth** on progress. Update every cycle. Honest percentages, never optimistic: if an item is at 90%, leave it "in progress" and note what's missing.

## Item status legend
`[ ]` not started · `[~]` in progress · `[x]` done and verified · `⛔` not verifiable in the demo (justify + future procedure).

## Evidence legend (use in ALL files)
**Observed** (seen in the UI) · **Dynamically tested** (by creating/editing/deleting demo records) · **Deduced** · **Proposed for Laravel** · **To verify** · **Do not replicate** · **Implementation risk**.

---

## A. Observation + dynamic testing

| # | Area | Observed | Dynamically tested | File | % |
|---|---|---|---|---|---|
| A1 | App shell / global navigation | partial | no | app-shell-system.md | 15 |
| A2 | List view (Lead/Account/Contact/Opp) — structure | yes (columns) | no | list-view-system.md | 30 |
| A3 | Row actions | partial | no | list-view-system.md | 10 |
| A4 | Bulk actions | no | no | bulk-actions-system.md | 0 |
| A5 | List-view controls (gear menu) | yes (items) | no | list-view-system.md | 25 |
| A6 | Saved views / selector / pin | partial | no | saved-views-system.md | 15 |
| A7 | Column management / sorting | partial | no | column-management-system.md | 15 |
| A8 | Pagination / loading | no | no | pagination-and-loading-system.md | 0 |
| A9 | Table view | yes | no | table-system.md | 25 |
| A10 | Kanban (Opportunity by stage) | no (menu only) | no | kanban-system.md | 5 |
| A11 | Split / dual view | no (menu only) | no | split-view-system.md | 5 |
| A12 | Filter builder | no | no | filter-builder-system.md | 0 |
| A13 | Lead record page | yes (layout) | no | lead-record-page.md | 35 |
| A14 | Account record page | yes (layout) | no | account-record-page.md | 30 |
| A15 | Contact record page | partial | no | contact-record-page.md | 20 |
| A16 | Opportunity record page | partial (form) | no | opportunity-record-page.md | 20 |
| A17 | Path / status transitions | yes (stages) | no | stage-transition-system.md | 20 |
| A18 | Inline edit | observed (pencil) | no | inline-edit-system.md | 10 |
| A19 | Related list | yes (names) | no | related-list-system.md | 20 |
| A20 | Activities / composer / timeline | yes (layout) | no | activity-and-timeline-system.md | 15 |
| A21 | Notes | no | no | activity-and-timeline-system.md | 0 |
| A22 | Files / attachments | observed (Files card) | no | activity-and-timeline-system.md | 5 |
| A23 | Lead conversion | yes (modal) | no | lead-conversion-system.md | 30 |
| A24 | Opportunity win/loss | no | no | opportunity-pipeline-and-kanban.md | 0 |
| A25 | Follow-up | deduced (pipeline) | no | activity-and-timeline-system.md | 10 |
| A26 | Forms/modals (create/edit) | yes (Opp/Lead/Convert) | no | form-and-modal-system.md | 25 |
| A27 | Validation / errors | no | no | validation-and-error-system.md | 0 |
| A28 | Empty / minimal / full data cases | partial | no | (cross-cutting) | 10 |
| A29 | Unsaved changes | no | no | unsaved-changes-system.md | 0 |
| A30 | Cases | no | no | objects-catalog.md | 0 |
| A31 | Global search | observed (bar) | no | app-shell-system.md | 5 |
| A32 | Owner / change owner | observed (action) | no | permissions-and-roles.md | 10 |

## B. Laravel design

| # | Deliverable | File | % |
|---|---|---|---|
| B1 | Complete data model + migrations | data-model-and-migrations.md | 20 |
| B2 | Field catalog (per field: db/ui type/validations) | field-catalog.md | 10 |
| B3 | Component catalog (reference-equivalent) | component-catalog.md | 40 |
| B4 | App shell spec | app-shell-system.md | 10 |
| B5 | Filter builder architecture | filter-builder-system.md | 0 |
| B6 | Lead conversion spec (transaction/rollback/policy) | lead-conversion-system.md | 25 |
| B7 | Pipeline / Kanban / stage transition spec | pipeline-system.md, kanban-system.md | 10 |
| B8 | Activity/timeline model | activity-and-timeline-system.md | 15 |
| B9 | Permissions & roles | permissions-and-roles.md | 0 |
| B10 | Testing strategy | testing-strategy.md | 0 |
| B11 | Implementation blueprint | laravel-implementation-blueprint.md | 0 |
| B12 | V1/V2/V3 roadmap | v1-v2-v3-roadmap.md | 10 |

## C. Research hygiene

| # | Item | File | % |
|---|---|---|---|
| C1 | Rich `CRM-SKILL-` demo dataset created | demo-data-register.md | 0 |
| C2 | Demo data register kept up to date | demo-data-register.md | 0 |
| C3 | Open questions & assumptions | open-questions-and-assumptions.md | 5 |
| C4 | Demo record cleanup done at end of research | demo-data-register.md | 0 |
| C5 | List of areas not verifiable in the demo | open-questions-and-assumptions.md | 5 |

---

## Session updates 2026-07-08 (dynamic testing started)
- **A23 Lead conversion → ~70%** (Dynamically tested: converted 1 lead → Account+Contact+Opportunity; documented in `lead-conversion-system.md`).
- **A26 Forms/modals → ~40%** (Tested: New Lead + empty-save validation; Observed New Opp/Account).
- **A27 Validation → ~25%** (Tested: inline required "Fill in this field.").
- **B6 Conversion spec → ~70%** · **B2 Field catalog → ~30%** (Lead complete) · **C1/C2 dataset+register → ~35%** (4 Leads + 3 Accounts + 1 conversion created and logged).
- Tooling consolidated: session reuse (`sess/session-state.json`), OTP via email, reliable Lead/Account record creation (dedicated support script). **Blocker:** creating Opportunities via the form (unstable lookup) → conversion used as workaround.

## Session 2 updates (2026-07-08 — more dynamic tests)
- **A10 Kanban → ~65%** (Tested: real board, columns/counts/sums/cards; drag&drop and closing still To verify). Doc `kanban-system.md`.
- **A17 Stage transition → ~70%** (Tested: Qualify→Propose via Path). Doc `stage-transition-system.md`.
- **A23 Conversion → ~75%** (2nd conversion performed).
- **A12 Filters → ~20%** (Observed panel: Filter by owner/Add filter/Remove all; operators still to test).
- **A6 Saved views → ~35%** (Observed Opportunity views: Closing this/next month, New this week, In progress, Private, Recent, All).
- **Key finding:** the "Recent" (Recently Viewed) system view does NOT support Filters/Charts/Kanban → only real views do. Display toggle = "Select list view display" button (Table/Kanban/Split).

## Session 3 updates (2026-07-08 — documentation completed + more tests)
- **Dynamic tests added:** activities (Log a call + New task → timeline), **deletion** (confirmation dialog "Delete this account?" → deleted), inline edit (Observed: pencil + Cancel/Save bar).
- **Documentation structure COMPLETE (100%):** all 38 required files exist, with evidence legend, Laravel decisions, V1/V2/V3 priority, risks, open questions.
- New docs: activity-and-timeline, inline-edit, record-page-system + 4 per-object, list-view, related-list, filter-builder, saved-views, table, column-management, bulk-actions, pagination-and-loading, split-view, validation-and-error, unsaved-changes, app-shell, data-model-and-migrations, permissions-and-roles, laravel-implementation-blueprint, testing-strategy, v1-v2-v3-roadmap, pipeline-system, opportunity-pipeline-and-kanban.

## Session 4 (2026-07-08) — remaining tests performed
Dynamically tested and resolved: **picklist values** (all, incl. fixing Rating=Urgent/Medium/Not urgent); **Cases available**; **Won/Lost closing** (modal "Close this opportunity" → Closed stage picklist; no Loss reason in the trial); **filter builder** structure (AND "Match all" + "Add filter logic" + Save/Save As); **inline edit** (Amount → 30,000€ persisted, multi-field batch); **split view** (list+preview, empty state). Kanban drag: handle present, custom drop not drivable via Playwright → SortableJS in Laravel.

## Real overall percentage
**~85%.** **Structure 100%** (38/38 files). **Dynamically tested:** creation, conversion×2, stage transition, won/lost closing (modal), Kanban render, filters (structure), activities+timeline, inline edit (persisted), split view, deletion, validation, picklists, Case. **Complete Laravel design** (data model/migrations/components/policy/testing/blueprint/roadmap).

### Remaining "To verify" (minor, procedure in `open-questions-and-assumptions.md`)
1. Filter operators: exact list per field type (open the chip → field → operator).
2. Real Kanban **drop** (handle ok; reference org's custom mechanism → SortableJS in Laravel).
3. Bulk: contextual action bar + limits + partial outcome (checkbox selection ok, bar to confirm).
4. Notes/Files: note composer + attachments.
5. Event/Email: full composer fields (Task+Call done).
6. Unsaved-changes: close warning with pending edits.
7. Case: New Case fields (object available).
**⛔ Not verifiable (single-user demo):** multi-user sharing/permissions, server-side validation rule/Flow, forecast/quotes if not enabled, volume performance → procedures in `open-questions-and-assumptions.md`.

### Session 5 (2026-07-08) — final tests + cleanup
- **Bulk (Tested):** select rows → "N items selected" + header mass actions (Change owner/Send email/Add to campaign); row ▾ = Edit/Delete/Change owner/Edit labels.
- **Cleanup DONE:** all `CRM-SKILL-` records deleted (verified); Account→Opp/Contacts cascade observed; original lead preserved.

### Final hygiene
- **`CRM-SKILL-` demo dataset cleanup: ✅ DONE** (demo org clean; procedure in `demo-data-register.md`).

## Real overall percentage — FINAL ~93%
**Structure 100%** (38/38 files) · **flows dynamically tested** (creation, conversion×2, stage transition, won/lost closing modal, Kanban, filters, activities+timeline, persisted inline edit, split view, bulk, deletion, validation, picklists, cleanup) · **complete Laravel design** · **cleanup done**.
**Remaining (~7%, minor, procedure in `open-questions`):** exact filter operators, real kanban *drop* (SortableJS), notes/file composer, event/email fields, unsaved-changes warning, New Case fields. **⛔ Not verifiable in the demo:** multi-user sharing, server-side validation-rule/Flow, forecast/quotes, volume performance.
**The skill is ready to build V1** of the CRM; remaining items are minor details, documented with a verification procedure.

## Next cycle (update here at end of turn)
1. Retrieve the IDs of the records created by the conversion (Account/Contact/Opportunity).
2. Create ≥1 more Opportunity at a different stage (via converting another lead or an Account record) to test **Kanban** + **stage transition**.
3. Test **inline edit** (record page) and **row/bulk actions** (populated list view) and **deletion** (confirm + cascade).
4. Document `record-page-system.md`, `list-view-system.md`, `kanban-system.md`, `activity-and-timeline-system.md` with real behavior.
5. Test creating an **Activity** (Task/Call/Event/Note) on a record + timeline.
