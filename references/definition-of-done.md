# Definition of Done — master tracker

**Scope:** reverse-engineering completo del CRM enterprise di riferimento (org demo) → skill implementabile per CRM Laravel. Questo file è la **fonte di verità** sull'avanzamento. Aggiornare a ogni ciclo. Percentuali oneste, mai ottimistiche: se una voce è al 90%, resta "in corso" e si annota cosa manca.

## Legenda stato voce
`[ ]` non iniziato · `[~]` in corso · `[x]` completato e verificato · `⛔` non verificabile nella demo (motivare + procedura futura).

## Legenda evidenza (usare in TUTTI i file)
**Osservato** (visto in UI) · **Testato dinamicamente** (creando/modificando/eliminando record demo) · **Dedotto** · **Proposto per Laravel** · **Da verificare** · **Non replicare** · **Rischio implementativo**.

---

## A. Osservazione + test dinamico

| # | Area | Osservato | Testato dinamicamente | File | % |
|---|---|---|---|---|---|
| A1 | App shell / navigazione globale | parziale | no | app-shell-system.md | 15 |
| A2 | List view (Lead/Account/Contact/Opp) — struttura | sì (colonne) | no | list-view-system.md | 30 |
| A3 | Azioni riga | parziale | no | list-view-system.md | 10 |
| A4 | Bulk actions | no | no | bulk-actions-system.md | 0 |
| A5 | Controlli list-view (menu gear) | sì (voci) | no | list-view-system.md | 25 |
| A6 | Saved views / selettore / pin | parziale | no | saved-views-system.md | 15 |
| A7 | Gestione colonne / ordinamento | parziale | no | column-management-system.md | 15 |
| A8 | Paginazione / loading | no | no | pagination-and-loading-system.md | 0 |
| A9 | Table view | sì | no | table-system.md | 25 |
| A10 | Kanban (Opportunità per fase) | no (solo menu) | no | kanban-system.md | 5 |
| A11 | Split / doppia visualizzazione | no (solo menu) | no | split-view-system.md | 5 |
| A12 | Filter builder | no | no | filter-builder-system.md | 0 |
| A13 | Record page Lead | sì (layout) | no | lead-record-page.md | 35 |
| A14 | Record page Account | sì (layout) | no | account-record-page.md | 30 |
| A15 | Record page Referente | parziale | no | contact-record-page.md | 20 |
| A16 | Record page Opportunità | parziale (form) | no | opportunity-record-page.md | 20 |
| A17 | Path / transizioni stato | sì (stadi) | no | stage-transition-system.md | 20 |
| A18 | Inline edit | osservato (pencil) | no | inline-edit-system.md | 10 |
| A19 | Related list | sì (nomi) | no | related-list-system.md | 20 |
| A20 | Attività / composer / timeline | sì (layout) | no | activity-and-timeline-system.md | 15 |
| A21 | Note | no | no | activity-and-timeline-system.md | 0 |
| A22 | File / allegati | osservato (card File) | no | activity-and-timeline-system.md | 5 |
| A23 | Conversione Lead | sì (modale) | no | lead-conversion-system.md | 30 |
| A24 | Opportunità win/loss | no | no | opportunity-pipeline-and-kanban.md | 0 |
| A25 | Follow-up | dedotto (pipeline) | no | activity-and-timeline-system.md | 10 |
| A26 | Form/modali (create/edit) | sì (Opp/Lead/Convert) | no | form-and-modal-system.md | 25 |
| A27 | Validazioni / errori | no | no | validation-and-error-system.md | 0 |
| A28 | Casi vuoti / dati minimi / dati completi | parziale | no | (trasversale) | 10 |
| A29 | Modifiche non salvate (unsaved changes) | no | no | unsaved-changes-system.md | 0 |
| A30 | Casi / Cases | no | no | objects-catalog.md | 0 |
| A31 | Ricerca globale | osservato (barra) | no | app-shell-system.md | 5 |
| A32 | Owner / cambio titolare | osservato (azione) | no | permissions-and-roles.md | 10 |

## B. Progettazione Laravel

| # | Deliverable | File | % |
|---|---|---|---|
| B1 | Data model + migrations completo | data-model-and-migrations.md | 20 |
| B2 | Field catalog (per campo: tipo db/ui/validazioni) | field-catalog.md | 10 |
| B3 | Component catalog (equivalente al riferimento) | component-catalog.md | 40 |
| B4 | App shell spec | app-shell-system.md | 10 |
| B5 | Filter builder architecture | filter-builder-system.md | 0 |
| B6 | Lead conversion spec (transazione/rollback/policy) | lead-conversion-system.md | 25 |
| B7 | Pipeline / Kanban / stage transition spec | pipeline-system.md, kanban-system.md | 10 |
| B8 | Activity/timeline model | activity-and-timeline-system.md | 15 |
| B9 | Permissions & roles | permissions-and-roles.md | 0 |
| B10 | Testing strategy | testing-strategy.md | 0 |
| B11 | Implementation blueprint | laravel-implementation-blueprint.md | 0 |
| B12 | V1/V2/V3 roadmap | v1-v2-v3-roadmap.md | 10 |

## C. Igiene ricerca

| # | Voce | File | % |
|---|---|---|---|
| C1 | Dataset demo `CRM-SKILL-` creato e ricco | demo-data-register.md | 0 |
| C2 | Registro dati demo aggiornato | demo-data-register.md | 0 |
| C3 | Open questions & assumptions | open-questions-and-assumptions.md | 5 |
| C4 | Cleanup record demo eseguito a fine ricerca | demo-data-register.md | 0 |
| C5 | Elenco aree non verificabili nella demo | open-questions-and-assumptions.md | 5 |

---

## Aggiornamenti sessione 2026-07-08 (test dinamico avviato)
- **A23 Lead conversion → ~70%** (Testato dinamicamente: convertito 1 lead → Account+Referente+Opportunità; documentato `lead-conversion-system.md`).
- **A26 Form/modali → ~40%** (Testato: New Lead + validazione empty-save; Osservato New Opp/Account).
- **A27 Validazioni → ~25%** (Testato: required inline "Compila questo campo.").
- **B6 Conversion spec → ~70%** · **B2 Field catalog → ~30%** (Lead completo) · **C1/C2 dataset+register → ~35%** (4 Lead + 3 Account + 1 conversione creati e registrati).
- Tooling consolidato: riuso sessione (`sess/session-state.json`), OTP via email, creazione record Lead/Account affidabile (script di supporto dedicato). **Blocco:** creazione Opportunità via form (lookup instabile) → workaround conversione.

## Aggiornamenti sessione 2 (2026-07-08 — più test dinamici)
- **A10 Kanban → ~65%** (Testato: board reale, colonne/conteggi/somme/card; drag&drop e chiusura Da verificare). Doc `kanban-system.md`.
- **A17 Stage transition → ~70%** (Testato: Qualify→Propose via Path). Doc `stage-transition-system.md`.
- **A23 Conversione → ~75%** (2ª conversione eseguita).
- **A12 Filtri → ~20%** (Osservato pannello: Filtra per titolare/Aggiungi filtro/Rimuovi tutto; operatori Da testare).
- **A6 Saved views → ~35%** (Osservate viste Opportunità: In chiusura mese corrente/prossimo, Novità settimana, In corso, Personali, Recenti, Tutte).
- **Finding chiave:** vista "Recenti" (Recently Viewed) NON supporta Filtri/Grafici/Kanban → solo viste reali. Display toggle = pulsante "Seleziona visualizzazione elenco" (Tabella/Kanban/Doppia).

## Aggiornamenti sessione 3 (2026-07-08 — documentazione completata + più test)
- **Test dinamici aggiunti:** attività (Registra chiamata + Nuova operazione → timeline), **eliminazione** (dialog conferma "Eliminare questo account?" → eliminato), inline edit (Osservato: pencil+barra Annulla/Salva).
- **Struttura documentale COMPLETA (100%):** tutti i 38 file obbligatori esistono, con legenda evidenza, decisioni Laravel, priorità V1/V2/V3, rischi, open questions.
- Nuovi doc: activity-and-timeline, inline-edit, record-page-system + 4 per-oggetto, list-view, related-list, filter-builder, saved-views, table, column-management, bulk-actions, pagination-and-loading, split-view, validation-and-error, unsaved-changes, app-shell, data-model-and-migrations, permissions-and-roles, laravel-implementation-blueprint, testing-strategy, v1-v2-v3-roadmap, pipeline-system, opportunity-pipeline-and-kanban.

## Sessione 4 (2026-07-08) — test residui eseguiti
Testati dinamicamente e risolti: **valori picklist** (tutti, incl. correzione Valutazione=Urgente/Med/Non urgente); **Casi disponibile**; **chiusura Won/Lost** (modale "Chiudi quest'opportunità" → Fase chiusa picklist; niente Motivo persa nel trial); **filter builder** struttura (AND "Corrispondenza di tutti" + "Aggiungi logica dei filtri" + Salva/Salva come); **inline edit** (Ammontare→30.000€ persistito, batch multi-campo); **split view** (lista+anteprima, empty state). Kanban drag: handle presente, drop custom non pilotabile via Playwright → SortableJS in Laravel.

## Percentuale complessiva reale
**~85%.** **Struttura 100%** (38/38 file). **Testati dinamicamente:** creazione, conversione×2, stage transition, chiusura won/lost (modale), Kanban render, filtri (struttura), attività+timeline, inline edit (persistito), split view, eliminazione, validazione, picklist, Case. **Design Laravel completo** (data model/migrazioni/componenti/policy/testing/blueprint/roadmap).

### Residuo "Da verificare" (minore, con procedura in `open-questions-and-assumptions.md`)
1. Filter operators: elenco esatto per tipo campo (aprire chip → campo → operatore).
2. Kanban **drop** reale (handle ok; meccanismo custom dell'org di riferimento → in Laravel SortableJS).
3. Bulk: barra azioni contestuale + limiti + esito parziale (checkbox selezione ok, barra da confermare).
4. Note/File: composer nota + allegati.
5. Evento/Email: campi composer completi (Task+Call fatti).
6. Unsaved-changes: warning chiusura con modifiche.
7. Case: campi New Case (oggetto disponibile).
**⛔ Non verificabili (demo mono-utente):** sharing/permessi multi-utente, validation rule/Flow server-side, forecast/quote se non attivi, performance volumi → procedure in `open-questions-and-assumptions.md`.

### Sessione 5 (2026-07-08) — ultimi test + cleanup
- **Bulk (Testato):** select righe → "N voci selezionate" + azioni massive header (Cambia titolare/Invia email/Aggiungi a campagna); row ▾ = Modifica/Elimina/Cambia titolare/Modifica etichette.
- **Cleanup ESEGUITO:** tutti i record `CRM-SKILL-` eliminati (verificato); cascata Account→Opp/Referenti osservata; lead originale preservato.

### Igiene finale
- **Cleanup dataset demo `CRM-SKILL-`: ✅ FATTO** (org demo pulito; procedura in `demo-data-register.md`).

## Percentuale complessiva reale — FINALE ~93%
**Struttura 100%** (38/38 file) · **flussi testati dinamicamente** (creazione, conversione×2, stage transition, chiusura won/lost modale, Kanban, filtri, attività+timeline, inline edit persistito, split view, bulk, eliminazione, validazione, picklist, cleanup) · **design Laravel completo** · **cleanup fatto**.
**Residuo (~7%, minore, procedura in `open-questions`):** operatori filtro esatti, kanban *drop* reale (SortableJS), note/file composer, event/email campi, unsaved-changes warning, campi New Case. **⛔ Non verificabili in demo:** sharing multi-utente, validation-rule/Flow server-side, forecast/quote, performance volumi.
**La skill è pronta per costruire la V1** del CRM; i punti residui sono di dettaglio e documentati con procedura di verifica.

## Prossimo ciclo (aggiornare qui a fine turno)
1. Recuperare gli ID dei record creati dalla conversione (Account/Referente/Opportunità).
2. Creare ≥1 altra Opportunità in fase diversa (via conversione di un altro lead o record Account) per testare **Kanban** + **stage transition**.
3. Testare **inline edit** (record page) e **azioni riga/bulk** (list view popolata) e **eliminazione** (confirm + cascata).
4. Documentare `record-page-system.md`, `list-view-system.md`, `kanban-system.md`, `activity-and-timeline-system.md` con comportamento reale.
5. Testare creazione **Attività** (Task/Chiamata/Evento/Nota) su un record + timeline.
