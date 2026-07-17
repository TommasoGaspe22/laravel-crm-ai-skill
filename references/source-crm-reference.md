# Reference org "All Open Leads" — anatomy & mapping to your app

Captured **read-only** from the live reference org (Italian locale). No records were modified; no real data is reproduced here. Use this as the UX target; rebuild with your own branding — never copy the source org's logos, asset URLs, or proprietary CSS.

## 1. Page anatomy of the list view (observed)

Top-to-bottom, left-to-right:

- **App nav rail** (far left, Sales app): Home, Contacts, Accounts, Sales, Service, Marketing… — this is the *app switcher*, out of scope for your app (assuming it already has a sidebar).
- **Top object tab bar**: Lead (active), Contacts, Accounts, Opportunities, Products, Price Books, Calendar, Analytics, Invoices — each with a ▾ that opens recent records + "New". **Global search** field centered in the header.
- **Object header block**:
  - Small object label ("Lead") above a large **list-view name with a ▾ selector** ("All Open Leads ▾") and a **pin icon** (favorite / set default view).
  - **Object action buttons**, right-aligned: `New`, `Import`, `Add to Campaign`, `Send Email`, `Change Owner`, and a ▾ overflow ("Show More Actions").
- **Meta line** under the title: `"{N} items • Sorted by {column} • Filtered by {filter} • Updated {time} ago"`.
- **List toolbar** (right of the table header): `Search this list…` search box, then icon buttons: list controls gear (New / Rename / Save / Save As / Sharing / Edit List Filters / Delete), display-density menu, `Refresh`, column-sort, `Edit List` (toggle inline-edit mode), `Charts`, `Filters` (opens right filter panel).
- **Data table** (`role="grid"`, dense, sticky header):
  - Header **select-all checkbox**, then columns.
  - Each column header: label + sort toggle + a per-column action menu ("Show column actions": sort asc/desc, wrap/clip text).
  - Each row: **row checkbox**, cells, **inline-edit pencil** on editable cells, and a trailing **row action menu ▾** (Edit / Delete / Change Owner / …).
- **Utility bar** docked at the very bottom (e.g. "To Do List").

### Observed columns (order)
`#` (row number) · **Full Name** (sortable, default sort) · **Company** · **State/Province** · **Phone** · **Email** · **Lead Status** · **Created Date** · **Owner Alias** (owner) · **Action** (▾).

### Observed lead status
`Nurturing`. Standard reference lead-status ladder: `Open - Not Contacted` → `Working - Contacted` → `Nurturing` → `Closed - Converted` / `Closed - Not Converted (Unqualified)`.

## 2. Behaviors to replicate

| Behavior | Notes |
|---|---|
| **List-view selector** | Named, saved server-side views; one is default/pinned. Target app = query-string views (`?view=`), see modules.md. |
| **Live count + sort/filter meta line** | Rebuild verbatim as a meta line. |
| **Column sort** | Click header toggles asc/desc; one active sort at a time; server-side. |
| **List search** | "Search this list" = server-side `q` across name/company/role/email/phone/notes. |
| **Filters panel** | Right drawer with field conditions. Target app V1 = curated filters (status, priority, owner, channel, follow-up bucket, date range). |
| **Row selection + mass actions** | Header select-all + per-row checkbox enable bulk bar (change status / assign owner / flag / export). |
| **Row action menu** | Per-row ▾ with quick actions (open, edit, contact, outcome, follow-up, change status, delete). |
| **Inline edit** | The reference org edits cells in place. Target app V1 = row-action quick edits + detail page (simpler, fewer edge cases). Inline edit is a V2 nicety. |
| **Refresh** | Re-query button; cheap. |
| **New / Import** | Reuse existing pipeline create + CSV import. |

## 3. Record (lead detail) page anatomy — standard reference layout

Reproduce from this standard layout (rebranded):

- **Highlights panel** (header): icon + record name, key fields row (Company, Title, Phone, Email, Owner, Status/Priority), and right-aligned quick action buttons (New Task, New Event, Log a Call, Email, Edit, Change Owner, Convert).
- **Status Path** (horizontal chevrons) for lead status with a "Mark Status as Complete" primary button — the target app maps this to its own status ladder (see data-model.md).
- **Tabs / columns**: `Details` (field sections), `Activity` (activity composer: Log a Call / New Task / Email / New Event + a chronological timeline of past & upcoming activities grouped by month), `History`/related lists (Campaigns, Notes, Files), `Chatter` (skip).
- **Right column**: Activity timeline and/or related lists.

**Target-app adaptation:** if you have no activities table in V1, the timeline is **derived** from pipeline fields (first contact → follow-up 1 → follow-up 2 → analysis appointment → sales outcome). The "Path" becomes your own status ladder. Quick actions become the minimal-click endpoints in modules.md.

## 4. Reference → target-app concept mapping

| Reference | Target app | Backing field |
|---|---|---|
| Lead object | Pipeline entry | `commercial_pipeline_entries` row |
| List view ("All Open Leads") | Saved view ("Open leads", default) | `?view=` query filter |
| Lead Status (status ladder) | Lead status (target app) | new `status` col (derived fallback) |
| Owner Alias (Owner) | Owner | new `owner_id` FK (+ legacy `contact_owner` text) |
| Rating / Priority | Priority (0–5) | `priority` |
| — (custom) Lead score | Lead score (1–10) | `lead_score` |
| Activities / Tasks | Follow-up 1&2, appointments (derived timeline) | `follow_up_*`, `second_follow_up_*`, `*_appointment_set` |
| Lead Source | Lead source | new `lead_source` (optional) + `contact_method` (channel) |
| Convert to Opportunity | Sales outcome + contract value | `sales_outcome`, `contract_value` |
| Campaigns, Web-to-Lead, Einstein, Path automation, mass email | **Not in V1** | — |

## 5. Field-level mapping (columns)

| Reference column | Target-app column | Field | Status |
|---|---|---|---|
| Full Name | Contact | `lead_name` | exists |
| Company | Company | `company` | exists |
| Role | Role | `role` | exists |
| Industry | Industry | `business_area` | exists |
| State/Province | Location | `address` | exists |
| Phone | Phone | `phone` | **add** |
| Email | Email | `email` | **add** |
| Lead Status | Status | `status` | **add** (+ derived) |
| Created Date | Created on | `created_at` | exists |
| Owner Alias | Owner | `owner_id` | **add** |
| (Priority/Score/Follow-up/Outcome/Value) | same | existing rich fields | exists |
