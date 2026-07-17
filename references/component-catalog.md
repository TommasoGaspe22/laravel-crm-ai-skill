# Component catalog (reference-equivalent, Blade + Alpine + Tailwind)

To make the CRM look *practically identical* to the reference enterprise CRM, rebuild these reusable components once and use them everywhere. They mirror the **reference design system** but are implemented in the project's stack — **no external design-system CSS, no external assets**. Put them in `resources/views/components/crm/` (Blade components) + `public/css/crm.css` (tokens + component styles) + small Alpine behaviors.

## Design tokens (crm.css `:root`)
Match the reference's calm, dense, enterprise feel:
- **Neutrals**: page bg `#f3f3f3`; card bg `#fff`; border `#e5e5e5`/`#c9c9c9`; text `#181818`, secondary `#444`, muted `#706e6b`.
- **Brand**: use your own brand color as the reference "brand blue" equivalent for primary buttons, active tab underline, links, current path step. (Reference blue `#0176d3` → swap your own brand.)
- **Semantic**: success/green `#2e844a`, warning/amber `#fe9339`, error/red `#ea001e`, info.
- **Radius** 4px; **shadow** subtle (`0 2px 2px rgba(0,0,0,.05)`); **font** system UI stack; base 13px, dense line-height; labels 12px muted uppercase-ish.
- Reuse existing model badge classes (`score-*`, `priority-*`) as tokens.

## Components (each: structure → behavior → where used)

### 1. `x-crm.page-header`
Object label (small) + record/list name (large) + right-aligned **action button group** + ▾ overflow. Props: `label`, `title`, `:actions`. Used on every list + record page. (Anatomy A.4 / B.1.)

### 2. `x-crm.path` (stage chevrons)
Horizontal chevron steps; completed = tinted fill, current = dark/brand, future = grey; right primary button "Mark as Complete". Props: `:steps`, `current`, `:onSelect`. Alpine handles click→confirm→PATCH. Used on Lead (status) + Opportunity (stage). (Anatomy B.2 / screen E for board variant.)

### 3. `x-crm.highlights` (record header details)
Under the header: key field chips (e.g. Company · Role · Phone · Email · Owner · Status/Priority badges). Props: `:fields`.

### 4. `x-crm.detail-section`
Collapsible card with a section title (`About`, `Get in Touch`, …) containing **field rows**: `label` + `value` + inline-edit `✎`. Slot-based. Alpine: toggle collapse; inline-edit swaps value→input, batches into a save bar. Used in left column of record pages. (Anatomy B.3.)

### 5. `x-crm.data-table`
Dense sortable table: header select-all + sortable `<th>` (`aria-sort`) + column-action ▾; rows with checkbox, cells (optional `✎`), trailing row-action ▾. Props: `:columns`, `:rows`, `:sort`, `selectable`. Server-side sort via links; Alpine for selection + row menus. The workhorse — Lead/Account/Contact/Opportunity list views. (Anatomy A.7.)

### 6. `x-crm.list-toolbar`
Search box + controls gear menu (New/Clone/Rename/Sharing/Select Fields/Delete/Reset Widths/Table·Kanban·Split) + density + refresh + sort + inline-edit toggle + charts + filters. Props: `:view`, `:displayMode`. (Anatomy A.6.)

### 7. `x-crm.view-selector`
List-view name + ▾ dropdown of saved views + pin (default). Alpine dropdown; links to `?view=`. (Anatomy A.4.)

### 8. `x-crm.filter-panel`
Right drawer: field conditions (add/remove), "Apply/Reset", active-filter chips. GET form (shareable URLs). (Anatomy A.6 / modules.md §5.)

### 9. `x-crm.activity-panel`
Composer buttons row (Email · Task · Call · Event, each ▾) + filter line + collapsible groups ("Upcoming & Overdue", by month) + empty state + "Show All". Backed by `crm_activities`. (Anatomy B.4.)

### 10. `x-crm.related-list`
Card: `Title (N)` + compact rows + "View All" + quick "New". Props: `title`, `:rows`, `createRoute`. Right column of record pages (Contacts, Opportunities, Cases, Files). (Anatomy B.5.)

### 11. `x-crm.record-modal`
Centered modal with title, `* = Required Information`, **field sections** (grey headers), footer `Cancel · Save & New · Save`. Used for Create/Edit (Anatomy C) and adapted for the **Convert** modal (Anatomy D: three Create/Choose sections). Alpine: focus-trap, Esc close, required validation.

### 12. Form fields
`x-crm.field.text`, `.textarea`, `.date` (calendar popover), `.number`, `.picklist` (combobox ▾ w/ `role=listbox`/`option`), `.lookup` (search-as-you-type against an endpoint, shows "N matches"), `.checkbox`, `.owner` (user lookup). Each with label, required marker, error slot, focus ring.

### 13. `x-crm.badge`
Pill for status/stage/priority/urgency. Color per semantic token; **always includes text** (never color-only). Uses `score-*`/`priority-*` classes.

### 14. `x-crm.kanban`
Columns per stage with count + amount sum; draggable cards (SortableJS deferred). Drop → PATCH stage. (Anatomy E.)

### 15. `x-crm.toast`
Top auto-dismiss success/error, driven by Laravel `session('status')`/errors. Consistent feedback after every action.

## Behavior primitives (Alpine)
- `dropdown` (menus, view selector, row/column actions) — click-outside + Esc + `aria-expanded`.
- `modal` — focus-trap + Esc + backdrop.
- `inlineEdit` — per-field edit state + batched save bar.
- `bulkSelect` — reactive selected-ids + bulk bar.
- `drawer` — filter panel open/close.
Keep all JS tiny and framework-free (no React/LWC). Server-render data; Alpine only decorates.

## Fidelity checklist (per screen)
- [ ] Header + actions match anatomy.
- [ ] Path present where applicable, with real stages.
- [ ] Detail sections named + inline-edit working.
- [ ] Activity panel + related lists present.
- [ ] Table dense, sortable, selectable, row menus, inline `✎`.
- [ ] Toolbar controls incl. display modes (Table/Kanban/Split).
- [ ] Create/Convert modals sectioned with correct footer.
- [ ] Badges text+color; toasts on actions; a11y (aria, focus, Esc).
