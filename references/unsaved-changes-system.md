# Unsaved changes system

**Scope:** rilevamento modifiche non salvate + warning. Fonte: **Osservato** (barra "Annulla/Salva" appare editando; warning alla chiusura **Da verificare**).

## Osservato / Dedotto
- Inline edit / modale: editando ≥1 campo appare una **barra `Annulla · Salva`** (stato "dirty").
- **Dedotto/Da verificare:** chiudere la modale (X/Annulla) con modifiche → warning "Perderai le modifiche?" (comportamento standard dell'org di riferimento). Da testare.

## Proposta Laravel (Proposto per Laravel)
- **Alpine `dirty` tracking:** ogni form/inline-edit tiene un flag `dirty` (confronto valori iniziali↔correnti).
- **Warning chiusura:** su close/X/navigazione con `dirty` → `confirm()` / dialog `x-crm.confirm-dialog` ("Ci sono modifiche non salvate. Uscire senza salvare?"). `beforeunload` per navigazione via browser.
- **Barra salva:** `Annulla` (ripristina valori iniziali, `dirty=false`) · `Salva` (submit).
- **Rischio implementativo:** falsi positivi (normalizzare valori prima del confronto); non bloccare l'utente in modo fastidioso; accessibilità del dialog.

## Priorità
- **V1:** dirty tracking + barra Annulla/Salva + warning su chiusura modale con modifiche. **V2:** `beforeunload`, autosave bozze. **V3:** recupero bozze.

## Open questions → `open-questions-and-assumptions.md`
Q5: warning unsaved-changes (testo, trigger) — da testare (aprire modale, modificare, chiudere).
