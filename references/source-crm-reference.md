# Reference org "All Open Leads" — anatomy & mapping to your app

Captured **read-only** from the live reference org (Italian locale). No records were modified; no real data is reproduced here. Use this as the UX target; rebuild with your own branding — never copy the source org's logos, asset URLs, or proprietary CSS.

## 1. Page anatomy of the list view (observed)

Top-to-bottom, left-to-right:

- **App nav rail** (far left, Sales app): Home, Contacts, Accounts, Sales, Service, Marketing… — this is the *app switcher*, out of scope for your app (assuming it already has a sidebar).
- **Top object tab bar**: Lead (active), Referenti (Contacts), Account, Opportunità, Prodotti, Listini prezzi, Calendario, Analytics, Fatture — each with a ▾ that opens recent records + "New". **Global search** field centered in the header.
- **Object header block**:
  - Small object label ("Lead") above a large **list-view name with a ▾ selector** ("Tutti i lead aperti ▾") and a **pin icon** (favorite / set default view).
  - **Object action buttons**, right-aligned: `Nuovo`, `Importa`, `Aggiungi a campagna`, `Invia email`, `Cambia titolare`, and a ▾ overflow ("Mostra un'altra azione").
- **Meta line** under the title: `"{N} elementi • Ordinati per {colonna} • Filtrati per {filtro} • Aggiornato {tempo} fa"`.
- **List toolbar** (right of the table header): `Cerca in questo elenco…` search box, then icon buttons: list controls gear (New / Rename / Save / Save As / Sharing / Edit List Filters / Delete), display-density menu, `Aggiorna` (refresh), column-sort, `Modifica elenco` (toggle inline-edit mode), `Grafici` (charts), `Filtri` (opens right filter panel).
- **Data table** (`role="grid"`, dense, sticky header):
  - Header **select-all checkbox**, then columns.
  - Each column header: label + sort toggle + a per-column action menu ("Mostra azioni colonna": sort asc/desc, wrap/clip text).
  - Each row: **row checkbox**, cells, **inline-edit pencil** on editable cells, and a trailing **row action menu ▾** (Edit / Delete / Change Owner / …).
- **Utility bar** docked at the very bottom (e.g. "To Do List").

### Observed columns (order)
`#` (row number) · **Nome e cognome** (sortable, default sort) · **Società** · **Stato/Provincia** · **Telefono** · **Email** · **Stato lead** · **Data creazione** · **Alias titolare** (owner) · **Azione** (▾).

### Observed lead status
`Nurturing`. Standard reference lead-status ladder: `Open - Not Contacted` → `Working - Contacted` → `Nurturing` → `Closed - Converted` / `Closed - Not Converted (Unqualified)`.

## 2. Behaviors to replicate

| Behavior | Notes |
|---|---|
| **List-view selector** | Named, saved server-side views; one is default/pinned. Target app = query-string views (`?view=`), see modules.md. |
| **Live count + sort/filter meta line** | Rebuild verbatim as a meta line. |
| **Column sort** | Click header toggles asc/desc; one active sort at a time; server-side. |
| **List search** | "Cerca in questo elenco" = server-side `q` across name/company/role/email/phone/notes. |
| **Filters panel** | Right drawer with field conditions. Target app V1 = curated filters (stato, priorità, owner, canale, follow-up bucket, date range). |
| **Row selection + mass actions** | Header select-all + per-row checkbox enable bulk bar (change status / assign owner / flag / export). |
| **Row action menu** | Per-row ▾ with quick actions (open, edit, contatta, esito, follow-up, cambia stato, elimina). |
| **Inline edit** | The reference org edits cells in place. Target app V1 = row-action quick edits + detail page (simpler, fewer edge cases). Inline edit is a V2 nicety. |
| **Refresh** | Re-query button; cheap. |
| **New / Import** | Reuse existing pipeline create + CSV import. |

## 3. Record (lead detail) page anatomy — standard reference layout

Reproduce from this standard layout (rebranded):

- **Highlights panel** (header): icon + record name, key fields row (Company, Title, Phone, Email, Owner, Status/Priority), and right-aligned quick action buttons (New Task, New Event, Log a Call, Email, Edit, Change Owner, Convert).
- **Status Path** (horizontal chevrons) for lead status with a "Mark Status as Complete" primary button — the target app maps this to its own status ladder (see data-model.md).
- **Tabs / columns**: `Dettagli` (field sections), `Attività` (activity composer: Log a Call / New Task / Email / New Event + a chronological timeline of past & upcoming activities grouped by month), `Cronologia`/related lists (Campaigns, Notes, Files), `Chatter` (skip).
- **Right column**: Activity timeline and/or related lists.

**Target-app adaptation:** if you have no activities table in V1, the timeline is **derived** from pipeline fields (first contact → follow-up 1 → follow-up 2 → analysis appointment → sales outcome). The "Path" becomes your own status ladder. Quick actions become the minimal-click endpoints in modules.md.

## 4. Reference → target-app concept mapping

| Reference | Target app | Backing field |
|---|---|---|
| Lead object | Voce pipeline commerciale | `commercial_pipeline_entries` row |
| List view ("Tutti i lead aperti") | Vista salvata ("Lead aperti", default) | `?view=` query filter |
| Stato lead (status ladder) | Stato lead (target app) | new `status` col (derived fallback) |
| Alias titolare (Owner) | Responsabile | new `owner_id` FK (+ legacy `contact_owner` text) |
| Rating / Priority | Priorità (0–5) | `priority` |
| — (custom) Lead score | Lead score (1–10) | `lead_score` |
| Activities / Tasks | Follow-up 1&2, appuntamenti (derived timeline) | `follow_up_*`, `second_follow_up_*`, `*_appointment_set` |
| Lead Source | Fonte lead | new `lead_source` (optional) + `contact_method` (canale) |
| Convert to Opportunity | Esito vendita + valore contratto | `sales_outcome`, `contract_value` |
| Campaigns, Web-to-Lead, Einstein, Path automation, mass email | **Not in V1** | — |

## 5. Field-level mapping (columns)

| Reference column | Target-app column | Field | Status |
|---|---|---|---|
| Nome e cognome | Referente | `lead_name` | exists |
| Società | Azienda | `company` | exists |
| Ruolo | Ruolo | `role` | exists |
| Settore | Settore | `business_area` | exists |
| Stato/Provincia | Località | `address` | exists |
| Telefono | Telefono | `phone` | **add** |
| Email | Email | `email` | **add** |
| Stato lead | Stato | `status` | **add** (+ derived) |
| Data creazione | Creato il | `created_at` | exists |
| Alias titolare | Responsabile | `owner_id` | **add** |
| (Priority/Score/Follow-up/Outcome/Value) | idem | existing rich fields | exists |
