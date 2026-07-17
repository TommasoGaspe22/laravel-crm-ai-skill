# Split view

**Scope:** two-pane view (list on the left, record on the right). Source: **Observed** ("Split View" option in the display selector); behavior **to verify** dynamically.

## Observed / Dynamically tested (2026-07-08)
- **"Split View"** option in the display toggle (next to Table/Kanban).
- **Tested:** layout confirmed = **compact list on the left** (Stage column, name/account/date/amount card, search, checkboxes) + **record panel on the right** with empty state **"No record selected — To get started, open a record from the list."**; **collapse arrow (◀)** between the panels.
- **To verify:** preview content on clicking a record, panel width/persistence, actions in the preview, responsive behavior.

## Proposed Laravel design (Proposed for Laravel)
- **V2/V3** (not V1). Route `?display=split`. Layout: list column (reuse a compact `x-crm.data-table`) + record panel (reuse `x-crm.page-header` + key sections), loaded via Alpine/partial `fetch` on click (avoids a full reload).
- **Implementation risk:** doubles the per-page query load; load the preview lazily; responsive → collapse to list-only on mobile.

## Priority
- **V2/V3.** In V1 the flow is: list → click → full record page (sufficient). Split view only if the team does mass triage.
- **Do not replicate (V1).**

## Open questions → `open-questions-and-assumptions.md`
Q10: split-view layout/actions/persistence.
