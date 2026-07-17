# Pipeline system (Opportunity)

**Scope:** the value-based sales pipeline built on Opportunities. Umbrella document: points to `opportunity-record-page.md`, `stage-transition-system.md`, `kanban-system.md`. Source: **Observed/Tested**.

## Concepts (Observed/Tested)
- **Opportunity** = a deal with a **Stage** (pipeline), **Amount**, **Close date**, **Probability**, **Forecast category**, **Next step**, Account+Contact, Owner.
- **Stages (real):** `Qualify → Meet & Present → Propose → Negotiate → Closed Won / Closed Lost`. Target app: `Qualify → Present → Propose → Negotiate → Won / Lost`.
- **Path** with per-stage guidance (Tested: Qualify→Propose transition).
- **Kanban** by stage with counts + € sums (Tested).
- **Closing** Won/Lost is terminal (**to verify** status+loss-reason modal).

## Pipeline metrics (Proposed for Laravel)
- Per stage: `count`, `sum(amount)`, `avg(probability)`.
- **Weighted pipeline:** `sum(amount * probability)` (simple forecast).
- Open vs. closed (win rate = won/(won+lost)); overdue (close_date < today & not closed).
- Views: Open · Mine · Closing this month · Won · Lost · By stage (grouped) · Kanban.

## Laravel
`CrmOpportunity` + `ChangeStage`/`CloseOpportunity` (`stage-transition-system.md`), Kanban (`kanban-system.md`), record page (`opportunity-record-page.md`), data/indexes (`data-model-and-migrations.md`).

## Priority
- **V1:** Opportunity object + stages + Path + record page + grouped "by stage" view + basic metrics (counts/sums/weighted) + won/lost closing.
- **V2:** Kanban drag&drop, forecast, advanced views.
- **V3:** products/line items, quotes, advanced forecasting, teams.

## Open questions
Won/lost closing (modal/fields); auto-probability per stage; forecast category values; line items.
