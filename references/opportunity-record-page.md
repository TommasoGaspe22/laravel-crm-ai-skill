# Opportunità record page

**Fonte:** Osservato/Testato (record reale + stage transition + inline edit + attività). Base: `record-page-system.md`. Campi: `field-catalog.md` (Opportunità).

- **Header:** `Opportunità` / nome (es. "CRM-SKILL-Azienda Manufacturing 001-"). Azioni: **Nuovo evento · Nuova operazione · Registra una chiamata · Modifica · Crea nuova fattura · ▾**.
- **Path (Fase):** `Qualify → Meet & Present → Propose → Negotiate → Chiuso` (Chiuso = Won/Lost). Pannello **"Guida per il successo"** per fase (Testato: guida "Make the offer…" su Propose). Commit due-step. Vedi `stage-transition-system.md`. **Testato:** Qualify→Propose.
- **Sezioni (Osservato):** `About` (Nome opportunità, Nome account [lookup], Data chiusura, Ammontare, Descrizione, Titolare opportunità) · `Status` (Fase, Probabilità %, Categoria di previsione, Fase successiva).
- **Inline edit (Testato/Osservato):** ✎ per campo (es. Ammontare) → input + barra Annulla/Salva.
- **Attività (Testato):** "Registra una chiamata" → item in timeline; "Nuova operazione" (Task: Oggetto+Scadenza).
- **Related list (Osservato):** **Ruoli dei referenti · File · Fatture**.
- **Kanban:** l'opportunità è una card nel Kanban per fase (`kanban-system.md`).
- **Laravel:** `CrmOpportunity` `belongsTo account`, `primary_contact`, `hasMany items` (V3), morph activities. Fase = enum config; `is_closed/is_won` derivati; probabilità mappata da fase. Vedi `data-model-full.md`.
- **Da verificare:** chiusura Won/Lost (modale stato+motivo persa); valori Categoria previsione; auto-probabilità per fase; line items/prodotti.
