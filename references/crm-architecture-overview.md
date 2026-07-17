# CRM architecture overview (reference CRM → Laravel)

**Obiettivo:** dare la visione d'insieme dell'architettura CRM da replicare, come mappa per tutti i file di dettaglio. **Scope:** concetti e strati, non implementazione riga-per-riga (quella è in `laravel-implementation-blueprint.md`).

## 1. Modello concettuale (dominio CRM B2B)

```
                 ┌─────────┐   converti    ┌──────────┐ 1  ┌────────────┐
   inbound ─────▶│  Lead   │──────────────▶│  Account │───▶│  Referente │
                 └─────────┘               └────┬─────┘ *  └─────┬──────┘
                      │                         │ 1              │ *
                      │                         ▼ *              │
                      │                   ┌───────────┐          │
                      └──────────────────▶│Opportunità│◀─────────┘  (referente primario)
                                          └─────┬─────┘
                                                │ * (fasi/pipeline)
     ┌────────────────────────────────────────┴──────────────┐
     ▼ (polimorfico "correlato a")                            ▼
┌──────────┐   ┌────────┐   ┌───────┐                   ┌──────────┐
│ Attività │   │  Note  │   │ File  │  ── su Lead/Account/Referente/Opportunità
└──────────┘   └────────┘   └───────┘
```

- **Lead** = top-of-funnel non qualificato. **Osservato**: stati `Nuovo→Contattato→Nurturing→Non qualificato→Convertito`.
- **Account** = azienda (hub). **Referente** = persona in un'azienda. **Opportunità** = trattativa a valore con **fasi** (`Qualify→…→Closed Won/Lost` — Osservato).
- **Attività/Note/File** = correlate polimorficamente a qualunque oggetto → alimentano **timeline** e **related list**.
- **Conversione** Lead → crea Account+Referente+Opportunità in transazione (**Osservato** modale).

## 2. Strati dell'applicazione Laravel (Proposto per Laravel)

| Strato | Responsabilità | Componenti chiave |
|---|---|---|
| **Presentazione** | UI enterprise-like | Blade components (`x-crm.*`), Alpine per interazioni, Tailwind + `crm.css` token |
| **HTTP** | routing, request, autorizzazione | Controllers sottili, Form Request (validazione), Policy |
| **Applicazione** | casi d'uso / azioni | Action classes (es. `ConvertLead`, `ChangeStage`, `LogActivity`), Service dove serve |
| **Dominio** | entità + regole | Eloquent models (`CrmAccount`, `CrmContact`, `CrmOpportunity`, `CrmActivity`, …), enum/config (stati, fasi) |
| **Persistenza** | schema + query | Migrations additive, indici, query builder filtri, soft delete |
| **Eventi/Audit** | timeline + audit | Model events/Observers → `crm_activities`/`audit_events` (timeline generator) |
| **Infrastruttura** | file, notifiche, job | Storage (file), Notifications (reminder/follow-up), Jobs (async) |

**Principio guida:** server-rendered + progressive enhancement (Alpine). Nessun SPA/React. Query server-side (filtri/sort/paginazione) con attenzione a N+1 e indici. Vedi `laravel-implementation-blueprint.md`.

## 3. Coesistenza con il codice esistente (il tuo progetto Laravel)
- Mantenere `commercial_pipeline_entries` + `/crm/leads` (V1 Lead operativo, già speccato in `modules.md`) come **strato Lead**.
- Aggiungere gli oggetti relazionali come **tabelle nuove `crm_*`** (additive, non distruttive): `crm_accounts/contacts/opportunities/activities/…`.
- Bridge = azione **Converti**. Dettagli in `data-model-full.md` + `lead-conversion-system.md`.
- **Rischio implementativo:** duplicazione temporanea tra pipeline flat e oggetti relazionali; governare con la conversione e viste chiare. Non normalizzare in modo distruttivo.

## 4. Mappa moduli → file di dettaglio
- App shell/navigazione → `app-shell-system.md`
- Liste → `list-view-system.md`, `table-system.md`, `saved-views-system.md`, `column-management-system.md`, `bulk-actions-system.md`, `pagination-and-loading-system.md`, `split-view-system.md`, `filter-builder-system.md`
- Record → `record-page-system.md` + per-oggetto + `inline-edit-system.md`, `related-list-system.md`
- Flussi → `activity-and-timeline-system.md`, `lead-conversion-system.md`, `pipeline-system.md`, `kanban-system.md`, `stage-transition-system.md`
- Form/dati → `form-and-modal-system.md`, `field-catalog.md`, `validation-and-error-system.md`, `unsaved-changes-system.md`, `data-model-and-migrations.md`
- Design → `component-catalog.md`, `screen-anatomy.md`
- Trasversali → `permissions-and-roles.md`, `testing-strategy.md`, `laravel-implementation-blueprint.md`, `v1-v2-v3-roadmap.md`

## 5. Priorità architetturale (sintesi — dettaglio in roadmap)
- **V1:** Lead + Account + Referente + Opportunità (fasi + Path), list view (tabella, sort, ricerca, filtri base, saved views base, azioni riga/bulk essenziali), record page (sezioni, inline edit, related, timeline base), conversione Lead, attività base, policy owner/staff, seed+test.
- **V2:** Kanban drag&drop, filter builder avanzato (AND/OR/gruppi), colonne configurabili+resize, split view, note/file ricchi, forecast base, reminder.
- **V3:** prodotti/listini/line items, team, forecasting avanzato, audit granulare, API pubblica.
- **Non replicare:** Flow/automation builder, report builder, Einstein/AI, sharing rules complesse, multi-currency avanzata.
