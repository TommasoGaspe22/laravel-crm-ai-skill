# Lead conversion system

**Scope:** flusso "Converti lead" → Account + Referente + Opportunità. Fonte: **Testato dinamicamente** (convertito `CRM-SKILL-Lead Nuovo 001` il 2026-07-08) + **Osservato** (modale). Include mapping campi e blueprint Laravel transazionale.

## Indice
1. Trigger · 2. Modale (anatomia) · 3. Comportamento testato · 4. Mapping campi · 5. Laravel · 6. Test · 7. Priorità · 8. Open questions

## 1. Trigger (Osservato)
Pulsante **`Converti`** nell'header azioni della record page Lead (primo pulsante, prima di `Cambia titolare`/`Modifica`). Disponibile finché il lead non è già convertito.

## 2. Modale "Converti lead" (Osservato)
Tre sezioni, ognuna con radio **Crea / Scegli**:
- **Account:** `Crea` (`*Nome account` pre-compilato da `Società`) — *oppure* `Scegli` (ricerca "Cerca account corrispondenti", mostra N corrispondenze → dedup).
- **Referente:** `Crea` (nome pre-compilato da Nome+Cognome) — *oppure* `Scegli` (mostra "N corrispondenza Referente rilevata").
- **Opportunità:** `Crea` (nome pre-compilato = `{Società}`) + checkbox **"Non creare un'opportunità alla conversione"** — *oppure* `Scegli`.
- Bottom: `*Titolare record` (owner), `*Stato convertito` (picklist; default nella scala = ultimo stato "Convertito"/"Qualificato").
- Footer: `Annulla · Converti`. Helper: *"Trasformeremo questo lead in tre nuovi record."*

## 3. Comportamento testato dinamicamente (2026-07-08)
Accettando i default (tutti "Crea") e cliccando **Converti**:
- **Successo:** schermata **"Il tuo lead è stato convertito"** (illustrazione bandiera) con **3 card**: ACCOUNT · REFERENTE · OPPORTUNITÀ, ciascuna coi campi chiave (Account: Telefono/Sito/Indirizzo/Titolare; Referente: Qualifica/Telefono/Email; Opportunità: Nome/Data chiusura/Ammontare/Titolare).
- **Record creati:** Account `CRM-SKILL-Azienda Manufacturing 001`, Referente `Mario CRM-SKILL-Lead Nuovo 001` (email `mario.rossi@crm-skill.test` **trasferita**, telefono `3000000001` **trasferito**), Opportunità `CRM-SKILL-Azienda Manufacturing 001` con **Data chiusura auto = 30/09/2026** (default ~fine trimestre/+N giorni).
- **Dedup non forzato:** l'azienda esisteva già come Account, ma con default "Crea" è stato creato un **Account duplicato** (l'org di riferimento non forza "Scegli"; offre solo la ricerca). → **Rischio implementativo:** senza dedup esplicito si creano duplicati.
- **Lead post-conversione:** rimane accessibile (vista dettaglio dell'org di riferimento), stato → Convertito; i campi diventano di sola lettura; azione `Converti` non più disponibile.
- Footer post: `Nuova operazione` / `Vai a lead`. Helper "Fai clic sul nome dell'opportunità per far avanzare questa trattativa".
- **Da verificare:** trasferimento di Note/File/Attività aperte dal Lead ai nuovi record; audit/timeline sui record creati; comportamento con "Non creare opportunità"; conversione con "Scegli" account esistente (dedup).

## 4. Mapping campi (Osservato/Dedotto)
| Lead | → Account | → Referente | → Opportunità |
|---|---|---|---|
| Società | Nome account | Nome account (rel) | Nome opportunità (`{Società}`), Account (rel) |
| Nome/Cognome | — | Nome/Cognome | — |
| Email | — | Email ✔(trasferita) | — |
| Telefono | Telefono | Telefono ✔ | — |
| Qualifica | — | Qualifica | — |
| Sito Web/Settore/N.dip/Reddito | Account (campi corrispondenti) | — | — |
| Titolare | Titolare | Titolare | Titolare |
| — | — | — | Data chiusura (auto), Fase (default 1ª) |

## 5. Blueprint Laravel (Proposto per Laravel)
Action `ConvertLead` (in transazione DB):
```
DB::transaction(function() use ($lead, $opts) {
  $account = $opts->accountId ? CrmAccount::find($opts->accountId)
            : CrmAccount::firstOrCreate(['name'=>$lead->company], [...map...]); // dedup opzionale
  $contact = $opts->contactId ? CrmContact::find($opts->contactId)
            : CrmContact::create([...map da lead, 'account_id'=>$account->id]);
  $opp = null;
  if (!$opts->skipOpportunity) $opp = CrmOpportunity::create([
     'name'=>$opts->oppName ?? $lead->company, 'account_id'=>$account->id,
     'primary_contact_id'=>$contact->id, 'stage'=>'qualifica',
     'close_date'=>$opts->closeDate ?? now()->addDays(90), 'owner_id'=>$lead->owner_id]);
  $lead->update(['status'=>'convertito','converted_at'=>now(),
     'converted_account_id'=>$account->id,'converted_contact_id'=>$contact->id,'converted_opportunity_id'=>$opp?->id]);
  // trasferisci attività/note/file polimorfici dal lead ai nuovi record
  // scrivi crm_activities di tipo "conversione" su lead+account+opp (audit/timeline)
});
```
- **Dedup (Proposto):** offrire match su `company`/`email` (come "Scegli") + opzione "usa esistente"; default configurabile (evita duplicati ciechi — a differenza dell'org di riferimento).
- **Policy:** `convert` consentita a staff owner/admin; validare owner + stato convertito.
- **Rollback:** transazione → se una create fallisce, rollback totale (nessun record orfano).
- **Audit:** riga `crm_activities`/`audit_events` "Lead convertito" collegata a lead, account, opp.
- **Rischio implementativo:** trasferimento relazioni polimorfiche (attività/note/file) va fatto dentro la transazione; gestire owner mancante; timezone su close_date.

## 6. Test consigliati (Laravel)
Feature: conversione con nuovo account · con account esistente (dedup) · senza opportunità · dati minimi · rollback su errore (mock failure) · trasferimento attività/note · policy (utente non autorizzato → 403) · stato lead post = convertito · idempotenza (non ri-convertire).

## 7. Priorità
- **V1:** conversione crea Account+Referente+Opportunità in transazione, con opzione dedup base e "no opportunità", audit. 
- **V2:** UI dedup ricca (match multipli), trasferimento note/file/attività completo, mapping campi configurabile.
- **V3:** regole di conversione automatiche, merge avanzato.
- **Non replicare:** conversione massiva batch complessa (come nell'org di riferimento), a meno di reale necessità.

## 8. Open questions → `open-questions-and-assumptions.md`
Trasferimento Note/File/Attività (Da verificare); dedup con "Scegli" (Da verificare); valori `Stato convertito`; comportamento "Non creare opportunità".
