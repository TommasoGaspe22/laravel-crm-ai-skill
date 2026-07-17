# List view system

**Scope:** viste elenco (list view) di tutti gli oggetti. Fonte: **Osservato/Testato** (liste popolate Lead/Opportunità, controlli, viste). Anatomia: `screen-anatomy.md` §A. Sotto-sistemi: `saved-views-system.md`, `table-system.md`, `column-management-system.md`, `bulk-actions-system.md`, `pagination-and-loading-system.md`, `split-view-system.md`, `filter-builder-system.md`.

## Indice
1. Struttura · 2. Controlli · 3. Modalità display · 4. Findings · 5. Laravel · 6. Priorità · 7. Open questions

## 1. Struttura (Osservato)
- Header: label oggetto + **nome vista con ▾ + pin** (default) + **azioni oggetto** (`Nuovo · Importa · …`).
- **Meta line:** "N elementi • Ordinati per {col} • Filtrati per {…} • Aggiornato {t} fa". (Testato: count si aggiorna con i dati; es. Opportunità "2 elementi".)
- **Toolbar:** ricerca in elenco · gear controlli · display selector · Aggiorna · Ordinamento colonne · Modifica elenco (inline) · Grafici · Filtri.
- **Tabella** densa: select-all + checkbox riga + celle (✎ editabili) + **azione riga ▾**.

## 2. Controlli (Osservato)
- **Gear "Controlli visualizzazione elenco":** `Nuovo · Clona · Rinomina · Impostazioni di condivisione · Seleziona i campi da visualizzare · Elimina · Reimposta larghezze colonna`. (Testato: menu reale.) → `saved-views-system.md`, `column-management-system.md`.
- **Selettore viste** (▾ sul nome): elenco viste (recenti/personali/standard). Testato per Opportunità: `In chiusura nel mese corrente/prossimo · Novità della settimana corrente · Opportunità in corso di realizzazione · Opportunità personali · Opportunità visualizzate recentemente · Recenti · Tutte le opportunità`.
- **Ordinamento:** click header colonna (asc/desc, uno alla volta). `Ordinamento colonne` per multi. → `table-system.md`.
- **Ricerca in elenco:** `q` server-side sui campi della vista.

## 3. Modalità display (Osservato/Testato)
Pulsante **"Seleziona visualizzazione elenco"** (icona griglia ▾) → **Tabella · Kanban · Doppia visualizzazione (Split)**. Vedi `table-system.md`, `kanban-system.md`, `split-view-system.md`.

## 4. Findings chiave (Testato)
- **La vista di sistema "Recenti" (Recently Viewed) NON supporta Filtri, Grafici, Kanban** ("non disponibili per questa visualizzazione elenco"). Solo le viste reali (standard/personali) li abilitano.
- Le azioni oggetto e le colonne cambiano per oggetto (vedi `source-crm-reference.md` + capture).
- Row action ▾ e select-all presenti su liste popolate.

## 5. Proposta Laravel (Proposto per Laravel)
- Controller `index` per oggetto: query server-side con **vista** (`?view=`), **ricerca** (`q`), **ordinamento** (`sort`/`dir` whitelisted), **filtri** (`filter-builder-system.md`), **paginazione** (`pagination-and-loading-system.md`). Eager-load per N+1.
- Componenti: `x-crm.page-header`, `x-crm.view-selector`, `x-crm.list-toolbar`, `x-crm.data-table`, `x-crm.filter-panel`, `x-crm.kanban` (vedi `component-catalog.md`).
- **Viste**: `App\Support\{Obj}ListViews` + tabella `saved_views` (V2). Distinzione "viste di sistema" (recenti/miei) senza filtri/kanban avanzati.
- Preferenze (colonne/densità/vista) persistite (session/localStorage V1; tabella V2).

## 6. Priorità
- **V1:** tabella densa (sort, ricerca, filtri base, viste predefinite, azioni riga, select+bulk essenziali, paginazione, stati vuoto/loading/errore). Vista "per fase" raggruppata (Opportunità).
- **V2:** Kanban drag&drop, split view, colonne configurabili+resize, saved views utente, filter builder avanzato, grafici.
- **V3:** viste condivise/permessi, grafici avanzati, import wizard ricco.
- **Non replicare (V1):** grafici embedded, doppia visualizzazione, condivisione viste granulare.

## 7. Open questions → `open-questions-and-assumptions.md`
Paginazione con molti record (Q13); persistenza preferenze; import wizard; grafici; azioni oggetto per oggetto.
