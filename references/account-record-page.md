# Account record page

**Fonte:** Osservato/Testato (record reale + eliminazione). Base: `record-page-system.md`. Campi: `field-catalog.md` (Account) + `data-model-and-migrations.md`.

- **Header:** `Account` / nome. Azioni: icona **org-chart** · **Nuovo referente · Nuova opportunità · Modifica · ▾**. Nessun Path (Account non ha fasi).
- **Sezioni (Osservato):** `About` (Nome account, Sito Web, Tipo, Descrizione, Società controllante [lookup self], Titolare account) · `Get in Touch` (Telefono, Indirizzo fatturazione, Indirizzo spedizioni) · `History`.
- **Attività:** composer + timeline.
- **Related list (Osservato):** **Referenti · Opportunità · Casi** (+ Attività, File). Da qui i quick-create "Nuovo referente"/"Nuova opportunità" (Testato: creazione opp dal contesto account funziona meglio del form standalone).
- **Eliminazione (Testato):** azione `Elimina` → dialog "Eliminare questo account?" (titolo "Elimina account") → conferma → redirect a list. Cascata su Referenti/Opportunità **Da verificare**.
- **Laravel:** `CrmAccount` (hub) → `hasMany contacts/opportunities/activities`; `belongsTo parent`. Related list con conteggi + quick-create. Vedi `data-model-full.md`.
- **Da verificare:** campi completi New Account (Tipo picklist valori); cascata eliminazione; gerarchia società controllante (org chart).
