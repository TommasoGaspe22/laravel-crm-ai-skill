# Open questions & assumptions

Traccia tutto ciò che è **incerto**, **non ancora testato**, o **assunto**. Ogni voce ha: stato, perché, e come risolverla (test o decisione). Aggiornare a ogni ciclo. In caso di interruzione tecnica, questo file + `definition-of-done.md` permettono di riprendere senza rifare analisi.

## Assunzioni correnti (da confermare con test dinamico)
| # | Assunzione | Base | Come confermare |
|---|---|---|---|
| AS1 | Le fasi Opportunità dell'org sono `Qualify→Meet & Present→Propose→Negotiate→Closed Won→Closed Lost` | Osservato (picklist New Opp) | Creare 1 opp per fase e verificare Path/Kanban |
| AS2 | La scala stati Lead è `Nuovo→Contattato→Nurturing→Non qualificato→Convertito` | Osservato (Path record) | Creare lead e cambiare stato lungo il Path |
| AS3 | La conversione crea Account+Referente+Opportunità in un'unica transazione | Osservato (modale + helper text) | Convertire un lead demo e verificare i 3 record + timeline |
| AS4 | Le related list per Account sono Referenti/Opportunità/Casi | Osservato | Popolare e verificare conteggi/azioni |
| AS5 | Le sezioni record Lead sono About/Get in Touch/Segment/History | Osservato (layout) | Verificare campi per sezione con inline edit |

## Domande aperte (non ancora osservate/testate)
| # | Domanda | Area | Priorità |
|---|---|---|---|
| Q1 | Come si comporta il **filter builder**: operatori per tipo, AND/OR, gruppi, date relative? | filter-builder | alta |
| Q2 | Il **Kanban** supporta drag&drop reale + conferme al cambio fase? Totali per colonna? | kanban | alta |
| Q3 | Comportamento **inline edit**: batch save bar, validazioni, campi non editabili? | inline-edit | alta |
| Q4 | **Validazioni** required + messaggi errore su ogni modale (Lead/Account/Contact/Opp)? | validation | alta |
| Q5 | **Unsaved changes**: warning alla chiusura modale con modifiche? | unsaved-changes | media |
| Q6 | **Bulk actions** disponibili per riga/selezione multipla (cambia owner, elimina, ecc.)? | bulk-actions | alta |
| Q7 | **Saved views**: creazione/clona/rinomina/condivisione/filtri — flusso completo? | saved-views | media |
| Q8 | **Note & File**: composer, allegati, related list, dove vivono? | activity/files | media |
| Q9 | **Casi (Cases)**: oggetto disponibile? campi? relazione Account/Contact? | cases | bassa |
| Q10 | **Split/Doppia visualizzazione**: layout e comportamento? | split-view | bassa |
| Q11 | Campi Lead completi + picklist (Valutazione, Fonte lead, Settore, N. dipendenti)? | field-catalog | alta |
| Q12 | Categoria di previsione (forecast) valori? Motivo perdita esiste su Closed Lost? | opportunity | media |
| Q13 | Comportamento **paginazione** con molti record (>50)? | pagination | media |

## Elementi potenzialmente NON verificabili nella demo (⛔ — motivare + procedura futura)
| # | Elemento | Perché | Procedura di verifica futura |
|---|---|---|---|
| NV1 | Permessi/sharing multi-utente reali | La demo ha 1 solo utente; non posso creare utenti/ruoli | Documentare da conoscenza pregressa dell'org di riferimento + testare in un'org multi-utente o via ruoli Laravel proposti |
| NV2 | Automazioni/Flow/validation rule server-side | Vietato toccarle e non sempre visibili | Dedurre dai comportamenti UI; segnare come "Da verificare" |
| NV3 | Forecast/Quote/Products se non abilitati | Feature non attive nella trial | Segnare "Da verificare"; proporre modello Laravel opzionale (V3) |
| NV4 | Performance con migliaia di record | Non popolabile realisticamente | Definire target + test di performance Laravel in `testing-strategy.md` |

## Note di ripresa (per continuità tra turni)
- Sessione org di riferimento: stato salvato in `scratchpad/sess/session-state.json` (riutilizzabile finché valido; a scadenza serve nuovo OTP dall'utente).
- Toolkit di automazione: script in `scratchpad/crmdriver/` (Playwright, `channel: chrome`).
- **Prossimo punto non verificato:** creazione primo dataset Lead + test New-Lead modal (vedi `definition-of-done.md` → Prossimo ciclo).
