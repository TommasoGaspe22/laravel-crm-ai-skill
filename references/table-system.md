# Table system (data grid)

**Scope:** the dense table used by list views. Source: **Observed/Tested** (populated lists).

## Observed
- `role="grid"`, sticky header, compact density, zebra/hover.
- **Header:** select-all checkbox + columns; every header with **sort** (asc/desc, one active, `aria-sort`) + **column actions** menu ("Show column actions": sort, wrap/clip text, width).
- **Row:** selection checkbox + cells + **✎ inline edit** on editable cells + **row action ▾** (Edit/Delete/Change Owner/…). First cell (name) = link to the record.
- **Cells:** links, badges (status/stage), avatar/owner (owner alias), formatted dates, truncated text with tooltip.
- **"Edit List" mode** → mass inline cell editing.
- **Sorting** (indirectly tested): meta line "Sorted by {col}".

## Proposed Laravel design (Proposed for Laravel)
- **Component** `x-crm.data-table`: props `:columns` (label, key, sortable, align, formatter, editable), `:rows`, `:sort`, `selectable`, `:rowActions`. Server-side sort via `?sort=&dir=` links; selection + menus via Alpine.
- **Density/columns**: `column-management-system.md`. **Sorting**: column whitelist (anti-injection); DB indexes on sortable columns; one active sort (V1), multi (V2).
- **Formatter** per type (date/currency/badge/link/owner). Truncation + `title`.
- **Performance:** server-side pagination (`pagination-and-loading-system.md`); eager-load displayed relations (owner/account); no N+1.
- **Accessibility:** `<th scope=col>`, sortable headers as `<button aria-sort>`, labeled checkboxes.

## Priority
- **V1:** dense table, single sort, badges/formatters, links, row action, select+bulk, responsive (horizontal scroll). **V2:** inline cell edit, multi-sort, column resize, sticky first column. **V3:** virtualization/data-grid lib (Tabulator) if needed.

## Open questions
Multi-sort; persistent resize; virtualization with many records (→ `pagination-and-loading-system.md`).
