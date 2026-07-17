# Opportunity pipeline & Kanban (index)

Umbrella document linking the Opportunity pipeline systems. Source: **Observed/Tested**.

- **Object & record page:** `opportunity-record-page.md` — fields (About/Status), actions, related lists (Contact Roles/Files/Invoices).
- **Stages & transitions:** `stage-transition-system.md` — Path `Qualify→…→Closed`, two-step commit, per-stage guidance (Tested: Qualify→Propose).
- **Kanban:** `kanban-system.md` — board by stage, counts + € sums, cards with a drag handle (Tested); drag&drop V2.
- **Pipeline & metrics:** `pipeline-system.md` — views, weighted forecast, win rate.
- **Data:** `data-model-and-migrations.md` (`crm_opportunities`).
- **List-view constraint:** Kanban/Filters only on real views, not on "Recent" (Tested — `list-view-system.md`).

**V1:** object + stages + Path + record page + grouped "by stage" view + basic metrics. **V2:** Kanban drag&drop + closing modal + forecast. **V3:** products/quotes/teams.

**To verify:** real drag&drop + confirmations; Won/Lost closing (status+loss reason); auto-probability/forecast per stage; performance with many cards.
