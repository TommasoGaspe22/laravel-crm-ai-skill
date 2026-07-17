# Split view (Doppia visualizzazione)

**Scope:** vista a due pannelli (lista a sinistra, record a destra). Fonte: **Osservato** (opzione "Doppia visualizzazione" nel display selector); comportamento **Da verificare** dinamicamente.

## Osservato / Testato dinamicamente (2026-07-08)
- Opzione **"Doppia visualizzazione"** nel toggle display (accanto a Tabella/Kanban).
- **Testato:** layout confermato = **lista compatta a sinistra** (colonna Fase, card nome/account/data/importo, ricerca, checkbox) + **pannello record a destra** con empty state **"Nessun record selezionato — Per iniziare, apri un record dall'elenco."**; **freccia di collasso (◀)** tra i pannelli.
- **Da verificare:** contenuto anteprima al click di un record, larghezza/persistenza pannelli, azioni nell'anteprima, responsive.

## Proposta Laravel (Proposto per Laravel)
- **V2/V3** (non V1). Route `?display=split`. Layout: colonna lista (riuso `x-crm.data-table` compatta) + pannello record (riuso `x-crm.page-header` + sezioni chiave), caricato via Alpine/`fetch` parziale sul click (evita full reload).
- **Rischio implementativo:** raddoppia query per pagina; caricare l'anteprima lazy; responsive → collassa a lista su mobile.

## Priorità
- **V2/V3.** In V1 il flusso è: lista → click → record page piena (sufficiente). Split view solo se il team fa triage massivo.
- **Non replicare (V1).**

## Open questions → `open-questions-and-assumptions.md`
Q10: layout/azioni/persistenza split view.
