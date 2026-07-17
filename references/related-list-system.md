# Related list system

**Scope:** liste correlate sulle record page. Fonte: **Osservato** (related list reali per oggetto).

## Osservato
- Card nella colonna destra/related con **titolo + (conteggio)**, alcune righe compatte, link **"Visualizza tutto"**, e **quick-create** (es. "Nuovo referente"/"Nuova opportunità" dall'header Account).
- Related per oggetto: **Account** → Referenti · Opportunità · Casi (+ Attività, File). **Referente** → Opportunità · Casi · File. **Opportunità** → Ruoli dei referenti · File · Fatture. **Lead** → File.
- **Testato:** creare un'opportunità dal contesto Account (related/azione) pre-collega l'account (più affidabile del form standalone).
- **Da verificare:** azioni inline nelle righe related (modifica/elimina), paginazione related, colonne mostrate, "Visualizza tutto" → list view filtrata.

## Proposta Laravel (Proposto per Laravel)
- **Componente** `x-crm.related-list`: props `title`, `:rows`, `:columns`, `createRoute`, `viewAllRoute`. Mostra prime N righe + conteggio + "Visualizza tutto" (→ list view dell'oggetto correlato filtrata sul parent) + quick-create (modale con parent pre-compilato).
- **Dati:** relazioni Eloquent (`hasMany`/`belongsToMany`) con `withCount` per i badge; eager-load per N+1; limit + "visualizza tutto".
- **Quick-create:** riusa `x-crm.record-modal` con `foreign key` pre-settata (come "Nuova opportunità" da Account).
- **Polimorfiche:** Attività/File/Note come related polimorfiche (`what` morph) → related list generica per qualsiasi oggetto.
- **Rischio implementativo:** N+1 su molte related; permessi (mostra solo record visibili); conteggi costosi (cache/withCount).

## Priorità
- **V1:** related list principali (Account→Referenti/Opportunità; Opportunità→Ruoli referenti; +Attività/File) con conteggo, prime righe, quick-create, "visualizza tutto".
- **V2:** azioni inline nelle righe, paginazione related, colonne configurabili.
- **V3:** related list custom, related di related.

## Open questions → `open-questions-and-assumptions.md`
Azioni inline/paginazione related; colonne related; "Ruoli dei referenti" (M:N opportunità-referente).
