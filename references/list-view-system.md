# List view system

**Scope:** list views for all objects. Source: **Observed/Tested** (populated Lead/Opportunity lists, controls, views). Anatomy: `screen-anatomy.md` §A. Sub-systems: `saved-views-system.md`, `table-system.md`, `column-management-system.md`, `bulk-actions-system.md`, `pagination-and-loading-system.md`, `split-view-system.md`, `filter-builder-system.md`.

## Index
1. Structure · 2. Controls · 3. Display modes · 4. Findings · 5. Laravel · 6. Priority · 7. Open questions

## 1. Structure (Observed)
- Header: object label + **view name with ▾ + pin** (default) + **object actions** (`New · Import · …`).
- **Meta line:** "N items • Sorted by {col} • Filtered by {…} • Updated {t} ago". (Tested: count updates with data; e.g. Opportunity "2 items".)
- **Toolbar:** list search · controls gear · display selector · Refresh · Column sort · Edit list (inline) · Charts · Filters.
- **Dense table:** select-all + row checkbox + cells (✎ editable) + **row action ▾**.

## 2. Controls (Observed)
- **"List view controls" gear:** `New · Clone · Rename · Sharing Settings · Select Fields to Display · Delete · Reset Column Widths`. (Tested: real menu.) → `saved-views-system.md`, `column-management-system.md`.
- **View selector** (▾ on the name): list of views (recent/personal/standard). Tested for Opportunity: `Closing This/Next Month · New This Week · Opportunities In Progress · My Opportunities · Recently Viewed Opportunities · Recent · All Opportunities`.
- **Sorting:** click a column header (asc/desc, one at a time). `Column sort` for multi. → `table-system.md`.
- **List search:** server-side `q` across the view's fields.

## 3. Display modes (Observed/Tested)
**"Select list view display"** button (grid icon ▾) → **Table · Kanban · Split View**. See `table-system.md`, `kanban-system.md`, `split-view-system.md`.

## 4. Key findings (Tested)
- **The "Recent" (Recently Viewed) system view does NOT support Filters, Charts, Kanban** ("not available for this list view"). Only real views (standard/personal) enable them.
- Object actions and columns change per object (see `source-crm-reference.md` + capture).
- Row action ▾ and select-all present on populated lists.

## 5. Proposed Laravel design (Proposed for Laravel)
- `index` controller per object: server-side query with **view** (`?view=`), **search** (`q`), **sort** (`sort`/`dir` whitelisted), **filters** (`filter-builder-system.md`), **pagination** (`pagination-and-loading-system.md`). Eager-load for N+1.
- Components: `x-crm.page-header`, `x-crm.view-selector`, `x-crm.list-toolbar`, `x-crm.data-table`, `x-crm.filter-panel`, `x-crm.kanban` (see `component-catalog.md`).
- **Views**: `App\Support\{Obj}ListViews` + `saved_views` table (V2). Distinguish "system views" (recent/mine) without advanced filters/kanban.
- Preferences (columns/density/view) persisted (session/localStorage V1; table V2).

## 6. Priority
- **V1:** dense table (sort, search, basic filters, preset views, row actions, essential select+bulk, pagination, empty/loading/error states). "By stage" grouped view (Opportunity).
- **V2:** Kanban drag&drop, split view, configurable+resizable columns, user saved views, advanced filter builder, charts.
- **V3:** shared views/permissions, advanced charts, rich import wizard.
- **Do not replicate (V1):** embedded charts, split view, granular view sharing.

## 7. Open questions → `open-questions-and-assumptions.md`
Pagination with many records (Q13); preference persistence; import wizard; charts; per-object actions.
