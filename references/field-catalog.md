# Field catalog — per-object field specification

**Scope:** ogni campo osservato negli oggetti dell'org di riferimento → tipo DB, tipo UI, obbligatorietà, validazioni, picklist, mapping Laravel. Fonte: modali New/Edit e record page (**Osservato**), test di validazione (**Testato dinamicamente**). L'org demo è quasi vuoto → alcuni valori picklist secondari sono **Da verificare**.

Colonne: Campo (label IT) · UI · Tipo Laravel/DB · Obblig. · Note/picklist · Priorità.

Legenda obblig.: **R!** = required bloccante (testato) · **R** = required indicato (`*`) ma con default → non blocca · `-` opzionale.

---

## Lead  *(Osservato: modale New; Testato dinamicamente: validazione empty-save)*

Sezioni layout (Osservato): **About · Get in Touch · Segment**. Required bloccanti (Testato dinamicamente, msg "Compila questo campo."): **Cognome**, **Società**. `Stato lead` è `*` ma ha default → non blocca.

| Campo | UI | Laravel/DB | Obbl. | Note | Prio |
|---|---|---|---|---|---|
| Titolo (salutation) | picklist | `string(20)` nullable | - | Sig./Sig.ra/… (Da verificare valori) | V2 |
| Nome (first name) | text | `string(80)` | - | | V1 |
| Cognome (last name) | text | `string(80)` | **R!** | | V1 |
| Nome e cognome | compound (calcolato) | accessor `full_name` | R | display; non colonna | V1 |
| Società | text | `string(200)` | **R!** | diventa Account alla conversione | V1 |
| Stato lead | picklist | `enum status` | R(default) | `Nuovo·Contattato·Nurturing·Non qualificato·Convertito` (Osservato Path) | V1 |
| Qualifica (title) | text | `string(120)` | - | ruolo referente | V1 |
| Sito Web | url | `string(190)` | - | | V2 |
| Descrizione | textarea | `text` | - | | V1 |
| Titolare lead (owner) | lookup(User) | `owner_id` FK | R(default=me) | | V1 |
| Valutazione (rating) | picklist | `enum rating` | - | `Caldo·Tiepido·Freddo` (Hot/Warm/Cold) — proxy priorità | V1 |
| Telefono | phone | `string(40)` | - | | V1 |
| Email | email | `string(190)` | - | valida formato email | V1 |
| Indirizzo | compound address | vedi sotto | - | Paese·Via·CAP·Città·Stato/Provincia | V2 |
| ↳ Paese/Via/CAP/Città/Provincia | text | `string` ciascuno | - | | V2 |
| N. di dipendenti | number | `unsignedInteger` | - | | V2 |
| Reddito annuale | currency | `decimal(15,2)` | - | | V2 |
| Fonte del lead | picklist | `enum lead_source` | - | valori Da verificare (Web, Referral, …) | V1 |
| Settore | picklist | `string/enum industry` | - | valori Da verificare | V1 |

**Proposto per Laravel:** i campi Lead confluiscono in `commercial_pipeline_entries` (esistente) + colonne additive (`data-model.md`). `Valutazione` mappa alla priorità; `Fonte del lead` → `lead_source`; `Settore` → `business_area`.
**Rischio implementativo:** `Nome e cognome` è compound (salutation+first+last) → in Laravel tenere campi separati + accessor, non un unico campo.
**Da verificare:** valori esatti di picklist `Titolo`, `Fonte del lead`, `Settore` (org vuoto) → estrarre aprendo ciascuna picklist nella modale, oppure da record reali.

---

## Account  *(Osservato: record page; New modal Da verificare in dettaglio)*
Sezioni: About · Get in Touch · History. Campi (Osservato): Nome account* · Sito Web · Tipo (picklist) · Descrizione · Società controllante (lookup self) · Titolare · Telefono · Indirizzo fatturazione · Indirizzo spedizioni. **Da completare** con New-modal capture (required, picklist Tipo).

## Referente (Contact)  *(Osservato: record page)*
Campi: Nome e cognome* · Nome account (lookup) · Qualifica · Fa capo a (lookup self) · Descrizione · Titolare · Telefono · Email · Indirizzo postale. **Da completare** con New-modal capture.

## Opportunità  *(Osservato: modale New — completo)*
Sezioni: **About · Status**. Campi: *Nome opportunità · *Nome account (lookup) · *Data chiusura (date) · Ammontare (currency) · Descrizione (textarea) · Titolare · *Fase (picklist) · Probabilità (%) (number) · *Categoria di previsione (picklist) · Fase successiva (text). Fasi (Osservato): `Qualify·Meet & Present·Propose·Negotiate·Closed Won·Closed Lost`. **Da verificare:** valori `Categoria di previsione`; se `Motivo perdita` compare su Closed Lost.

## Task / Event (Attività)  *(Da verificare — New modal non ancora aperta)*
Attesi (standard): Task → Oggetto · Scadenza · Stato · Priorità · Assegnato a · Nome(contatto) · Correlato a · Commenti. Event → Oggetto · Inizio · Fine · Luogo · Correlato a. **Da catturare dinamicamente.**

## Case (Caso)  *(Da verificare — oggetto forse non attivo nella trial)*
Da esplorare: disponibilità tab, campi (Oggetto, Stato, Priorità, Origine, Account/Contact). Se non attivo → ⛔ + procedura futura.

---

## Valori picklist reali (Testato dinamicamente, 2026-07-08)
Estratti aprendo le picklist nelle modali New (mappare a `config('crm.crm_*')`):
- **Lead — Stato lead:** `Nuovo · Contattato · Nurturing · Qualificato · Non qualificato` (+ `Convertito` solo via conversione, non selezionabile). *(nota: la scala reale include `Qualificato`; il Path visuale mostrava un sottoinsieme.)*
- **Lead — Valutazione (rating):** `Urgente · Mediamente urgente · Non urgente` *(NON Caldo/Tiepido/Freddo — correzione).* → mappa a priorità.
- **Lead — Titolo (salutation):** `Sig. · Sig.ina · Sig.ra · Dott. · Prof. · Mx.`
- **Lead — Fonte del lead:** `Pubblicità · Segnalazione (referral) dipendente · Segnalazione (referral) esterna · Partner · Public Relations · Seminar - Internal · Seminar - Partner · Fiera · Web · Passaparola · Altro`
- **Lead — Settore:** Agricoltura, Abbigliamento, Banche, Biotecnologie, Industria chimica, Comunicazioni, Edilizia, Consulting, Istruzione, Elettronica, Energia, Ingegneria, Intrattenimento, Ambiente, Finanza, Ristorazione, Pubblica amministrazione, Sanità, Strutture ricettive, Servizi assicurativi, Macchinari, Produzione, Media, No profit, Attività ricreative, Vendita al dettaglio, Spedizioni, Tecnologie, Telecomunicazioni, Trasporti, Pubblici servizi, Altro. *(condiviso Lead/Account come `industry`)*
- **Account — Tipo:** `Analista · Concorrente · Cliente · Integratore · Investitore · Partner · Stampa · Cliente potenziale · Rivenditore · Altro`
- **Opportunità — Fase:** `Qualify · Meet & Present · Propose · Negotiate · Closed Won · Closed Lost`
- **Opportunità — Categoria di previsione:** `Omessa · In corso di realizzazione · Massimo potenziale · Impegno · Chiuso`
- **Chiusura opportunità:** modale "Chiudi quest'opportunità" → campo required **Fase** ("Seleziona una fase chiusa" = Closed Won/Lost). **Nessun campo "Motivo persa"** nel trial (sarebbe custom → `loss_reason` proposto in Laravel resta opzionale/config).
- **Case:** oggetto **disponibile** nell'org (grid presente) → campi **Da verificare** (New Case).

## TODO catalogo residuo
- [ ] Contact: New modal (campi/required) — Osservato da record; New modal Da verificare
- [ ] Task/Event/Email: campi composer completi (Task: Oggetto+Scadenza Osservato)
- [ ] Case: campi (New Case)
