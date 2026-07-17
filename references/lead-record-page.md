# Lead record page

**Fonte:** Osservato/Testato (record reale + conversione). Base comune: `record-page-system.md`. Campi: `field-catalog.md` (Lead).

- **Header:** `Lead` / nome (es. "Sig.ina Tommaso Gasperoni"). Azioni: **Converti · Cambia titolare · Modifica · ▾**.
- **Path (Stato lead):** `Nuovo → Contattato → Nurturing → Non qualificato → Convertito` (commit due-step, `stage-transition-system.md`). `Convertito` raggiungibile solo via conversione.
- **Sezioni (sinistra, Osservato):** `About` (Stato lead, Nome e cognome, Società, Qualifica, Sito Web, Descrizione, Titolare lead, Valutazione) · `Get in Touch` (Telefono, Email, Indirizzo) · `Segment` (N. dipendenti, Reddito annuale, Fonte del lead, Settore) · `History`.
- **Attività:** composer + timeline (`activity-and-timeline-system.md`).
- **Related list (destra):** `File`. (Prima della conversione il Lead ha poche relazioni.)
- **Azione chiave — Converti** (`lead-conversion-system.md`): crea Account+Referente+Opportunità.
- **Post-conversione (Testato):** stato → Convertito, campi read-only, `Converti` non più disponibile.
- **Laravel:** su `commercial_pipeline_entries` + `/crm/leads/{id}` (modulo Lead esistente, `modules.md`) esteso con Path stato + azione Converti. Inline edit + timeline + quick actions.
- **Da verificare:** trasferimento note/file/attività alla conversione; History/audit.
