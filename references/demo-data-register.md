# Demo data register (`CRM-SKILL-`)

**Scope:** registro di TUTTI i record demo creati nell'org di riferimento per la ricerca. Aggiornare a ogni creazione/modifica/eliminazione. Serve per test ripetibili e per il **cleanup finale**.

## Regole (obbligatorie)
- Ogni record creato ha prefisso **`CRM-SKILL-`** nel nome/oggetto principale.
- Email solo fittizie: `@example.test`, `@crm-skill.test`, `@crm-demo.test`.
- Telefoni/URL/file fittizi. Nessun dato personale reale.
- Non toccare record non creati da me. Non inviare email/inviti reali.
- Cleanup finale: eliminare tutti i `CRM-SKILL-` (procedura in fondo).

## Convenzione naming
`CRM-SKILL-{Oggetto} {Variante} {NNN}` — es. `CRM-SKILL-Lead Nuovo 001`, `CRM-SKILL-Azienda Manufacturing 001`, `CRM-SKILL-Opportunità Automazione Fatture`.

## Registro record

| ID | Tipo | Nome | Obiettivo test | Stato | Relazioni | Flussi testati | Mantieni? | Note |
|---|---|---|---|---|---|---|---|---|
| 00Qbl000005Rh9GEAS | Lead | CRM-SKILL-Lead Nuovo 001 | list view, record page, conversione | Nuovo | Società CRM-SKILL-Azienda Manufacturing 001 | creazione (Testato) | sì | Nome=Mario, email/tel fittizi |
| 00Qbl000005RjCfEAK | Lead | CRM-SKILL-Lead Contattato 002 | stato Contattato, inline edit | Contattato | — | creazione + set stato (Testato) | sì | Nome=Luca |
| 00Qbl000005RjHVEA0 | Lead | CRM-SKILL-Lead NonQualif 004 | stato Non qualificato | Non qualificato | — | creazione + set stato (Testato) | sì | Nome=Sara |
| 00Qbl000005RjJ7EAK | Lead | CRM-SKILL-Lead Minimo 005 | dati minimi (solo required) | Nuovo (default) | — | creazione dati minimi (Testato) | sì | solo Cognome+Società |
| *(fallito)* | Lead | CRM-SKILL-Lead Nurturing 003 | stato Nurturing | — | — | picklist flaky → da ricreare o set via inline edit | — | ritentare |

**Osservazione (Testato dinamicamente):** creazione via URL diretto di creazione lead dell'org di riferimento affidabile; picklist `Stato lead` intermittente (re-render lato org); required bloccanti = Cognome+Società (msg "Compila questo campo."). ID Lead prefix `00Q`.

### Account creati (Testato dinamicamente)
| ID | Nome | Tipo | Note |
|---|---|---|---|
| 001bl000008DwzKAAS | CRM-SKILL-Azienda Manufacturing 001 | Cliente | tel + sito web fittizi; usato per opportunità |
| 001bl000008DvaHAAS | CRM-SKILL-Azienda Retail 002 | Potenziale cliente | usato per opportunità |
| 001bl000008DxQlAAK | CRM-SKILL-Azienda SenzaRelazioni 003 | — | caso "senza referenti/opportunità" |

ID Account prefix `001`. Campi New Account (Osservato): Nome account* (unico required), Tipo(picklist), Telefono, Sito Web, Società controllante(lookup), indirizzi.

### ⚠️ Blocco noto: creazione Opportunità via automazione
**Rischio implementativo / nota tooling:** la modale New Opportunity ha un **lookup Account** che, pilotato via Playwright, destabilizza il form (dopo l'interazione col lookup gli altri campi diventano non interattivi). Tentativi multipli (click opzione, keyboard ArrowDown+Enter, ricerca corta/lunga, creazione da record Account) falliti in modo intermittente.
**Workaround adottato:** creare le Opportunità tramite il **flusso di conversione Lead** (Lead→Account+Referente+Opportunità), che è anche un flusso chiave da testare. In alternativa, creazione manuale.
**Osservato comunque:** campi, sezioni (About/Status), fasi (`Qualify→…→Closed Lost`), Categoria previsione auto da Fase — sufficiente per `field-catalog`/`opportunity-record-page`.

### Conversione eseguita (Testato dinamicamente, 2026-07-08)
`CRM-SKILL-Lead Nuovo 001` (00Qbl000005Rh9GEAS) **convertito** → creati:
- Account `CRM-SKILL-Azienda Manufacturing 001` (**duplicato** — l'org di riferimento non ha forzato "Scegli"; da recuperare ID)
- Referente `Mario CRM-SKILL-Lead Nuovo 001` (email/telefono trasferiti)
- Opportunità `CRM-SKILL-Azienda Manufacturing 001` (Data chiusura auto 30/09/2026, Fase iniziale)
Lead → stato **Convertito** (read-only). ID esatti dei 3 record: **Da recuperare** (query o navigazione dalle card).

### Sessione 2 (2026-07-08) — test dinamici aggiuntivi
- **Lead 003 Nurturing esiste** (il save era andato a buon fine nonostante il report di errore): visibile in AllOpenLeads.
- **Conversione Lead 002** (`00Qbl000005RjCfEAK` Contattato) eseguita → 2ª Opportunità creata.
- **Opportunità presenti (2):** `CRM-SKILL-Azienda Manufacturing 001-` (ID **006bl000005LLplAAG**, ora **Propose**) + `CRM-SKILL-Azienda Retail 002-` (Qualify). Entrambe close 30/09/2026, Ammontare vuoto.
- **Stage transition testata** (Testato): Manufacturing opp Qualify→Propose via Path ("Contrassegna come Fase corrente").
- **Kanban testato** (Testato): board "Tutte le opportunità" con colonne per fase, conteggi + somma €, card con drag handle.
- **Filtri pannello** (Osservato): "Filtra per titolare", "Aggiungi filtro", "Rimuovi tutto" — solo su viste reali, NON su "Recenti".

## Stato creazione dataset (aggiornato)
- **Lead: 5** (00Q… Rh9G convertito / RjCf convertito / RjHV / RjJ7 / +Nurturing003) · **Account: 3 + 2 (da conversioni, dup)** · **Referente: 2** (da conversioni) · **Opportunità: 2** (da conversioni; stati Propose+Qualify) · Attività: 0.
- **Cleanup nota:** include gli Account duplicati + Referenti + Opportunità generati dalle 2 conversioni. Query cleanup per nome contenente `CRM-SKILL-` su ogni oggetto.
- Prossimo: creare **Attività** (Task/Chiamata/Evento/Nota) su un record + timeline; testare **inline edit** e **azioni riga/bulk** + **eliminazione**; valorizzare Ammontare opp (per somme Kanban); testare drag&drop Kanban + chiusura Won/Lost.

## Dataset pianificato (da creare)

**Lead** (stati + varianti): Nuovo · In lavorazione · Contattato · Qualificato · Non qualificato · Non raggiungibile · Convertito; follow-up futuro/oggi/scaduto; priorità alta/bassa; con/senza owner; con/senza attività; con note/file/email/telefono; dati minimi/completi; testo lungo; potenziale duplicato.

**Account**: senza referenti · 1 referente · N referenti · senza opp · opp aperte/vinte/perse · attività future/storiche · note/file · dati minimi/completi.

**Referenti**: CEO · CFO · CIO · Operations · Amministrazione · IT; con/senza attività/email/telefono/opportunità; minimi/completi.

**Opportunità**: una per fase reale (`Qualify → Meet & Present → Propose → Negotiate → Closed Won → Closed Lost`); importi diversi/mancanti; probabilità diverse; closing date passata/futura; next step; motivo perdita; con/senza attività; account+referente collegati; minimi/completi.

**Attività**: Task oggi/scaduto/futuro/completato; chiamata pianificata/completata; email registrata; evento futuro/passato; nota; + eventi di sistema (cambio owner/stato/priorità/fase, conversione, win/loss, file allegato).

## Stato creazione dataset
- Lead: 0/28 · Account: 0/13 · Referenti: 0/14 · Opportunità: 0/20 · Attività: 0/21. **Totale: 0.**

## ✅ CLEANUP ESEGUITO (2026-07-08)
Tutti i record `CRM-SKILL-` **eliminati** (verificato: liste vuote + record convertiti "Impossibile caricare"): 4 Lead diretti + 2 Lead convertiti + 4 Account + 2 Referenti + Opportunità (cascata da eliminazione Account). Il lead pre-esistente non-CRM-SKILL ("Tommaso Gasperoni", 00Qbl000005RNI3EAO) **preservato** (non creato da me). Org demo pulito. **Osservato (cleanup):** eliminare un Account **cascata** su Opportunità+Referenti collegati → conferma modello relazionale + rischio implementativo cascata in Laravel.

## Procedura di cleanup finale (per usi futuri)
1. In ogni list view (Lead, Account, Referenti, Opportunità, Attività, Casi): filtrare/ordinare per nome contenente `CRM-SKILL-`.
2. Selezionare tutti → azione bulk **Elimina** (o riga per riga se bulk non disponibile).
3. Verificare che le eliminazioni a cascata (opportunità/attività collegate) non lascino orfani; eliminare eventuali residui.
4. Svuotare il Cestino (Recycle Bin) se accessibile, per rimuovere definitivamente.
5. Confermare in questo file: data cleanup, conteggio eliminati, residui.
**Rischio implementativo:** l'eliminazione bulk e la cascata vanno verificate; annotare il comportamento reale osservato durante il cleanup (è anche un test valido per `bulk-actions-system.md`).
