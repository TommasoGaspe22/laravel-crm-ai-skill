# Kanban system (Opportunità pipeline board)

**Scope:** vista Kanban delle Opportunità raggruppata per Fase. Fonte: **Testato dinamicamente** (2026-07-08, board reale con 2 opportunità demo in fasi diverse) + **Osservato**. Blueprint Laravel (SortableJS, V2).

## Indice
1. Come si attiva · 2. Anatomia (testato) · 3. Comportamenti · 4. Limitazioni Osservate · 5. Laravel · 6. Priorità · 7. Open questions

## 1. Attivazione (Osservato/Testato)
- Toolbar list view → pulsante **"Seleziona visualizzazione elenco"** (icona griglia con ▾, a destra del gear "Controlli visualizzazione elenco") → opzioni **Tabella / Kanban / Doppia visualizzazione (Split)**.
- **⚠️ Vincolo (Osservato):** disponibile solo su list view reali (es. **"Tutte le opportunità"**), **NON** sulla vista di sistema **"Recenti" (Recently Viewed)**, dove Kanban/Filtri/Grafici sono disabilitati ("I filtri/grafici non sono disponibili per questa visualizzazione elenco"). → In Laravel: le viste "recenti/sistema" non offrono kanban/filtri; solo le viste salvate/utente sì.

## 2. Anatomia board (Testato dinamicamente)
- **Colonne = valori picklist di raggruppamento** (default **Fase**): `Qualify · Meet & Present · Propose · Negotiate · Closed Won` (le closed sono colonne finali). Header colonna = **chevron blu** con **nome fase + (conteggio)**.
- **Somma importi per colonna:** sotto l'header, totale `€` (es. `0 €` perché Ammontare vuoto). → aggrega `SUM(amount)` per fase.
- **Card opportunità:** maniglia **drag** (⋮⋮) a sinistra, **nome/account** (link), **data chiusura**, menu **▾ azioni** per card. (Con Ammontare valorizzato comparirebbe anche l'importo.)
- **Meta line** identica alla tabella: "N elementi • Ordinati per … • Aggiornato …".
- **Filtri** e ricerca condivisi con la vista tabella (pannello Filtri sulla destra).

## 3. Comportamenti (Osservato / Testato)
- **Drag & drop card tra colonne** → cambia Fase. Osservato: **maniglia drag (⋮⋮) presente** su ogni card. **Testato:** il drag automatico via Playwright (HTML5 dragTo) **non** aziona il drop custom dell'org di riferimento (la card non si sposta) → il meccanismo esiste ma va verificato manualmente; in Laravel useremo **SortableJS** (dip. V2) con `PATCH stage` al drop. Verso Closed → apre la modale **"Chiudi quest'opportunità"** (fase chiusa required, vedi `stage-transition-system.md`).
- **Raggruppamento alternativo:** l'org di riferimento consente cambiare il campo di raggruppamento (Da verificare quali campi).
- **Colonne con 0 record:** mostrate comunque (es. Meet & Present (0), Negotiate (0)).
- **Chiusura:** spostare in Closed Won/Lost → **Da verificare** (probabile dialog motivo/stato chiusura).
- **Performance con molte card:** Da verificare (non popolabile realisticamente nella demo → target in `testing-strategy.md`).

## 4. Limitazioni / Non replicare
- **Non replicare (V1):** raggruppamenti multipli avanzati, forecast inline nel kanban, kanban su oggetti senza pipeline.
- La board dell'org di riferimento è client-side pesante; nel target app è **V2** (dopo che `stage` è persistito e stabile).

## 5. Blueprint Laravel (Proposto per Laravel)
- **Route/vista:** `/dashboard/crm/opportunities?view=…&display=kanban`.
- **Componente** `x-crm.kanban` + `x-crm.kanban-card` (vedi `component-catalog.md`). Colonne = `config('crm.crm_opportunity_stages')`; card = opportunità.
- **Dati:** query opportunità della vista, raggruppate per `stage`; per colonna: `count()` + `sum(amount)`. Eager-load account per evitare N+1.
- **Drag & drop:** **SortableJS** (dipendenza V2). On drop → `PATCH /opportunities/{id}/stage {stage}` (Action `ChangeStage`, vedi `stage-transition-system.md`): valida transizione, aggiorna `stage`/`probability`/`is_closed`, scrive `crm_activities` "cambio fase", ritorna toast. Ottimistic UI con rollback su errore.
- **Chiusura (Won/Lost):** drop su colonna closed → modale "motivo chiusura" prima del commit.
- **Rischio implementativo:** concorrenza (due utenti spostano la stessa card) → usare `updated_at` optimistic lock; validare che la fase target sia ammessa; permessi (policy `update`).
- **Accessibilità:** drag&drop deve avere alternativa tastiera (menu "Sposta a fase…" nella card ▾).

## 6. Priorità
- **V1:** vista "per fase" **raggruppata read-only** (lista raggruppata per stage con conteggi/somme) + cambio fase via menu card/record Path (senza drag).
- **V2:** Kanban con **drag & drop** (SortableJS), somme per colonna, modale chiusura.
- **V3:** raggruppamenti alternativi, WIP limit, forecast inline.

## 7. Open questions → `open-questions-and-assumptions.md`
Q2 (drag&drop reale + conferme), chiusura Won/Lost dialog, raggruppamenti alternativi, performance.
