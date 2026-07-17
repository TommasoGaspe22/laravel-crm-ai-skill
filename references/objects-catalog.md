# Sales-flow objects catalog (captured from the live org)

The full reference sales flow, replicated for your own app. Each object below was inspected **read-only** in the org (Italian locale, trial with default + light customization). Columns, fields, picklists, and actions marked "(real)" were captured from the org; the rest is standard for this kind of reference org. `*` = required field.

## The flow (how objects connect)

```
Lead ──(Convert)──▶ Account ──has many──▶ Contact
                        │
                        └──has many──▶ Opportunity ──line items──▶ Product × Price Book (price)
Activity (Task/Event) ──related to──▶ Lead | Account | Contact | Opportunity
```

- **Account** is the hub (a company). Contacts and Opportunities hang off it.
- **Lead** is top-of-funnel; "Convert" turns one Lead into Account + Contact + Opportunity.
- **Opportunity** carries the money + stage pipeline.
- **Activity** logs the interactions against any object (replaces the current *derived* timeline with real events).

## Coexistence with the existing CRM (important)

Your app already has the flat `commercial_pipeline_entries` (operational lead-working list) — **keep it**. Layer the relational objects as **new tables** (additive, never touching pipeline). Recommended split:
- `commercial_pipeline_entries` + the `/crm/leads` list view = top-of-funnel lead operations (already specced in modules.md).
- New relational objects (`crm_accounts`, `crm_contacts`, `crm_opportunities`, …) = the qualified/downstream sales flow.
- **Conversion** bridges them: a `convertLead()` action creates Account + Contact + Opportunity from a pipeline entry and marks it converted (mirrors the reference "Convert").

Every object follows the **same module pattern** already defined for Lead in `modules.md`: reference-style list view (saved views, filters, sort, bulk/row actions) + detail page (highlights, path/stage, fields, related lists, timeline) + minimal-click quick actions. This file defines the per-object specifics; `data-model-full.md` defines the schema.

---

## 1. Account — Companies

- **List columns (real):** Account name · Phone · Account owner.
- **Record fields (real):** Account name* · Website · Type · Description · Parent account (self-FK) · Owner · Phone · Billing address · Shipping address.
- **Header actions (real):** New Contact · New Opportunity · Edit.
- **Related lists:** Contacts, Opportunities, Activities, Files.
- **Target app:** table `crm_accounts`. Reuse `company`/`business_area`/`address` semantics from the pipeline for backfill. Saved views: All · My accounts · Customers (Type=Customer) · Prospects.

## 2. Contact

- **List columns (real):** Full name · Account name · Phone · Email · Contact owner.
- **Record fields (real):** Full name* · Account name (FK→Account) · Title (job title) · Reports to (self-FK) · Description · Owner · Phone · Email · Mailing address.
- **Header actions (real):** New Opportunity · Edit.
- **Target app:** table `crm_contacts`, `belongsTo account`, self-relation `reports_to`. Backfill from pipeline `lead_name/role/email/phone`.

## 3. Opportunity (value pipeline)

- **List columns (real):** Opportunity name · Account name · **Stage** · Close date · Opportunity owner.
- **Record/New fields (real):** Opportunity name* · Account name* (FK) · Close date* · Amount · Description · Owner · **Stage\*** · Probability (%) · Forecast category · Next step.
- **Stages / Stage picklist (real, in this org):**
  `Qualify → Meet & Present → Propose → Negotiate → Closed Won → Closed Lost` (plus `--None--`).
  **Target-app adaptation, same order:** `Qualify → Present → Propose → Negotiate → Won → Lost`. Store a stable key; render the localized label. `Won`/`Lost` are the closed states.
- **Target app:** table `crm_opportunities` with `stage` (enum), `amount`, `close_date`, `probability`, `next_step`, `account_id`, `contact_id` (primary), `owner_id`. The **status path** on the detail page uses these stages (this is the reference "Path"). Saved views: Open · Mine · Closing this month · Won · Lost · By stage.

## 4. Products / Price Books — Product2 / Pricebook2

- **Product list columns (real):** Product name · Product class · Product code · Product description · Product family.
- **Pricebook list columns (real):** Price book name · Description · Start date · End date · Last modified date · Active.
- **Relationship:** a *PricebookEntry* links Product + Pricebook + price; an *OpportunityLineItem* links Opportunity + PricebookEntry + quantity.
- **Target app:** `crm_products`, `crm_price_books`, `crm_price_book_entries`, `crm_opportunity_items`. **Flag: likely more than most apps need at first** — treat as a later phase; V1 of the sales flow can let an Opportunity carry a free-text/product-less `amount`. Build the catalog only when the team sells a real SKU list.

## 5. Activities / Calendar — Task + Event

- **Task fields (standard):** Subject · Due date · Status · Priority · Assigned to (owner) · Name (contact) · Related to (account/opp) · Comments.
- **Event fields (standard):** Subject · Start · End · Location · Invitees · Related to.
- **Target app:** one polymorphic table `crm_activities` (`type` = task|call|email|event, `subject`, `due_at`/`start_at`/`end_at`, `status`, `owner_id`, `subject_type`+`subject_id` morph to lead/account/contact/opportunity, `notes`). **This replaces the *derived* timeline** with real logged activities — the upgrade the architecture doc deferred to V2. Calendar view: start with a list/agenda; add FullCalendar only when events are real.

---

## Lead (recap) — the entry point

- **List columns (real):** Full name · Title · Company · Phone · Email · Lead status · Owner.
- **Record fields (real):** Lead status · Full name · Company · Title · Website · Description · Owner · Rating · Phone · Email · Address · Number of employees · Annual revenue · Lead source · Industry.
- **Status ladder (real):** `New → Contacted → Nurturing → Unqualified → Converted`. Align the app's `crm_lead_statuses` in data-model.md to this exact ladder.
- **Header actions (real):** **Convert** · Change Owner · Edit.
- **Target app:** the `commercial_pipeline_entries` + `/crm/leads` module (see modules.md). Add the **Convert** quick action → creates Account+Contact+Opportunity.

---

## Captured page-layout details (real — for the detail pages)

Section cards (left column) and related lists (right column) as configured in the org. Rebuild these exact groupings (`screen-anatomy.md` B).

| Object | Detail sections (left) | Related lists (right) | List display modes |
|---|---|---|---|
| **Lead** | About · Get in Touch · Segment · History | Files | Table · Kanban · Split View |
| **Account** | About · Get in Touch · History | Contacts · Opportunities · Cases | same |
| **Contact** | About · Get in Touch · History | Opportunities · Cases · Files | same |
| **Opportunity** | About · Status | (Products/line items · Contact roles · Activities) | Table · **Kanban by Stage** · Split |

**Create/Edit modal groupings (real):** Opportunity → **About** (Name, Account, Close date, Amount, Description, Owner) + **Status** (Stage, Probability %, Forecast category, Next step), footer `Cancel · Save & New · Save`.

**List-view controls menu (real):** `New · Clone · Rename · Sharing Settings · Select Fields to Display · Delete · Reset Column Widths · Table · Kanban · Split View`.

**Convert modal (real):** three Create/Choose sections (Account · Contact · Opportunity) + "Don't create an opportunity" checkbox + `*Record owner` + `*Converted status` (default "Qualified"). See `screen-anatomy.md` D + `data-model-full.md` convert flow.
