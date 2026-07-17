# Pagination & loading system

**Scope:** pagination, progressive loading, skeletons, loading/error states for lists. Source: **Observed** (small lists; behavior with many records **to verify**).

## Observed / Deduced
- Meta line "N items • Updated …"; **Refresh** reloads.
- The reference org uses **infinite scroll / progressive loading** (loads more rows on scroll) with a "50+" count; server-side sorting.
- **To verify:** load threshold, behavior beyond 2000 records, loading indicators.

## Proposed Laravel design (Proposed for Laravel)
- **Server-side pagination** (`LengthAwarePaginator`), page size from `config('crm.dashboard.page_sizes')` + `withQueryString()` (keeps view/filters/sort). Preferred over infinite-scroll for simplicity/SEO/robustness.
- **Loading:** skeleton rows (Alpine `x-cloak` + markup) during navigation; spinner only for async actions (quick action/inline save).
- **States:** empty (curated per view), error/unavailable (degrades like the existing `pipelineUnavailable` — never a 500), missing record (handled 404).
- **Performance:** indexes on filtered/sorted columns; targeted `select`; eager-load (no N+1); efficient count (or approximate for huge lists, V2).
- **V2:** optional infinite scroll (Alpine + `?cursor=` endpoint), count caching.

## Priority
- **V1:** classic pagination + skeleton + empty/error states + persistent query-string. **V2:** infinite scroll, caching, optimized count. **V3:** virtualization.

## Open questions → `open-questions-and-assumptions.md`
Q13: behavior with many records; thresholds; count on large datasets (performance target in `testing-strategy.md`).
