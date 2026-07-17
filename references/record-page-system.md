# Record page system

**Scope:** struttura comune delle pagine record (Lead/Account/Referente/Opportunità). Fonte: **Osservato/Testato** (screenshot reali + interazioni). Per i dettagli per-oggetto vedi `lead-record-page.md`, `account-record-page.md`, `contact-record-page.md`, `opportunity-record-page.md`. Anatomia visiva in `screen-anatomy.md` §B.

## Struttura (Osservato) — layout a 3 colonne
1. **Header:** icona oggetto + label + **nome record** (grande) + gruppo **azioni** a destra (+ ▾ overflow). Azioni per oggetto:
   - Lead: `Converti · Cambia titolare · Modifica · ▾`
   - Account: `Nuovo referente · Nuova opportunità · Modifica · ▾` (+ icona org-chart)
   - Referente: `Nuova opportunità · Modifica · ▾`
   - Opportunità: `Nuovo evento · Nuova operazione · Registra una chiamata · Modifica · Crea nuova fattura · ▾`
2. **Path** (solo Lead & Opportunità): chevron stadi + guida per stadio + pulsante commit. Vedi `stage-transition-system.md`.
3. **Colonna sinistra — Detail sections** (card collassabili) con field rows + **inline edit ✎**. Sezioni reali per oggetto (Osservato): Lead `About/Get in Touch/Segment/History`; Account/Contact `About/Get in Touch/History`; Opportunità `About/Status`.
4. **Colonna centrale — Activity panel** (composer + timeline). Vedi `activity-and-timeline-system.md`.
5. **Colonna destra — Related lists** (card `Titolo (N)` + righe + "Visualizza tutto" + quick create). Related per oggetto (Osservato): Account `Referenti/Opportunità/Casi`; Contact `Opportunità/Casi/File`; Opportunità `Ruoli dei referenti/File/Fatture`; Lead `File`. Vedi `related-list-system.md`.

## Comportamenti (Osservato/Testato)
- **Inline edit** (`inline-edit-system.md`): ✎ per field, barra batch Annulla/Salva.
- **Navigazione a record correlati:** i lookup (es. Nome account) sono link cliccabili → record correlato.
- **Modifica** (azione header) → modale edit completa (`form-and-modal-system.md`).
- **Eliminazione** (Testato): ▾/azione `Elimina` → **dialog di conferma** ("Eliminare questo account?" / titolo "Elimina account", `Annulla e chiudi`/`Elimina`) → redirect alla list view. Cascata su record correlati **Da verificare**.
- **Stato senza attività / senza relazioni:** empty state curati nelle card.
- **Loading:** skeleton durante il caricamento; **Da verificare** dettagli.
- **Banner AI (Agentforce)** e **card Slack**: **Non replicare**.

## Proposta Laravel (Proposto per Laravel)
- Route `/dashboard/crm/{obj}/{id}` → controller `show`. Componenti: `x-crm.page-header`, `x-crm.path`, `x-crm.detail-section`, `x-crm.activity-panel`, `x-crm.related-list` (vedi `component-catalog.md`).
- **Layout config** per oggetto: array di sezioni→campi + related lists → riusabile e configurabile (mirror "page layout" dell'org di riferimento senza l'editor).
- Eager-load relazioni (owner, account, related counts) per evitare N+1.
- Policy `view` sul record; azioni (Modifica/Elimina/Converti/…) gate-ate.
- **Eliminazione:** `x-crm.confirm-dialog` + `DELETE` con policy `delete` (admin) + soft delete + gestione cascata/relazioni (impedire orfani o cascatare con conferma).

## Priorità
- **V1:** header+azioni, detail sections con inline edit, activity panel, related lists principali, modifica/elimina con conferma, path dove serve.
- **V2:** layout configurabile per oggetto/profilo, related list ricche (paginazione, azioni inline), skeleton avanzati.
- **V3:** page layout editor, componenti custom.

## Open questions → `open-questions-and-assumptions.md`
Cascata eliminazione; skeleton/loading; stato con molte relazioni; navigazione back; responsive record page.
