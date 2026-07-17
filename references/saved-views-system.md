# Saved views system

**Scope:** saved list views. Source: **Observed** (view selector + controls menu).

## Observed
- **Selector** (▾ on the view name) with: **standard** views (e.g. Opportunity: `Closing This/Next Month`, `New This Week`, `Opportunities In Progress`, `All Opportunities`), **personal** (`My Opportunities`), **system** (`Recent`/`Recently Viewed`, `Recently Viewed Opportunities`). View search ("Search views…"). **Pin** = set as default.
- **Management** (controls gear): `New · Clone · Rename · Sharing Settings · Delete` (+ Select Fields, Reset Widths).
- **Constraint (Tested):** **system** views (Recent) don't support Filters/Kanban/Charts; only standard/personal do.
- **To verify:** creating a view (defining filters + columns + sharing), sharing (who can see it), shared views.

## Proposed Laravel design (Proposed for Laravel)
- **V1:** views predefined in code (`App\Support\{Obj}ListViews`) selectable via `?view=` (mirrors standard views), + default pin (user preference).
- **V2:** **`saved_views`** table (`id, object, name, owner_id, is_shared, filters_json, columns_json, sort_json, is_default`) → user-created/cloned/renamed/deleted views; selector + management UI (`x-crm.view-selector`). Filter serialization = `filter-builder-system.md`.
- **Policy:** personal views (owner) vs shared (visible to staff/team); only owner/admin can edit/delete.
- **Implementation risk:** validate `filters_json`/`columns_json` (field whitelist); never trust values directly in queries (anti-injection); per-user default.

## Priority
- **V1:** preset views + default pin. **V2:** user saved views (CRUD + basic sharing). **V3:** granular/team sharing, dynamic views.
- **Do not replicate (V1):** complex role/queue-based sharing.

## Open questions → `open-questions-and-assumptions.md`
Q7 create/clone/rename/share view flow; standard/personal/system distinction; sharing.
