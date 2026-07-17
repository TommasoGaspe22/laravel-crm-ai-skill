# Kanban system (Opportunity pipeline board)

**Scope:** the Kanban view of Opportunities grouped by Stage. Source: **Dynamically tested** (2026-07-08, real board with 2 demo opportunities at different stages) + **Observed**. Laravel blueprint (SortableJS, V2).

## Index
1. How it's activated · 2. Anatomy (tested) · 3. Behaviors · 4. Observed limitations · 5. Laravel · 6. Priority · 7. Open questions

## 1. Activation (Observed/Tested)
- List-view toolbar → **"Select list view display"** button (grid icon ▾, right of the "List view controls" gear) → options **Table / Kanban / Split View**.
- **⚠️ Constraint (Observed):** available only on real list views (e.g. **"All Opportunities"**), **NOT** on the **"Recent" (Recently Viewed)** system view, where Kanban/Filters/Charts are disabled ("Filters/Charts aren't available for this list view"). → In Laravel: "recent/system" views don't offer kanban/filters; only saved/user views do.

## 2. Board anatomy (Dynamically tested)
- **Columns = grouping picklist values** (default **Stage**): `Qualify · Meet & Present · Propose · Negotiate · Closed Won` (closed stages are the final columns). Column header = **blue chevron** with **stage name + (count)**.
- **Sum per column:** below the header, a `€` total (e.g. `€0` because Amount is empty). → aggregates `SUM(amount)` per stage.
- **Opportunity card:** **drag** handle (⋮⋮) on the left, **name/account** (link), **close date**, per-card **▾ actions** menu. (With Amount set, the value would also show.)
- **Meta line** identical to the table: "N items • Sorted by … • Updated …".
- **Filters** and search shared with the table view (Filters panel on the right).

## 3. Behaviors (Observed / Tested)
- **Drag & drop a card between columns** → changes Stage. Observed: **drag handle (⋮⋮) present** on every card. **Tested:** automated drag via Playwright (HTML5 dragTo) **doesn't** trigger the reference org's custom drop (the card doesn't move) → the mechanism exists but needs manual verification; in Laravel we'll use **SortableJS** (V2 dep) with `PATCH stage` on drop. Moving to Closed → opens the **"Close this opportunity"** modal (closed stage required, see `stage-transition-system.md`).
- **Alternative grouping:** the reference org allows changing the grouping field (which fields, **to verify**).
- **Columns with 0 records:** still shown (e.g. Meet & Present (0), Negotiate (0)).
- **Closing:** moving to Closed Won/Lost → **to verify** (likely a reason/closing-status dialog).
- **Performance with many cards:** to verify (can't realistically populate in the demo → target in `testing-strategy.md`).

## 4. Limitations / Do not replicate
- **Do not replicate (V1):** advanced multi-grouping, inline forecast in the kanban, kanban on objects without a pipeline.
- The reference org's board is heavy client-side; in the target app it's **V2** (once `stage` is persisted and stable).

## 5. Laravel blueprint (Proposed for Laravel)
- **Route/view:** `/dashboard/crm/opportunities?view=…&display=kanban`.
- **Component** `x-crm.kanban` + `x-crm.kanban-card` (see `component-catalog.md`). Columns = `config('crm.crm_opportunity_stages')`; cards = opportunities.
- **Data:** the view's opportunity query, grouped by `stage`; per column: `count()` + `sum(amount)`. Eager-load account to avoid N+1.
- **Drag & drop:** **SortableJS** (V2 dependency). On drop → `PATCH /opportunities/{id}/stage {stage}` (`ChangeStage` Action, see `stage-transition-system.md`): validates the transition, updates `stage`/`probability`/`is_closed`, writes a "stage change" `crm_activities` entry, returns a toast. Optimistic UI with rollback on error.
- **Closing (Won/Lost):** dropping on the closed column → "closing reason" modal before committing.
- **Implementation risk:** concurrency (two users move the same card) → use `updated_at` optimistic lock; validate the target stage is allowed; permissions (`update` policy).
- **Accessibility:** drag&drop needs a keyboard alternative (a "Move to stage…" item in the card's ▾ menu).

## 6. Priority
- **V1:** **read-only grouped "by stage"** view (list grouped by stage with counts/sums) + stage change via card menu/record Path (no drag).
- **V2:** Kanban with **drag & drop** (SortableJS), per-column sums, closing modal.
- **V3:** alternative groupings, WIP limits, inline forecast.

## 7. Open questions → `open-questions-and-assumptions.md`
Q2 (real drag&drop + confirmations), Won/Lost closing dialog, alternative groupings, performance.
