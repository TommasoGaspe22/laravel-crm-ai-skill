# Come usare la skill `laravel-crm-blueprint`

Guida pratica per usare questa skill. È un **reverse-engineering completo di un CRM enterprise di riferimento** trasformato in una **specifica implementabile per un CRM Laravel** (pensata per un progetto Laravel generico, riusabile in qualunque app). Nessun asset/branding/dato proprietario dell'org di riferimento: solo pattern, UX e logica.

---

## 1. Cos'è (e cosa NON è)
- **È:** una specifica di prodotto + UX + database + business logic + componenti + test + roadmap, così dettagliata che un agente può costruire la V1 del CRM senza reinventare le decisioni chiave.
- **NON è:** codice del CRM già implementato. L'implementazione va fatta seguendo la skill (non è ancora stata scritta nella repo applicativa).

## 2. Attivazione in Claude Code
- **Automatica:** lavorando dentro il tuo progetto Laravel, la skill si attiva su richieste tipo *"costruisci il CRM leads"*, *"vista lead stile enterprise"*, *"pagina dettaglio opportunità"*, *"pipeline/kanban"*, *"conversione lead"*, ecc. (vedi il campo `description` in `SKILL.md`).
- **Manuale:** apri `SKILL.md` (entry point) → segui i rimandi ai file in `references/`.

## 3. Da dove partire (ordine di lettura)
1. **`SKILL.md`** — overview, principio guida, reuse map, guardrail, build order.
2. **`references/README.md`** — indice completo dei file + metodo + legenda evidenza.
3. **`references/definition-of-done.md`** — stato reale/percentuali oneste + cosa resta "Da verificare".
4. Poi il file del dominio che ti serve (vedi §5).

## 4. Legenda evidenza (in ogni file)
Ogni affermazione importante è etichettata: **Osservato** (visto in UI) · **Testato dinamicamente** (creando/modificando record demo) · **Dedotto** · **Proposto per Laravel** · **Da verificare** · **Non replicare** · **Rischio implementativo**. → Fidati di "Testato/Osservato"; tratta "Proposto per Laravel" come raccomandazione; verifica i "Da verificare" prima di darli per assodati.

## 5. Percorsi d'uso

### A) Capire il CRM di riferimento da replicare
`source-crm-reference.md` · `screen-anatomy.md` · `objects-catalog.md` · i vari `*-system.md` (list-view, filtri, record-page, kanban, attività, conversione…).

### B) Implementare la V1 in Laravel (percorso principale)
1. `laravel-implementation-blueprint.md` — architettura, strati, action/policy/observer, naming, rischi.
2. `data-model-and-migrations.md` — tabelle `crm_*`, campi, indici, relazioni (additive, non distruttive).
3. `component-catalog.md` + `ux-design-system.md` + `screen-anatomy.md` — componenti Blade/Alpine in stile enterprise + token.
4. `modules.md` — modulo Lead concreto (rotte/controller/viste/quick-action) da cui derivano gli altri oggetti.
5. `permissions-and-roles.md` + `testing-strategy.md` — policy + test.
6. `v1-v2-v3-roadmap.md` — cosa fare in V1 (NON tutto!) e cosa rimandare.
7. **Build order** (in `SKILL.md`): Fase A (Lead layer: estrai `CommercialPipelineActionService` + migration additiva + `/crm/leads`) → Fase B (Account/Contact/Opportunità + conversione + attività).

### C) Costruire un singolo modulo/schermata
Apri il file specifico: `list-view-system.md`, `record-page-system.md` (+ `lead/account/contact/opportunity-record-page.md`), `filter-builder-system.md`, `kanban-system.md`, `stage-transition-system.md`, `lead-conversion-system.md`, `activity-and-timeline-system.md`, `inline-edit-system.md`, `form-and-modal-system.md`, `validation-and-error-system.md`, `field-catalog.md`.

### D) Completare i test residui sull'org demo (opzionale)
Vedi `definition-of-done.md` §residuo + `open-questions-and-assumptions.md`. Tooling in `scratchpad/sfdriver/` (Playwright): riuso sessione `sf-state.json`; se scade, l'OTP arriva via email (leggibile col connettore Gmail, account `tommasogaspe2`). Ricrea un mini-dataset con prefisso **`CRM-SKILL-`** e dati fittizi (`@crm-skill.test`), poi **pulisci** (procedura in `demo-data-register.md`).

## 6. Regole d'oro (dai guardrail)
- **NON** clonare la piattaforma enterprise di riferimento (Flow, report builder, AI, sharing complesso) → `v1-v2-v3-roadmap.md` §Non replicare.
- **Riusa** `commercial_pipeline_entries` + il motore next-action esistente; aggiungi tabelle `crm_*` **additive e reversibili**; non rompere pipeline/oggi/import-export esistenti.
- **Server-side first** (filtri/sort/paginazione in SQL, no N+1); Alpine solo per interazioni.
- **Policy su ogni rotta/azione**; validazione lato server; **nessun dato reale del CRM di riferimento** in repo/commit/seed.
- Stack fisso: Laravel 13 · Blade · Alpine · Tailwind · SQLite. Nuove dipendenze solo in V2 (SortableJS per Kanban, FullCalendar per eventi).

## 7. Mappa rapida file (per bisogno)
| Ti serve… | Apri |
|---|---|
| Indice + metodo | `references/README.md` |
| Stato/avanzamento | `references/definition-of-done.md` |
| Riferimento reale | `references/source-crm-reference.md`, `screen-anatomy.md` |
| Oggetti + relazioni | `references/objects-catalog.md`, `data-model-and-migrations.md` |
| Componenti UI | `references/component-catalog.md`, `ux-design-system.md` |
| Liste/filtri/kanban | `references/list-view-system.md`, `filter-builder-system.md`, `kanban-system.md` |
| Record/dettaglio | `references/record-page-system.md` (+ per-oggetto), `inline-edit-system.md` |
| Flussi | `references/lead-conversion-system.md`, `stage-transition-system.md`, `activity-and-timeline-system.md` |
| Implementare | `references/laravel-implementation-blueprint.md`, `modules.md` |
| Permessi/test | `references/permissions-and-roles.md`, `testing-strategy.md` |
| Priorità | `references/v1-v2-v3-roadmap.md` |

## 8. Prossimo passo consigliato
Chiedi a Claude, dentro il tuo progetto Laravel: *"implementa la Fase A del CRM seguendo la skill laravel-crm-blueprint"* → partirà da estrazione service + migration additiva + `/crm/leads`, in TDD, senza toccare le funzioni esistenti.
