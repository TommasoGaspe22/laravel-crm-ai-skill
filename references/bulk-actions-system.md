# Bulk actions system

**Scope:** azioni massive su selezione multipla in list view. Fonte: **Osservato** (checkbox select-all + riga; azioni oggetto tipo "Cambia titolare", "Invia email"); dettaglio bulk **Da verificare** dinamicamente.

## Osservato / Testato dinamicamente (2026-07-08)
- Header **checkbox select-all** + checkbox per riga → selezione multipla. **Testato:** selezionate 2 righe → compare **"2 voci selezionate"** sotto il titolo.
- Con righe selezionate, le **azioni oggetto in header diventano azioni massive** (Testato, Lead): **Cambia titolare · Invia email · Aggiungi a campagna** (+ Nuovo/Importa). **Modifica elenco** → mass inline edit.
- **Row action ▾ (Testato, Lead):** `Modifica · Elimina · Cambia titolare · Modifica etichette`.
- **Da verificare:** limiti selezione (es. 200), conferma bulk, esito parziale, mass edit inline celle, dove sta "Elimina" massivo (probabile overflow ▾).

## Proposta Laravel (Proposto per Laravel)
- **UI:** `x-crm.bulk-action-bar` che appare quando `selectedIds.length > 0` (Alpine reactive), con azioni: **cambia stato/fase**, **assegna owner**, **flag/unflag**, **elimina** (admin), **esporta selezionati**, **mass edit** (V2).
- **Endpoint** `PATCH /crm/{obj}/bulk` con `ids[] + action + payload`: valida `ids` esistenti, **autorizza ogni record** (policy), esegue in **transazione**, ritorna esito (successi/falliti) + toast riepilogo.
- **Limiti:** cap selezione (es. 200) → oltre, chunk/queue job (V2). Prevenire timeout su SQLite/Aruba.
- **Rischio implementativo:** autorizzazione per-record (non solo globale); esito parziale (report righe fallite); idempotenza; audit per ogni record modificato.

## Priorità
- **V1:** select-all + bulk cambia stato/assegna owner/flag/elimina/esporta, in transazione, con conferma + toast. **V2:** mass inline edit, invio email massivo, chunk/queue per grandi selezioni. **V3:** azioni bulk custom, campagne.
- **Non replicare (V1):** invio email massivo reale, aggiungi a campagna.

## Open questions → `open-questions-and-assumptions.md`
Q6: barra bulk contestuale, limiti selezione, mass inline edit, esito parziale — da testare.
