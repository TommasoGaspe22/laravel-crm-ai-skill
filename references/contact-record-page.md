# Contact (Referente) record page

**Fonte:** Osservato (record reale + creazione via conversione). Base: `record-page-system.md`. Campi: `field-catalog.md` (Referente).

- **Header:** `Referente` / nome (es. "Mario CRM-SKILL-Lead Nuovo 001"). Azioni: **Nuova opportunità · Modifica · ▾**. Nessun Path.
- **Sezioni (Osservato):** `About` (Nome e cognome, Nome account [lookup], Qualifica, Fa capo a [lookup self], Descrizione, Titolare referente) · `Get in Touch` (Telefono, Email, Indirizzo postale) · `History`.
- **Attività:** composer + timeline (un referente è il "who"/Nome delle attività).
- **Related list (Osservato):** **Opportunità · Casi · File** (+ Attività). Ruolo del referente nelle opportunità = related "Ruoli dei referenti" lato Opportunità.
- **Creazione via conversione (Testato):** email/telefono trasferiti dal Lead; collegato all'Account creato.
- **Laravel:** `CrmContact` `belongsTo account`, self `reports_to`, `hasMany opportunities` (come referente primario) + `belongsToMany opportunities` via ruoli (`crm_opportunity_contact_roles`, V2).
- **Da verificare:** campi completi New Contact; "Fa capo a" (gerarchia); Ruoli referenti su opportunità.
