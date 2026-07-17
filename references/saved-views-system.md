# Saved views system

**Scope:** viste salvate (list view). Fonte: **Osservato** (selettore viste + menu controlli).

## Osservato
- **Selettore** (▾ sul nome vista) con: viste **standard** (es. Opportunità: `In chiusura nel mese corrente/prossimo`, `Novità della settimana corrente`, `Opportunità in corso di realizzazione`, `Tutte le opportunità`), **personali** (`Opportunità personali`), **di sistema** (`Recenti`/`Recently Viewed`, `Opportunità visualizzate recentemente`). Ricerca viste ("Cerca negli elenchi…"). **Pin** = imposta come default.
- **Gestione** (gear controlli): `Nuovo · Clona · Rinomina · Impostazioni di condivisione · Elimina` (+ Seleziona campi, Reimposta larghezze).
- **Vincolo (Testato):** le viste **di sistema** (Recenti) non supportano Filtri/Kanban/Grafici; solo standard/personali sì.
- **Da verificare:** creazione vista (definizione filtri + colonne + condivisione), condivisione (chi può vedere), viste condivise.

## Proposta Laravel (Proposto per Laravel)
- **V1:** viste predefinite in codice (`App\Support\{Obj}ListViews`) selezionabili via `?view=` (mirror standard views), + pin default (preferenza utente).
- **V2:** tabella **`saved_views`** (`id, object, name, owner_id, is_shared, filters_json, columns_json, sort_json, is_default`) → viste utente create/clonate/rinominate/eliminate; UI selettore + gestione (`x-crm.view-selector`). Serializzazione filtri = `filter-builder-system.md`.
- **Policy:** viste personali (owner) vs condivise (visibili a staff/team); solo owner/admin modifica/elimina.
- **Rischio implementativo:** validare `filters_json`/`columns_json` (whitelist campi); non fidarsi dei valori per query (anti-injection); default per utente.

## Priorità
- **V1:** viste predefinite + pin default. **V2:** saved views utente (CRUD + condivisione base). **V3:** condivisione granulare/team, viste dinamiche.
- **Non replicare (V1):** condivisione per ruolo/coda complessa.

## Open questions → `open-questions-and-assumptions.md`
Q7 flusso creazione/clona/rinomina/condivisione vista; distinzione standard/personali/sistema; condivisione.
