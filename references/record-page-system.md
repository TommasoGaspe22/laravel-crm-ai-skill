# Record page system

**Scope:** common structure of record pages (Lead/Account/Contact/Opportunity). Source: **Observed/Tested** (real screenshots + interactions). For per-object details see `lead-record-page.md`, `account-record-page.md`, `contact-record-page.md`, `opportunity-record-page.md`. Visual anatomy in `screen-anatomy.md` §B.

## Structure (Observed) — 3-column layout
1. **Header:** object icon + label + **record name** (large) + **actions** group on the right (+ ▾ overflow). Actions per object:
   - Lead: `Convert · Change Owner · Edit · ▾`
   - Account: `New Contact · New Opportunity · Edit · ▾` (+ org-chart icon)
   - Contact: `New Opportunity · Edit · ▾`
   - Opportunity: `New Event · New Task · Log a Call · Edit · Create New Invoice · ▾`
2. **Path** (Lead & Opportunity only): stage chevrons + per-stage guidance + commit button. See `stage-transition-system.md`.
3. **Left column — Detail sections** (collapsible cards) with field rows + **inline edit ✎**. Real sections per object (Observed): Lead `About/Get in Touch/Segment/History`; Account/Contact `About/Get in Touch/History`; Opportunity `About/Status`.
4. **Middle column — Activity panel** (composer + timeline). See `activity-and-timeline-system.md`.
5. **Right column — Related lists** (card `Title (N)` + rows + "View All" + quick create). Related lists per object (Observed): Account `Contacts/Opportunities/Cases`; Contact `Opportunities/Cases/Files`; Opportunity `Contact Roles/Files/Invoices`; Lead `Files`. See `related-list-system.md`.

## Behaviors (Observed/Tested)
- **Inline edit** (`inline-edit-system.md`): ✎ per field, batch Cancel/Save bar.
- **Navigating to related records:** lookups (e.g. Account name) are clickable links → related record.
- **Edit** (header action) → full edit modal (`form-and-modal-system.md`).
- **Deletion** (Tested): ▾/`Delete` action → **confirmation dialog** ("Delete this account?" / title "Delete Account", `Cancel`/`Delete`) → redirect to the list view. Cascading to related records **to verify**.
- **No-activity / no-relations state:** curated empty states in the cards.
- **Loading:** skeleton while loading; details **to verify**.
- **AI banner (Einstein-style)** and **chat card**: **Do not replicate**.

## Proposed Laravel design (Proposed for Laravel)
- Route `/dashboard/crm/{obj}/{id}` → `show` controller. Components: `x-crm.page-header`, `x-crm.path`, `x-crm.detail-section`, `x-crm.activity-panel`, `x-crm.related-list` (see `component-catalog.md`).
- **Layout config** per object: array of sections→fields + related lists → reusable and configurable (mirrors the reference org's "page layout" without the editor).
- Eager-load relations (owner, account, related counts) to avoid N+1.
- `view` policy on the record; actions (Edit/Delete/Convert/…) gated.
- **Deletion:** `x-crm.confirm-dialog` + `DELETE` with `delete` policy (admin) + soft delete + cascade/relation handling (prevent orphans or cascade with confirmation).

## Priority
- **V1:** header+actions, detail sections with inline edit, activity panel, main related lists, edit/delete with confirmation, path where relevant.
- **V2:** configurable layout per object/profile, rich related lists (pagination, inline actions), advanced skeletons.
- **V3:** page layout editor, custom components.

## Open questions → `open-questions-and-assumptions.md`
Deletion cascade; skeleton/loading; state with many relations; back navigation; responsive record page.
