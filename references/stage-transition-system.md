# Stage / status transition system (Path)

**Scope:** il componente **Path** e le transizioni di stato/fase su Lead (Stato lead) e Opportunità (Fase). Fonte: **Testato dinamicamente** (2026-07-08: spostata un'opportunità Qualify → Propose via Path) + **Osservato** (Lead path).

## Indice
1. Path (anatomia) · 2. Transizione (testato) · 3. Regole · 4. Laravel · 5. Priorità · 6. Open questions

## 1. Path — anatomia (Osservato/Testato)
- Barra orizzontale di **chevron**: stadi completati (verde/tinta), corrente (scuro), futuri (grigio).
- **Lead** (Stato lead): `Nuovo → Contattato → Nurturing → Non qualificato → Convertito`.
- **Opportunità** (Fase): `Qualify → Meet & Present → Propose → Negotiate → Chiuso` (il chevron **"Chiuso"** raggruppa Closed Won/Lost; selezionandolo si apre la scelta stato di chiusura — **Da verificare**).
- **Pannello "Guida per il successo"** sotto il Path: testo guida per lo stadio selezionato (Osservato su Opp Propose: *"Make the offer. Does the quote cover…"*). → nel CRM target, testo guida per fase (config), utile per onboarding commerciali.
- Pulsante primario a destra, **dipendente dalla selezione**:
  - Stadio corrente selezionato → **"Contrassegna Stato come Completo/a"** (avanza al successivo).
  - Stadio diverso selezionato → **"Contrassegna come Fase corrente"** (imposta quello come corrente).

## 2. Transizione testata dinamicamente
- Click sul chevron **"Propose"** → lo stadio si seleziona (mostra guida) e il pulsante diventa **"Contrassegna come Fase corrente"**.
- Click sul pulsante → **Fase aggiornata a Propose** (verificato: record page evidenzia lo stadio corrente = Propose **e** list view mostra `Fase = Propose` **e** Kanban colloca la card nella colonna Propose). Nessun campo obbligatorio richiesto per questa transizione.
- **Chiusura (Testato dinamicamente):** selezionando il chevron **"Chiuso"** il pulsante diventa **"Seleziona Fase"** → apre la modale **"Chiudi quest'opportunità"** con **campo required "Fase"** = picklist *"Seleziona una fase chiusa…"* (**Closed Won / Closed Lost**) → `Annulla · Salva`. **Nel trial NON compare un campo "Motivo persa"** (sarebbe un campo custom). → in Laravel `loss_reason` resta opzionale/config, richiesto solo se il team lo vuole su "Persa".

## 3. Regole (Osservato/Dedotto)
- Selezione ≠ commit: serve il pulsante per confermare (due step). → evita cambi accidentali.
- Cambiare fase aggiorna anche **Probabilità** e **Categoria di previsione** (l'org di riferimento le lega alla fase) — **Da verificare** i valori esatti.
- Chiusura (Won/Lost) è terminale; l'opportunità risulta chiusa (`is_closed`/`is_won`).

## 4. Blueprint Laravel (Proposto per Laravel)
- **Componente** `x-crm.path` (vedi `component-catalog.md`): props `:steps`, `current`, `:guidance`, `:onSelect`. Due-step: seleziona → conferma.
- **Action** `ChangeStage(opportunity, toStage, extra?)`:
  - valida che `toStage ∈ config('crm_opportunity_stages')`;
  - se `toStage` terminale (vinta/persa) → richiede campi (es. `loss_reason` per persa) via modale;
  - aggiorna `stage`, deriva `probability` (mappa fase→probabilità di default, override manuale ammesso), `is_closed`/`is_won`;
  - scrive `crm_activities` di tipo "cambio fase" (timeline/audit): `{from, to, user, at}`;
  - policy `update`; transazione; toast.
- **Lead status** analogo: Action `ChangeLeadStatus`; lo stato "Convertito" si raggiunge **solo** via conversione (non selezionabile a mano) — vedi `lead-conversion-system.md`.
- **Rischio implementativo:** validazione transizioni (consentire salti? l'org di riferimento li consente); campi obbligatori alla chiusura; audit; concorrenza (optimistic lock).

## 5. Priorità
- **V1:** Path Lead + Opportunità con commit due-step, cambio fase con audit in timeline, chiusura Won/Lost con motivo persa.
- **V2:** guida per fase (testo config), probabilità auto per fase, validazioni campi-obbligatori per transizione.
- **V3:** regole di transizione configurabili (blocchi, approvazioni).
- **Non replicare:** approval process complessi, path per oggetti multipli custom.

## 6. Open questions → `open-questions-and-assumptions.md`
Modale chiusura Won/Lost + campi richiesti; mapping fase→probabilità/forecast; se l'org di riferimento consente salti di fase; campi obbligatori per transizione.
