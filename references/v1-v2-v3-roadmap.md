# V1 / V2 / V3 Roadmap

**Scope:** justified prioritization of every feature. Source: synthesis of all `*-system.md` files. Rule: don't put everything in V1; justify it.

## V1 — essential operational CRM (indispensable for the sales team)
- **Objects:** Lead (on the existing pipeline), Account, Contact, Opportunity (+ basic Activities). Relations + **Lead conversion**.
- **List view:** dense table, search, single sort, **curated filters** (status/stage/owner/channel/bucket/date/flag) via query-string + chips, **preset views** + default pin, row actions, **essential select+bulk** (change status/assign/flag/delete/export), pagination, empty/loading/error states. **Grouped "by stage"** view (Opportunity).
- **Record page:** header+actions, **detail sections + inline edit**, **Path** (Lead status, Opp stage) with commit + audit, **activity panel + timeline** (task/call/note), main related lists, **delete with confirmation**.
- **Conversion:** Account+Contact+Opportunity in one transaction, basic dedup, "no opportunity" option, audit.
- **Quick actions:** contacted, outcome, follow-up (+3/7/14), priority, status, note, close (won/lost+reason).
- **Cross-cutting:** owner policy (admin vs sales, delete admin-only, "assigned to me"), inline validation, toast, basic global search, app-shell reuse + object-nav, sensitive-action audit, demo seed, tests.
- *Rationale:* covers the daily work (who to contact, moving deals forward, logging activity) with minimal friction; leverages existing data/engine.

## V2 — advanced CRM (productivity and control)
- **Kanban drag&drop** (SortableJS) + column sums + won/lost closing modal.
- **Generic filter builder** (field/operator/value, relative dates, save view, OR), **user saved views** (CRUD + basic sharing).
- **Configurable + resizable columns** (per view), **mass inline edit**, **split view**.
- **Rich activities:** events/calendar (FullCalendar), logged email, **reminders/notifications**, timeline filters.
- **Rich Notes/Files**, related list with inline actions + pagination.
- **Full unsaved-changes** handling, optimistic lock, rich business rules.
- **Team/manager roles**, configurable owner-scope, rich audit, search (Scout), count caching, jobs/queue for bulk/notifications.
- *Rationale:* increases efficiency and governance as volume/usage grows; introduces the only 2 dependencies (SortableJS, FullCalendar) only once the data justifies them.

## V3 — Enterprise / optional
- **Product/price-book/line-item catalog**, advanced forecasting, **M:N contact roles**, quotes.
- **Field-level security**, advanced sharing rules/teams, granular shared views, impersonation (if needed).
- **Public API**, integrations (external email/telephony/calendar), rich import wizard, data-grid virtualization.
- *Rationale:* value only for structured organizations; high cost/complexity.

## Do not replicate (from the reference platform)
Flow/Process Builder/automation builder · Report/Dashboard builder · built-in AI assistant · complex sharing rules/org-wide defaults · app switcher/utility bar · advanced multi-currency · per-profile page-layout editor · conversation insights · embedded chat integrations.
*Rationale:* platform-only features not needed for the target app's sales flow, too heavy/proprietary, or replaceable with simpler solutions.

## V1→V2→V3 promotion criteria
A feature only moves up in priority if: (a) it solves a real friction point observed by the team, (b) the data justifies it (e.g. Kanban once stage is stable), (c) the cost/risk is sustainable on the stack (Blade/Alpine/SQLite).
