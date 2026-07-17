# Filter builder system

**Scope:** the list-view filters panel (treated as its own module). Source: **Observed** (real panel on "All Opportunities"); detailed operators **to verify** (dynamic testing of individual operators still to do).

## Observed / Dynamically tested (2026-07-08)
- **Trigger:** **Filters** button in the toolbar (enabled **only** on real views e.g. "All Opportunities", not on "Recent").
- **Right panel** ("Filters", X to close), not a blocking overlay (table still visible on the left). Header: **Cancel · Save (▾ = Save / Save As)**.
- **Scope:** **"Filter by owner"** (All / Mine / queue), value "All Opportunities".
- **Logic:** **"Match all these filters"** (= AND by default) + **"Add filter logic"** (custom boolean expression AND/OR/NOT with condition numbers).
- **Filters:** **"Add filter"** → **"New filter*"** chip (highlighted) to configure **Field → Operator → Value**; **X** to remove; **"Remove all"**.
- **Saving:** Save applies; "Save As" creates a new view from the filter combination.
- **To verify (remaining):** exact operator list per field type (open the chip → field → operator); relative dates; active-filter chip indicators above the table; zero-results behavior; persistence. *(Structure confirmed; the reference org's standard operator values are in §Architecture.)*

## Laravel architecture (Proposed for Laravel)
Dedicated module (mirrors the reference org but safe and performant):
- `FilterDefinition` (a view's filter list) · `FilterField` (filterable field: name, type, label) · `FilterOperator` (per field type) · `FilterGroup` (nested AND/OR) · `AppliedFilter` (field+operator+value) · `FilterValueResolver` (parses values, incl. **relative dates** "today/±N days") · `QueryBuilderAdapter` (translates to Eloquent/SQL) · `SavedView` (persistence) · `FilterPolicy` (filterable fields per role) · `FilterSerialization` (query-string ↔ structure, shareable views via URL).
- **Operators per type:**
  - text: `=`, `≠`, `contains`, `doesn't contain`, `starts with`
  - number/amount: `=`, `≠`, `<`, `≤`, `>`, `≥`, `between`
  - date: absolute (`=`,`<`,`>`,`between`) + **relative** (`today`, `yesterday`, `last N days`, `this month`, `next N days`)
  - picklist/enum: `in`, `not in`
  - boolean: `true/false`
  - owner/lookup: `= me`, `= user`, `in team`
- **Safe queries:** field+operator whitelist (never direct input into SQL); parameterized bindings (anti SQL-injection); indexes on filterable fields (`stage`, `owner_id`, `status`, dates); consistent timezone for relative dates; complexity limits (max N conditions) for performance; avoid N+1 with eager-load.
- **Persistence & URL:** filters serialized in the query-string (bookmarkable/shareable views) + `saved_views` to save them.
- **UI:** `x-crm.filter-panel` (drawer): owner scope, condition list (add/remove/edit), Apply/Reset, **active-filter chips** above the table, curated zero-results state.

## Priority
- **V1:** curated per-object filters (status/stage/owner/channel/follow-up bucket/date range/flag) via query-string + active chips + reset. Simple AND.
- **V2:** generic filter builder (dynamic field/operator/value), relative dates, save view, OR.
- **V3:** nested condition groups, custom logic, cross-object filters.
- **Do not replicate (V1):** advanced boolean logic, filters on formula/rollup fields.

## Open questions → `open-questions-and-assumptions.md`
Q1 (operators per type, AND/OR/groups, relative dates, zero results, saving, persistence) — **to test dynamically** by opening and applying real filters.
