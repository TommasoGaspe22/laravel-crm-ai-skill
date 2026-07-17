# Pagination & loading system

**Scope:** paginazione, caricamento progressivo, skeleton, stati loading/errore delle liste. Fonte: **Osservato** (liste piccole; comportamento con molti record **Da verificare**).

## Osservato / Dedotto
- Meta line "N elementi • Aggiornato …"; **Aggiorna** (refresh) ricarica.
- L'org di riferimento usa **infinite scroll / caricamento progressivo** (carica altre righe scrollando) con conteggio "50+"; ordinamento server-side.
- **Da verificare:** soglia caricamento, comportamento >2000 record, indicatori loading.

## Proposta Laravel (Proposto per Laravel)
- **Paginazione server-side** (`LengthAwarePaginator`), page size da `config('crm.dashboard.page_sizes')` + `withQueryString()` (mantiene view/filtri/sort). Preferita a infinite-scroll per semplicità/SEO/robustezza.
- **Loading:** skeleton rows (Alpine `x-cloak` + markup) durante navigazione; spinner solo per azioni async (quick action/inline save).
- **Stati:** empty (curato per vista), errore/unavailable (degrada come `pipelineUnavailable` esistente — mai 500), record mancante (404 gestito).
- **Performance:** indici su colonne filtrate/ordinate; `select` mirati; eager-load (no N+1); count efficiente (o approssimato per liste enormi, V2).
- **V2:** infinite scroll opzionale (Alpine + endpoint `?cursor=`), cache conteggi.

## Priorità
- **V1:** paginazione classica + skeleton + empty/error states + query-string persistente. **V2:** infinite scroll, cache, count ottimizzato. **V3:** virtualizzazione.

## Open questions → `open-questions-and-assumptions.md`
Q13: comportamento con molti record; soglie; count su dataset grandi (target performance in `testing-strategy.md`).
