# Form & modal system

**Scope:** anatomia e comportamento delle modali create/edit/convert/confirm, con proposta Laravel (componente `x-crm.record-modal` + Form Request). Fonte: **Osservato** (New Lead, New Opportunità, Converti) + **Testato dinamicamente** (validazione empty-save su New Lead).

## Indice
1. Anatomia comune modale · 2. New/Edit record · 3. Validazione & errori · 4. Convert · 5. Confirm/Delete · 6. Unsaved changes · 7. Laravel · 8. Priorità · 9. Open questions

## 1. Anatomia comune (Osservato)
- **Trigger:** pulsante `Nuovo`/`Modifica`/azione riga/azione record.
- **Overlay** scuro modale, contenuto centrato, `X` in alto a destra.
- **Header:** titolo `Crea {Oggetto}` / `Modifica {Oggetto}`. Nota `* = Informazioni richieste` in alto a destra.
- **Body:** **sezioni** con header grigio che raggruppano i campi; scroll interno se lungo; layout a 2 colonne su desktop.
- **Footer:** `Annulla` · (`Salva e Nuovo`) · `Salva` (primario). Sulla conversione: `Annulla · Converti`.
- **Campi:** text, textarea, date (icona calendario), number, currency, **picklist** (combobox ▾ con `role=listbox/option`), **lookup** (ricerca con lente, "N corrispondenze"), checkbox, owner (lookup User).

## 2. New / Edit — specifiche per oggetto

### New Lead (Osservato + Testato dinamicamente)
- Sezioni: **About · Get in Touch · Segment**.
- Campi: vedi `field-catalog.md` (Lead). Compound **Nome e cognome** (Titolo+Nome+Cognome).
- **Required bloccanti (Testato):** `Cognome`, `Società`. `Stato lead` è `*` ma con default → non blocca.
- Picklist: `Stato lead`, `Valutazione`, `Fonte del lead`, `Settore`.

### New Opportunità (Osservato)
- Sezioni: **About** (Nome*, Account* lookup, Data chiusura*, Ammontare, Descrizione, Titolare) · **Status** (Fase*, Probabilità %, Categoria previsione*, Fase successiva).
- Footer: `Annulla · Salva e Nuovo · Salva`.

### New Account / Contact (Da verificare — catturare New modal)
- Account: Nome account* + Tipo(picklist) + Sito Web + Telefono + indirizzi + Società controllante(lookup self).
- Contact: Nome e cognome* + Nome account(lookup) + Qualifica + Email + Telefono + Fa capo a(lookup self).

## 3. Validazione & errori (Testato dinamicamente — New Lead)
- **Empty-save:** i campi required mostrano errore inline **"Compila questo campo."** sotto il campo, bordo rosso, focus sul primo invalido; il salvataggio è bloccato, la modale resta aperta.
- **Osservato/Dedotto:** email valida per formato; date con date-picker; number/currency numerici. Messaggi in italiano.
- **Da verificare:** validazioni cross-field, lunghezze massime, messaggi per formato email/URL, errori server-side (validation rule) → non testabili senza campi/regole custom.

## 4. Convert modal (Osservato — Lead → 3 record)
Titolo **"Converti lead"**. Tre sezioni con radio **Crea / Scegli**: Account · Referente · Opportunità (+ checkbox "Non creare un'opportunità"). Bottom: `*Titolare record`, `*Stato convertito` (default "Qualificato"). Dettaglio in `lead-conversion-system.md`.

## 5. Confirm / Delete (Da verificare)
Atteso: dialog di conferma con messaggio + `Annulla`/`Elimina`. Da testare dinamicamente (eliminando un record demo) → documentare testo, comportamento cascata, toast.

## 6. Unsaved changes (Da verificare)
Atteso: warning alla chiusura modale con modifiche non salvate. Da testare (aprire, modificare, chiudere con X).

## 7. Proposta Laravel
- **Componente** `x-crm.record-modal`: props `title`, `:sections`, `:footer`, slot per campi; Alpine `modal` (focus-trap, Esc, backdrop). Campi via `x-crm.field.*` (`field-catalog.md`/`component-catalog.md`).
- **Validazione:** `FormRequest` per oggetto (es. `StoreLeadRequest`) → regole `required`, `email`, `numeric`, `date`, `in:{picklist}`; messaggi IT ("Compila questo campo." → `:attribute è obbligatorio`). Mostrare errori inline sotto il campo (mirror del riferimento).
- **Azioni:** `Salva` (store/update), `Salva e Nuovo` (redirect a create), `Annulla` (chiudi). Toast su successo (`session('status')`).
- **Unsaved changes:** Alpine `dirty` flag + `beforeunload`/conferma su chiusura.
- **Rischio implementativo:** compound Name → in Laravel campi separati; lookup async → endpoint dedicato con debounce + policy; picklist → config/enum condivisi con validazione.

## 8. Priorità
- **V1:** New/Edit Lead/Account/Contact/Opportunità, validazione required inline, Convert, Confirm/Delete, toast.
- **V2:** Salva e Nuovo, unsaved-changes warning, lookup async ricco, sezioni collassabili.
- **V3:** record types multipli, campi dipendenti/condizionali avanzati.
- **Non replicare:** dynamic forms server-driven complessi, page-layout assignment per profilo.

## 9. Open questions (→ `open-questions-and-assumptions.md`)
Q4 validazioni complete per oggetto · Q5 unsaved-changes · New Account/Contact modal · Confirm/Delete testo+cascata · picklist values (Tipo, Categoria previsione, Fonte, Settore).
