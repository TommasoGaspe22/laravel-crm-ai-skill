# Opportunity pipeline & Kanban (indice)

Documento ombrello che collega i sistemi della pipeline Opportunità. Fonte: **Osservato/Testato**.

- **Oggetto & record page:** `opportunity-record-page.md` — campi (About/Status), azioni, related (Ruoli referenti/File/Fatture).
- **Fasi & transizioni:** `stage-transition-system.md` — Path `Qualify→…→Chiuso`, commit due-step, guida per fase (Testato: Qualify→Propose).
- **Kanban:** `kanban-system.md` — board per fase, conteggi + somme €, card con drag handle (Testato); drag&drop V2.
- **Pipeline & metriche:** `pipeline-system.md` — viste, forecast pesato, win rate.
- **Dati:** `data-model-and-migrations.md` (`crm_opportunities`).
- **Vincolo list view:** Kanban/Filtri solo su viste reali, non su "Recenti" (Testato — `list-view-system.md`).

**V1:** oggetto + fasi + Path + record page + vista "per fase" raggruppata + metriche base. **V2:** Kanban drag&drop + chiusura modale + forecast. **V3:** prodotti/quote/team.

**Da verificare:** drag&drop reale + conferme; chiusura Won/Lost (stato+motivo persa); auto-probabilità/forecast per fase; performance con molte card.
