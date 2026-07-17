# Sales-flow objects catalog (captured from the live org)

The full reference sales flow, replicated for your own app. Each object below was inspected **read-only** in the org (Italian locale, trial with default + light customization). Columns, fields, picklists, and actions marked "(reale)" were captured from the org; the rest is standard for this kind of reference org. `*` = required field.

## The flow (how objects connect)

```
Lead ──(Converti)──▶ Account ──has many──▶ Referente (Contact)
                        │
                        └──has many──▶ Opportunità ──line items──▶ Prodotto × Listino (prezzo)
Attività (Task/Evento) ──correlate a──▶ Lead | Account | Referente | Opportunità
```

- **Account** is the hub (an azienda). Contacts and Opportunities hang off it.
- **Lead** is top-of-funnel; "Converti" turns one Lead into Account + Contact + Opportunity.
- **Opportunità** carries the money + stage pipeline.
- **Attività** log the interactions against any object (replaces the current *derived* timeline with real events).

## Coexistence with the existing CRM (important)

Your app already has the flat `commercial_pipeline_entries` (operational lead-working list) — **keep it**. Layer the relational objects as **new tables** (additive, never touching pipeline). Recommended split:
- `commercial_pipeline_entries` + the `/crm/leads` list view = top-of-funnel lead operations (already specced in modules.md).
- New relational objects (`crm_accounts`, `crm_contacts`, `crm_opportunities`, …) = the qualified/downstream sales flow.
- **Conversion** bridges them: a `convertLead()` action creates Account + Contact + Opportunity from a pipeline entry and marks it converted (mirrors the reference "Converti").

Every object follows the **same module pattern** already defined for Lead in `modules.md`: reference-style list view (saved views, filters, sort, bulk/row actions) + detail page (highlights, path/stage, fields, related lists, timeline) + minimal-click quick actions. This file defines the per-object specifics; `data-model-full.md` defines the schema.

---

## 1. Account — Aziende

- **List columns (reale):** Nome account · Telefono · Titolare account.
- **Record fields (reale):** Nome account* · Sito Web · Tipo · Descrizione · Società controllante (parent account, self-FK) · Titolare · Telefono · Indirizzo fatturazione · Indirizzo spedizioni.
- **Header actions (reale):** Nuovo referente · Nuova opportunità · Modifica.
- **Related lists:** Referenti, Opportunità, Attività, File.
- **Target app:** table `crm_accounts`. Reuse `company`/`business_area`/`address` semantics from the pipeline for backfill. Saved views: Tutti · Miei account · Clienti (Tipo=Cliente) · Prospect.

## 2. Referente — Contact

- **List columns (reale):** Nome e cognome · Nome account · Telefono · Email · Titolare referente.
- **Record fields (reale):** Nome e cognome* · Nome account (FK→Account) · Qualifica (ruolo/title) · Fa capo a (reports-to, self-FK) · Descrizione · Titolare · Telefono · Email · Indirizzo postale.
- **Header actions (reale):** Nuova opportunità · Modifica.
- **Target app:** table `crm_contacts`, `belongsTo account`, self-relation `reports_to`. Backfill from pipeline `lead_name/role/email/phone`.

## 3. Opportunità — Opportunity (pipeline del valore)

- **List columns (reale):** Nome opportunità · Nome account · **Fase** · Data chiusura · Titolare opportunità.
- **Record/New fields (reale):** Nome opportunità* · Nome account* (FK) · Data chiusura* · Ammontare (amount) · Descrizione · Titolare · **Fase\*** · Probabilità (%) · Categoria di previsione (forecast category) · Fase successiva (next step).
- **Fasi / Stage picklist (reale, in quest'org):**
  `Qualify → Meet & Present → Propose → Negotiate → Closed Won → Closed Lost` (più `--Nessuno--`).
  **Adattamento italiano, stesso ordine:** `Qualifica → Presentazione → Proposta → Negoziazione → Vinta → Persa`. Store a stable key; render the Italian label. `Vinta`=won, `Persa`=lost are the closed states.
- **Target app:** table `crm_opportunities` with `stage` (enum), `amount`, `close_date`, `probability`, `next_step`, `account_id`, `contact_id` (primary), `owner_id`. The **status path** on the detail page uses these stages (this is the reference "Path"). Saved views: Aperte · Mie · In chiusura questo mese · Vinte · Perse · Per fase.

## 4. Prodotti / Listini — Product2 / Pricebook2

- **Product list columns (reale):** Nome prodotto · Classe di prodotto · Codice prodotto · Descrizione prodotto · Famiglia di prodotti.
- **Pricebook list columns (reale):** Nome listino prezzi · Descrizione · Inizio validità · Fine validità · Data ultima modifica · Attivo.
- **Relationship:** a *PricebookEntry* links Product + Pricebook + prezzo; an *OpportunityLineItem* links Opportunità + PricebookEntry + quantità.
- **Target app:** `crm_products`, `crm_price_books`, `crm_price_book_entries`, `crm_opportunity_items`. **Flag: likely more than most apps need at first** — treat as a later phase; V1 of the sales flow can let an Opportunity carry a free-text/product-less `amount`. Build the catalog only when the team sells a real SKU list.

## 5. Attività / Calendario — Task + Event

- **Task fields (standard):** Oggetto · Scadenza (due) · Stato · Priorità · Assegnato a (owner) · Nome (contact) · Correlato a (account/opp) · Commenti.
- **Event fields (standard):** Oggetto · Inizio · Fine · Luogo · Invitati · Correlato a.
- **Target app:** one polymorphic table `crm_activities` (`type` = task|call|email|event, `subject`, `due_at`/`start_at`/`end_at`, `status`, `owner_id`, `subject_type`+`subject_id` morph to lead/account/contact/opportunity, `notes`). **This replaces the *derived* timeline** with real logged activities — the upgrade the architecture doc deferred to V2. Calendar view: start with a list/agenda; add FullCalendar only when events are real.

---

## Lead (recap) — the entry point

- **List columns (reale):** Nome e cognome · Qualifica · Società · Telefono · Email · Stato lead · Titolare.
- **Record fields (reale):** Stato lead · Nome e cognome · Società · Qualifica · Sito Web · Descrizione · Titolare · Valutazione (rating) · Telefono · Email · Indirizzo · N. di dipendenti · Reddito annuale · Fonte del lead · Settore.
- **Status ladder (reale):** `Nuovo → Contattato → Nurturing → Non qualificato → Convertito`. Align the app's `crm_lead_statuses` in data-model.md to this exact ladder.
- **Header actions (reale):** **Converti** · Cambia titolare · Modifica.
- **Target app:** the `commercial_pipeline_entries` + `/crm/leads` module (see modules.md). Add the **Converti** quick action → creates Account+Contact+Opportunità.

---

## Captured page-layout details (reale — for the detail pages)

Section cards (left column) and related lists (right column) as configured in the org. Rebuild these exact groupings (`screen-anatomy.md` B).

| Object | Detail sections (left) | Related lists (right) | List display modes |
|---|---|---|---|
| **Lead** | About · Get in Touch · Segment · History | File | Tabella · Kanban · Doppia visualizzazione |
| **Account** | About · Get in Touch · History | Referenti · Opportunità · Casi | idem |
| **Referente** | About · Get in Touch · History | Opportunità · Casi · File | idem |
| **Opportunità** | About · Status | (Prodotti/line items · Contatti ruolo · Attività) | Tabella · **Kanban per Fase** · Doppia |

**Create/Edit modal groupings (reale):** Opportunità → **About** (Nome, Account, Data chiusura, Ammontare, Descrizione, Titolare) + **Status** (Fase, Probabilità %, Categoria di previsione, Fase successiva), footer `Annulla · Salva e Nuovo · Salva`.

**List-view controls menu (reale):** `Nuovo · Clona · Rinomina · Impostazioni di condivisione · Seleziona i campi da visualizzare · Elimina · Reimposta larghezze colonna · Tabella · Kanban · Doppia visualizzazione`.

**Convert modal (reale):** three Crea/Scegli sections (Account · Referente · Opportunità) + "Non creare un'opportunità" checkbox + `*Titolare record` + `*Stato convertito` (default "Qualificato"). See `screen-anatomy.md` D + `data-model-full.md` convert flow.
