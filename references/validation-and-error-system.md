# Validation & error system

**Scope:** validazioni, messaggi d'errore, stati errore. Fonte: **Testato dinamicamente** (empty-save New Lead) + **Osservato**.

## Testato / Osservato
- **Required inline (Testato):** salvataggio con campi obbligatori vuoti → errore **"Compila questo campo."** sotto il campo, bordo rosso, focus sul primo invalido, salvataggio bloccato, modale aperta. (Lead: Cognome, Società.)
- **Osservato/Dedotto:** formato email/URL validato; date via date-picker; number/currency numerici; picklist con valori ammessi.
- **Da verificare:** validazioni cross-field, lunghezze massime, messaggi per formato, errori server-side (validation rule custom → nell'org demo non testabili senza toccare config: **⛔** documentare da conoscenza), errori di salvataggio (toast di errore), conflitti di concorrenza.

## Proposta Laravel (Proposto per Laravel)
- **FormRequest** per oggetto/azione: `required`, `email`, `url`, `numeric`, `date` (≥ oggi dove serve), `in:{picklist}`, `max:`; messaggi IT ("Compila questo campo." → `Il campo :attribute è obbligatorio.`).
- **Errori inline** sotto il campo (mirror del riferimento); riepilogo in cima alla modale se >1.
- **Errori server/azione:** toast di errore (`x-crm.toast`); nessun 500 → catch + messaggio leggibile.
- **Concorrenza:** optimistic lock (`updated_at`) → 409 con messaggio "record modificato da altri".
- **Business rules** (mirror validation rule del riferimento): in `FormRequest`/Action (es. non chiudere opportunità senza motivo persa) — dominio esplicito, testato.
- **Rischio implementativo:** validazione lato server SEMPRE (mai fidarsi del client); sanitizzazione; messaggi non espongono dettagli sensibili.

## Priorità
- **V1:** required + tipo + picklist inline, messaggi IT, toast errore, no-500. **V2:** cross-field, optimistic lock, business rules ricche. **V3:** regole configurabili (validation-rule-like).

## Open questions → `open-questions-and-assumptions.md`
Q4: validazioni complete per oggetto; NV2 (validation rule/Flow server-side non ispezionabili → dedurre).
