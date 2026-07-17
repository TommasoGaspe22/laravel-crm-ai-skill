# Filter builder system

**Scope:** pannello filtri delle list view (trattato come modulo a sé). Fonte: **Osservato** (pannello reale su "Tutte le opportunità"); operatori dettagliati **Da verificare** (test dinamico dei singoli operatori ancora da fare).

## Osservato / Testato dinamicamente (2026-07-08)
- **Trigger:** pulsante **Filtri** nella toolbar (abilitato **solo** su viste reali es. "Tutte le opportunità", non su "Recenti").
- **Pannello a destra** ("Filtri", X per chiudere), non overlay bloccante (tabella visibile a sinistra). Header: **Annulla · Salva (▾ = Salva / Salva come)**.
- **Scope:** **"Filtra per titolare"** (Tutte / Mie / coda), valore "Tutte le opportunità".
- **Logica:** **"Corrispondenza di tutti questi filtri"** (= AND di default) + **"Aggiungi logica dei filtri"** (espressione booleana custom AND/OR/NOT con numeri di condizione).
- **Filtri:** **"Aggiungi filtro"** → chip **"Nuovo filtro*"** (evidenziato) da configurare **Campo → Operatore → Valore**; **X** per rimuovere; **"Rimuovi tutto"**.
- **Salvataggio:** Salva applica; "Salva come" crea una nuova vista dalla combinazione filtri.
- **Da verificare (residuo):** elenco operatori esatti per tipo campo (aprire il chip → campo → operatore); date relative; indicatori chip filtri attivi sopra la tabella; comportamento zero risultati; persistenza. *(Struttura confermata; i valori operatore standard dell'org di riferimento sono in §Architettura.)*

## Architettura Laravel (Proposto per Laravel)
Modulo dedicato (mirror dell'org di riferimento ma sicuro e performante):
- `FilterDefinition` (elenco filtri di una vista) · `FilterField` (campo filtrabile: nome, tipo, label) · `FilterOperator` (per tipo campo) · `FilterGroup` (AND/OR annidati) · `AppliedFilter` (campo+operatore+valore) · `FilterValueResolver` (parse valori, incl. **date relative** "oggi/±N giorni") · `QueryBuilderAdapter` (traduce in Eloquent/SQL) · `SavedView` (persistenza) · `FilterPolicy` (campi filtrabili per ruolo) · `FilterSerialization` (query-string ↔ struttura, viste condivisibili via URL).
- **Operatori per tipo:**
  - testo: `=`, `≠`, `contiene`, `non contiene`, `inizia con`
  - numero/importo: `=`, `≠`, `<`, `≤`, `>`, `≥`, `tra`
  - data: assolute (`=`,`<`,`>`,`tra`) + **relative** (`oggi`, `ieri`, `ultimi N giorni`, `questo mese`, `prossimi N giorni`)
  - picklist/enum: `in`, `non in`
  - booleano: `vero/falso`
  - owner/lookup: `= me`, `= utente`, `in team`
- **Query sicura:** whitelist campi+operatori (mai input diretto in SQL); binding parametrico (anti SQL-injection); indici sui campi filtrabili (`stage`, `owner_id`, `status`, date); timezone consistente su date relative; limiti di complessità (max N condizioni) per performance; evitare N+1 con eager-load.
- **Persistenza & URL:** filtri serializzati in query-string (viste bookmarkabili/condivisibili) + `saved_views` per salvarli.
- **UI:** `x-crm.filter-panel` (drawer): scope owner, lista condizioni (aggiungi/rimuovi/modifica), Applica/Reset, **chip filtri attivi** sopra la tabella, stato zero-risultati curato.

## Priorità
- **V1:** filtri curati per oggetto (stato/fase/owner/canale/bucket follow-up/date range/flag) via query-string + chip attivi + reset. AND semplice.
- **V2:** filter builder generico (campo/operatore/valore dinamici), date relative, salvataggio vista, OR.
- **V3:** gruppi di condizioni annidati, logica custom, filtri cross-oggetto.
- **Non replicare (V1):** logica booleana avanzata, filtri su campi formula/rollup.

## Open questions → `open-questions-and-assumptions.md`
Q1 (operatori per tipo, AND/OR/gruppi, date relative, zero risultati, salvataggio, persistenza) — **da testare dinamicamente** aprendo e applicando filtri reali.
