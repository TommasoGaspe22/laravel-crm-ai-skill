# Opportunity record page

**Source:** Observed/Tested (real record + stage transition + inline edit + activities). Base: `record-page-system.md`. Fields: `field-catalog.md` (Opportunity).

- **Header:** `Opportunity` / name (e.g. "CRM-SKILL-Manufacturing Co 001-"). Actions: **New Event · New Task · Log a Call · Edit · Create New Invoice · ▾**.
- **Path (Stage):** `Qualify → Meet & Present → Propose → Negotiate → Closed` (Closed = Won/Lost). **"Guidance for Success"** panel per stage (Tested: guidance "Make the offer…" on Propose). Two-step commit. See `stage-transition-system.md`. **Tested:** Qualify→Propose.
- **Sections (Observed):** `About` (Opportunity name, Account name [lookup], Close date, Amount, Description, Opportunity owner) · `Status` (Stage, Probability %, Forecast category, Next step).
- **Inline edit (Tested/Observed):** ✎ per field (e.g. Amount) → input + Cancel/Save bar.
- **Activity (Tested):** "Log a Call" → item in the timeline; "New Task" (Task: Subject+Due date).
- **Related list (Observed):** **Contact Roles · Files · Invoices**.
- **Kanban:** the opportunity is a card in the by-stage Kanban (`kanban-system.md`).
- **Laravel:** `CrmOpportunity` `belongsTo account`, `primary_contact`, `hasMany items` (V3), morph activities. Stage = config enum; `is_closed/is_won` derived; probability mapped from stage. See `data-model-full.md`.
- **To verify:** Won/Lost closing (status+loss-reason modal); Forecast category values; auto-probability per stage; line items/products.
