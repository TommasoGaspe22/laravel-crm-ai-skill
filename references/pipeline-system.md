# Pipeline system (Opportunità)

**Scope:** la pipeline di vendita a valore basata sulle Opportunità. Documento ombrello: rimanda a `opportunity-record-page.md`, `stage-transition-system.md`, `kanban-system.md`. Fonte: **Osservato/Testato**.

## Concetti (Osservato/Testato)
- **Opportunità** = trattativa con **Fase** (pipeline), **Ammontare**, **Data chiusura**, **Probabilità**, **Categoria previsione**, **Fase successiva**, Account+Referente, Titolare.
- **Fasi (reali):** `Qualify → Meet & Present → Propose → Negotiate → Closed Won / Closed Lost`. Target app (IT): `Qualifica → Presentazione → Proposta → Negoziazione → Vinta / Persa`.
- **Path** con guida per fase (Testato: transizione Qualify→Propose).
- **Kanban** per fase con conteggi + somme € (Testato).
- **Chiusura** Won/Lost terminale (**Da verificare** modale stato+motivo persa).

## Metriche pipeline (Proposto per Laravel)
- Per fase: `count`, `sum(amount)`, `avg(probability)`.
- **Pipeline pesata:** `sum(amount * probability)` (forecast semplice).
- Aperte vs chiuse (win rate = won/(won+lost)); scadute (close_date < oggi & non chiuse).
- Viste: Aperte · Mie · In chiusura questo mese · Vinte · Perse · Per fase (raggruppata) · Kanban.

## Laravel
`CrmOpportunity` + `ChangeStage`/`CloseOpportunity` (`stage-transition-system.md`), Kanban (`kanban-system.md`), record page (`opportunity-record-page.md`), dati/indici (`data-model-and-migrations.md`).

## Priorità
- **V1:** oggetto Opportunità + fasi + Path + record page + vista "per fase" raggruppata + metriche base (conteggi/somme/pesata) + chiusura won/lost.
- **V2:** Kanban drag&drop, forecast, viste avanzate.
- **V3:** prodotti/line items, quote, forecasting avanzato, team.

## Open questions
Chiusura won/lost (modale/campi); auto-probabilità per fase; forecast category valori; line items.
