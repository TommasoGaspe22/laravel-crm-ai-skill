# Screen-by-screen anatomy (captured, pixel-level)

Every CRM screen archetype, described so it can be rebuilt to match the reference org. Captured read-only from the org (Italian). Build these as your own branded Blade components (see `component-catalog.md`). Measurements are relative — match proportions/spacing, not exact px.

Legend: `▾` overflow/menu · `✎` inline-edit pencil · `*` required.

---

## A. List view (es. Lead / Account / Opportunità)

Region-by-region, top→bottom:

1. **Trial/promo bar** (reference-platform-only) — DO NOT replicate.
2. **Global header**: logo left · centered **global search** (`Cerca…`) · right icons (help, setup, notifications w/ badge, avatar). Target app: reuse existing dashboard header; add the centered search.
3. **Object tab bar**: horizontal tabs per object (Lead · Referenti · Account · Opportunità · Prodotti · Listini · Calendario · Analytics · Fatture), each with a ▾ that shows recent records + "Nuovo". Active tab underlined. Target app: a top object-nav strip inside the CRM area.
4. **Object header row**:
   - Left: small object label + **list-view name with ▾ selector** + **pin** (imposta come predefinita).
   - Right: **object action buttons** (es. `Nuovo · Importa · Aggiungi a campagna · Invia email · Cambia titolare`) + ▾ overflow.
5. **Meta line**: `"{N} elementi • Ordinati per {col} • Filtrati per {filtro} • Aggiornato {t} fa"`.
6. **List toolbar** (right-aligned icon row):
   - `Cerca in questo elenco…` search box.
   - **List-view controls gear** → menu (reale): `Nuovo · Clona · Rinomina · Impostazioni di condivisione · Seleziona i campi da visualizzare · Elimina · Reimposta larghezze colonna · —— · Tabella · Kanban · Doppia visualizzazione`.
   - **Display-density**, **Aggiorna** (refresh), **Ordinamento colonne**, **Modifica elenco** (toggle inline-edit), **Grafici** (charts), **Filtri** (opens right panel).
7. **Data table** (`role="grid"`, dense, sticky header):
   - Header select-all checkbox + columns; each header: label + sort caret (`aria-sort`) + column-action ▾ (Ordina crescente/decrescente, A capo/Taglia testo, larghezza).
   - Row: checkbox · cells (editable cells show `✎` on hover) · trailing **row action ▾** (Modifica, Elimina, Cambia titolare…). First cell (name) is a link to the record.
8. **Display modes** (from the gear): **Tabella** (default), **Kanban** (board grouped by a picklist — for Opportunità grouped by *Fase*), **Doppia visualizzazione** (split: list left, record preview right).
9. **Utility bar** bottom (To Do List) — optional, skip in V1.

**States:** empty (curated per view), loading skeleton rows, error/unavailable panel, selected-rows bulk bar.

---

## B. Record (detail) page — Lead / Account / Contact / Opportunità

Consistent 3-column reference layout:

1. **Record header**:
   - Left: object label + **record name** (large) + object icon.
   - Right: **primary action buttons** + ▾ overflow. Real per object:
     - Lead: `Converti · Cambia titolare · Modifica · ▾`
     - Account: `(org-chart icon) · Nuovo referente · Nuova opportunità · Modifica · ▾`
     - Contact: `Nuova opportunità · Modifica · ▾`
     - Opportunità: `Modifica · ▾` (+ stage path actions)
2. **Path component** (only Lead & Opportunità): horizontal chevrons of the stages, completed ones tinted, current one dark, future ones grey; right side primary button **"Contrassegna Stato come Completo/a"** (or "Seleziona Stato Convertito" on the last step). See component-catalog.
   - Lead stages (reale): `Nuovo → Contattato → Nurturing → Non qualificato → Convertito`.
   - Opportunità stages (reale): `Qualifica → Presentazione → Proposta → Negoziazione → Vinta → Persa`.
3. **Left column — Detail section cards** (collapsible, real section names from this org's layout): `About`, `Get in Touch`, `Segment` (Lead), `History`. Each holds field rows with label + value + `✎` inline-edit. Fields per object are in `objects-catalog.md`.
4. **Middle column — Activity panel**:
   - **Composer buttons** row: `Email · Nuova operazione/Evento · Registra una chiamata · Nuovo evento` (each icon + ▾).
   - Toggle "Mostra solo le attività con approfondimenti".
   - Filter line: `Entro 2 mesi • Tutte le attività • Tutti i tipi` + gear.
   - `Aggiorna · Espandi tutto`.
   - Collapsible groups: **"Imminenti e scadute"**, then past by month. Empty state: "Nessuna attività da mostrare…".
   - `Mostra tutte le attività` button.
5. **Right column — Related lists / cards**: cards like Referenti, Opportunità, Casi, File (each `Titolo (N)` + rows + "Visualizza tutto"). (Org also shows a Slack card — skip.)

**Target-app note:** in V1 the middle Activity panel = the new `crm_activities` timeline + quick-log composer; related lists (right) render the object's relations from `data-model-full.md`.

---

## C. Create / Edit modal (es. Crea opportunità)

- Centered **modal** with title `Crea {oggetto}` / `Modifica {oggetto}`, X to close.
- Top-right note `* = Informazioni richieste`.
- **Field sections** (grey section headers) grouping fields. Opportunità (reale): **About** (`*Nome opportunità`, `*Nome account` lookup, `*Data chiusura` date, `Ammontare`, `Descrizione`, `Titolare`) + **Status** (`*Fase` picklist, `Probabilità (%)`, `*Categoria di previsione`, `Fase successiva`).
- Inputs: text, textarea, date-picker (calendar icon), number, **lookup** (search field w/ magnifier), **picklist** (combobox ▾).
- **Footer**: `Annulla · Salva e Nuovo · Salva` (primary).
- Inline validation on required fields; error text under the field.

---

## D. Convert modal (Lead → Account + Referente + Opportunità)  *(reale)*

Title **"Converti lead"**, X to close. Three stacked sections, each with a **Crea / Scegli** radio pair:

1. **Account**: `Crea` (`*Nome account` prefilled from Società) **oppure** `Scegli` (search "Cerca account corrispondenti", shows N corrispondenze).
2. **Referente**: `Crea` (name prefilled) **oppure** `Scegli` (shows "N corrispondenza Referente rilevata").
3. **Opportunità**: `Crea` (opp name prefilled `"{azienda}-"`) + checkbox **"Non creare un'opportunità alla conversione"** **oppure** `Scegli`.
- Bottom: `*Titolare record` (owner lookup) · `*Stato convertito` (picklist, default "Qualificato").
- Footer: `Annulla · Converti` (primary).
- Helper text: *"Trasformeremo questo lead in tre nuovi record: Account, Referente, Opportunità."*

**Target app:** this is the `convertLead()` flow in `data-model-full.md` — the UI is a modal with these three find-or-create sections.

---

## E. Kanban (Opportunità per fase)

Board with one **column per stage** (`Qualifica … Vinta/Persa`), cards = opportunities (name, account, amount, close date), **drag & drop** a card to change stage (SortableJS, deferred dep). Column header shows count + sum(amount). Reachable from the list-view gear → "Kanban".

---

## Cross-cutting behaviors to replicate

- **Inline edit**: hover a field/cell → `✎`; click → input; multiple edits batch into a "Salva/Annulla" footer bar.
- **Sort**: click header toggles asc/desc, single active sort, `aria-sort` reflected.
- **Bulk**: header checkbox selects visible; bulk bar with mass actions; mass inline-edit via "Modifica elenco".
- **Saved views**: selector + pin default; controls menu (rename/save/save-as/share/filters/delete).
- **Toasts**: success/error toast after actions (top, auto-dismiss).
- **Keyboard/a11y**: focusable controls, `aria-label` on icon buttons, focus-trap in modals, Esc closes.
