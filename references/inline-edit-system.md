# Inline edit system

**Scope:** modifica inline dei campi su record page e list view. Fonte: **Osservato/Testato** (2026-07-08: pencil apre edit inline con barra "Annulla/Salva"; save selector calibrazione in corso).

## Osservato / Testato dinamicamente (2026-07-08)
- Ogni field row su record page mostra un **pencil ✎** (title "Modifica {Campo}") in hover/focus.
- Click ✎ → il campo diventa **input editabile** in-place (text/number/picklist/lookup a seconda del tipo). Es. `Nome account` diventa editabile con **X** per rimuovere il lookup.
- Editando appare una **barra di salvataggio** in basso con **`Annulla · Salva`** (batch: più modifiche, un solo Salva). → anche meccanismo **unsaved-changes** (`unsaved-changes-system.md`).
- **Testato:** modificato inline **Ammontare** di un'opportunità a **30.000 €** → salvato e persistito (visibile in record, list e Kanban/split). Edit **batch multi-campo** (Ammontare + Descrizione) via la barra Salva.
- Su list view: modalità **"Modifica elenco"** (toolbar) abilita edit inline celle (mass inline edit). **Da verificare** in dettaglio.
- **Da verificare:** validazioni inline (required svuotato), campi non editabili (formula/rollup), edit picklist/lookup inline, conflitti/concorrenza.

## Proposta Laravel (Proposto per Laravel)
- **Componente** `x-crm.detail-section` con field rows editabili (Alpine `inlineEdit`): click ✎ → swap value↔input; stato `dirty` per riga; barra flottante `Annulla/Salva` quando ≥1 campo dirty.
- **Endpoint** `PATCH /crm/{obj}/{id}` (parziale, solo campi modificati) con `FormRequest` (stesse regole del form completo, ma `sometimes`), policy `update`, ritorno JSON (valore aggiornato + eventuali errori per-campo) per aggiornare la UI senza reload.
- **Tipi:** text/number/date/currency inline; picklist → combobox; lookup → ricerca async (endpoint dedicato).
- **Validazione:** errori per-campo mostrati sotto la cella; se un campo required viene svuotato → blocco salvataggio.
- **Rischio implementativo:** concorrenza (optimistic lock via `updated_at`); campi calcolati non editabili; XSS su testo; permessi per-campo (field-level) → V2/V3.
- **Accessibilità:** ✎ è `button` con `aria-label`; input riceve focus; Esc annulla; la barra salva è raggiungibile da tastiera.

## Priorità
- **V1:** inline edit su record page per campi semplici (text/number/date/picklist), batch save bar, validazione base.
- **V2:** inline edit su list view ("Modifica elenco"), lookup inline, mass edit selezione multipla.
- **V3:** field-level security, edit condizionale.
- **Non replicare:** edit inline su campi calcolati/formula.

## Open questions → `open-questions-and-assumptions.md`
Q3 inline edit (batch/validazioni/campi non editabili/mass edit list) — Da testare in dettaglio.
