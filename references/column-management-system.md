# Column management system

**Scope:** column management for list views. Source: **Observed** (controls menu).

## Observed
- Controls gear → **"Select Fields to Display"** (choose visible/hidden columns + order) and **"Reset Column Widths"**.
- Column header → actions menu: sort, **wrap / clip text**, resize (drag the edge).
- **To verify:** "Select fields" dialog (2 lists: available/selected + reordering); per-user/per-view persistence; saved widths.

## Proposed Laravel design (Proposed for Laravel)
- **Column catalog** per object (single definition: `key, label, sortable, default_visible, align, formatter`).
- **UI** `x-crm.column-manager`: 2-pane dialog (available/visible) + drag reorder; wrap/clip toggle; reset.
- **Persistence:** V1 `localStorage` (per user/browser); V2 in `saved_views.columns_json` (per view, shareable).
- **Rules:** always-visible columns (e.g. record name + row action) can't be hidden; whitelist for query/sort.
- **Implementation risk:** validate chosen columns (whitelist) before using them in `select`/`orderBy`.

## Priority
- **V1:** visible-column toggle + reorder (localStorage), protected always-visible columns. **V2:** persistent widths/resize, wrap/clip, columns per saved view. **V3:** computed columns.

## Open questions
Select-fields dialog (structure); width persistence; default wrap/clip.
