# Column management system

**Scope:** gestione colonne delle list view. Fonte: **Osservato** (menu controlli).

## Osservato
- Gear controlli → **"Seleziona i campi da visualizzare"** (scegli colonne visibili/nascoste + ordine) e **"Reimposta larghezze colonna"**.
- Header colonna → menu azioni: ordina, **a capo / taglia testo**, ridimensiona (drag bordo).
- **Da verificare:** dialog "Seleziona campi" (2 liste disponibili/selezionate + riordino); persistenza per utente/vista; larghezze salvate.

## Proposta Laravel (Proposto per Laravel)
- **Catalogo colonne** per oggetto (una sola definizione: `key, label, sortable, default_visible, align, formatter`).
- **UI** `x-crm.column-manager`: dialog 2-pane (disponibili/visibili) + drag reorder; toggle a capo/taglia; reset.
- **Persistenza:** V1 `localStorage` (per utente/browser); V2 in `saved_views.columns_json` (per vista, condivisibile).
- **Regole:** colonne always-visible (es. nome record + azione riga) non nascondibili; whitelist per query/ordinamento.
- **Rischio implementativo:** validare le colonne scelte (whitelist) prima di usarle in `select`/`orderBy`.

## Priorità
- **V1:** toggle colonne visibili + reorder (localStorage), always-visible protette. **V2:** larghezze/resize persistenti, a capo/taglia, colonne per vista salvata. **V3:** colonne calcolate.

## Open questions
Dialog seleziona-campi (struttura); persistenza larghezze; a-capo/taglia default.
