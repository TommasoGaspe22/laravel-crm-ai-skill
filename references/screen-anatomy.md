# Screen-by-screen anatomy (captured, pixel-level)

Every CRM screen archetype, described so it can be rebuilt to match the reference org. Captured read-only from the org (Italian). Build these as your own branded Blade components (see `component-catalog.md`). Measurements are relative — match proportions/spacing, not exact px.

Legend: `▾` overflow/menu · `✎` inline-edit pencil · `*` required.

---

## A. List view (e.g. Lead / Account / Opportunity)

Region-by-region, top→bottom:

1. **Trial/promo bar** (reference-platform-only) — DO NOT replicate.
2. **Global header**: logo left · centered **global search** (`Search…`) · right icons (help, setup, notifications w/ badge, avatar). Target app: reuse existing dashboard header; add the centered search.
3. **Object tab bar**: horizontal tabs per object (Lead · Contacts · Accounts · Opportunities · Products · Price Books · Calendar · Analytics · Invoices), each with a ▾ that shows recent records + "New". Active tab underlined. Target app: a top object-nav strip inside the CRM area.
4. **Object header row**:
   - Left: small object label + **list-view name with ▾ selector** + **pin** (set as default).
   - Right: **object action buttons** (e.g. `New · Import · Add to Campaign · Send Email · Change Owner`) + ▾ overflow.
5. **Meta line**: `"{N} items • Sorted by {col} • Filtered by {filter} • Updated {t} ago"`.
6. **List toolbar** (right-aligned icon row):
   - `Search this list…` search box.
   - **List-view controls gear** → menu (real): `New · Clone · Rename · Sharing Settings · Select Fields to Display · Delete · Reset Column Widths · —— · Table · Kanban · Split View`.
   - **Display density**, **Refresh**, **Column sort**, **Edit List** (toggle inline-edit), **Charts**, **Filters** (opens right panel).
7. **Data table** (`role="grid"`, dense, sticky header):
   - Header select-all checkbox + columns; each header: label + sort caret (`aria-sort`) + column-action ▾ (Sort ascending/descending, Wrap/clip text, width).
   - Row: checkbox · cells (editable cells show `✎` on hover) · trailing **row action ▾** (Edit, Delete, Change Owner…). First cell (name) is a link to the record.
8. **Display modes** (from the gear): **Table** (default), **Kanban** (board grouped by a picklist — for Opportunity grouped by *Stage*), **Split View** (list left, record preview right).
9. **Utility bar** bottom (To Do List) — optional, skip in V1.

**States:** empty (curated per view), loading skeleton rows, error/unavailable panel, selected-rows bulk bar.

---

## B. Record (detail) page — Lead / Account / Contact / Opportunity

Consistent 3-column reference layout:

1. **Record header**:
   - Left: object label + **record name** (large) + object icon.
   - Right: **primary action buttons** + ▾ overflow. Real per object:
     - Lead: `Convert · Change Owner · Edit · ▾`
     - Account: `(org-chart icon) · New Contact · New Opportunity · Edit · ▾`
     - Contact: `New Opportunity · Edit · ▾`
     - Opportunity: `Edit · ▾` (+ stage path actions)
2. **Path component** (only Lead & Opportunity): horizontal chevrons of the stages, completed ones tinted, current one dark, future ones grey; right side primary button **"Mark Status as Complete"** (or "Select Converted Status" on the last step). See component-catalog.
   - Lead stages (real): `New → Contacted → Nurturing → Unqualified → Converted`.
   - Opportunity stages (real): `Qualify → Present → Propose → Negotiate → Won → Lost`.
3. **Left column — Detail section cards** (collapsible, real section names from this org's layout): `About`, `Get in Touch`, `Segment` (Lead), `History`. Each holds field rows with label + value + `✎` inline-edit. Fields per object are in `objects-catalog.md`.
4. **Middle column — Activity panel**:
   - **Composer buttons** row: `Email · New Task/Event · Log a Call · New Event` (each icon + ▾).
   - Toggle "Show only activities with insights".
   - Filter line: `Within 2 months • All Activities • All Types` + gear.
   - `Refresh · Expand All`.
   - Collapsible groups: **"Upcoming & Overdue"**, then past by month. Empty state: "No activities to show…".
   - `Show All Activities` button.
5. **Right column — Related lists / cards**: cards like Contacts, Opportunities, Cases, Files (each `Title (N)` + rows + "View All"). (Org also shows a Slack card — skip.)

**Target-app note:** in V1 the middle Activity panel = the new `crm_activities` timeline + quick-log composer; related lists (right) render the object's relations from `data-model-full.md`.

---

## C. Create / Edit modal (e.g. Create opportunity)

- Centered **modal** with title `Create {object}` / `Edit {object}`, X to close.
- Top-right note `* = Required Information`.
- **Field sections** (grey section headers) grouping fields. Opportunity (real): **About** (`*Opportunity name`, `*Account name` lookup, `*Close date` date, `Amount`, `Description`, `Owner`) + **Status** (`*Stage` picklist, `Probability (%)`, `*Forecast category`, `Next step`).
- Inputs: text, textarea, date-picker (calendar icon), number, **lookup** (search field w/ magnifier), **picklist** (combobox ▾).
- **Footer**: `Cancel · Save & New · Save` (primary).
- Inline validation on required fields; error text under the field.

---

## D. Convert modal (Lead → Account + Contact + Opportunity)  *(real)*

Title **"Convert Lead"**, X to close. Three stacked sections, each with a **Create / Choose** radio pair:

1. **Account**: `Create` (`*Account name` prefilled from Company) **or** `Choose` (search "Search for matching accounts", shows N matches).
2. **Contact**: `Create` (name prefilled) **or** `Choose` (shows "N contact match found").
3. **Opportunity**: `Create` (opp name prefilled `"{company}-"`) + checkbox **"Don't create an opportunity upon conversion"** **or** `Choose`.
- Bottom: `*Record owner` (owner lookup) · `*Converted status` (picklist, default "Qualified").
- Footer: `Cancel · Convert` (primary).
- Helper text: *"We'll turn this lead into three new records: Account, Contact, Opportunity."*

**Target app:** this is the `convertLead()` flow in `data-model-full.md` — the UI is a modal with these three find-or-create sections.

---

## E. Kanban (Opportunity by stage)

Board with one **column per stage** (`Qualify … Won/Lost`), cards = opportunities (name, account, amount, close date), **drag & drop** a card to change stage (SortableJS, deferred dep). Column header shows count + sum(amount). Reachable from the list-view gear → "Kanban".

---

## Cross-cutting behaviors to replicate

- **Inline edit**: hover a field/cell → `✎`; click → input; multiple edits batch into a "Save/Cancel" footer bar.
- **Sort**: click header toggles asc/desc, single active sort, `aria-sort` reflected.
- **Bulk**: header checkbox selects visible; bulk bar with mass actions; mass inline-edit via "Edit List".
- **Saved views**: selector + pin default; controls menu (rename/save/save-as/share/filters/delete).
- **Toasts**: success/error toast after actions (top, auto-dismiss).
- **Keyboard/a11y**: focusable controls, `aria-label` on icon buttons, focus-trap in modals, Esc closes.
