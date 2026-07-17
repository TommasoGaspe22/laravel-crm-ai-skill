# Field catalog — per-object field specification

**Scope:** every field observed on the reference org's objects → DB type, UI type, requiredness, validations, picklists, Laravel mapping. Source: New/Edit modals and record page (**Observed**), validation testing (**Dynamically tested**). The demo org is nearly empty → some secondary picklist values are **To verify**.

Columns: Field (label) · UI · Laravel/DB type · Req. · Notes/picklist · Priority.

Requiredness legend: **R!** = blocking required (tested) · **R** = marked required (`*`) but has a default → doesn't block · `-` optional.

---

## Lead  *(Observed: New modal; Dynamically tested: empty-save validation)*

Layout sections (Observed): **About · Get in Touch · Segment**. Blocking required fields (Dynamically tested, msg "Fill in this field."): **Last name**, **Company**. `Lead status` is `*` but has a default → doesn't block.

| Field | UI | Laravel/DB | Req. | Notes | Prio |
|---|---|---|---|---|---|
| Salutation | picklist | `string(20)` nullable | - | Mr./Mrs./… (values to verify) | V2 |
| First name | text | `string(80)` | - | | V1 |
| Last name | text | `string(80)` | **R!** | | V1 |
| Full name | compound (computed) | accessor `full_name` | R | display; not a column | V1 |
| Company | text | `string(200)` | **R!** | becomes the Account on conversion | V1 |
| Lead status | picklist | `enum status` | R(default) | `New·Contacted·Nurturing·Unqualified·Converted` (Observed on Path) | V1 |
| Title | text | `string(120)` | - | contact's job title | V1 |
| Website | url | `string(190)` | - | | V2 |
| Description | textarea | `text` | - | | V1 |
| Lead owner | lookup(User) | `owner_id` FK | R(default=me) | | V1 |
| Rating | picklist | `enum rating` | - | proxy for priority | V1 |
| Phone | phone | `string(40)` | - | | V1 |
| Email | email | `string(190)` | - | validates email format | V1 |
| Address | compound address | see below | - | Country·Street·ZIP·City·State/Province | V2 |
| ↳ Country/Street/ZIP/City/Province | text | `string` each | - | | V2 |
| Number of employees | number | `unsignedInteger` | - | | V2 |
| Annual revenue | currency | `decimal(15,2)` | - | | V2 |
| Lead source | picklist | `enum lead_source` | - | values to verify (Web, Referral, …) | V1 |
| Industry | picklist | `string/enum industry` | - | values to verify | V1 |

**Proposed for Laravel:** Lead fields flow into `commercial_pipeline_entries` (existing) + additive columns (`data-model.md`). `Rating` maps to priority; `Lead source` → `lead_source`; `Industry` → `business_area`.
**Implementation risk:** `Full name` is compound (salutation+first+last) → in Laravel keep separate fields + an accessor, not a single column.
**To verify:** exact picklist values for `Salutation`, `Lead source`, `Industry` (empty org) → extract by opening each picklist in the modal, or from real records.

---

## Account  *(Observed: record page; New modal to verify in detail)*
Sections: About · Get in Touch · History. Fields (Observed): Account name* · Website · Type (picklist) · Description · Parent account (self lookup) · Owner · Phone · Billing address · Shipping address. **To complete** with New-modal capture (required fields, Type picklist).

## Contact  *(Observed: record page)*
Fields: Full name* · Account name (lookup) · Title · Reports to (self lookup) · Description · Owner · Phone · Email · Mailing address. **To complete** with New-modal capture.

## Opportunity  *(Observed: New modal — complete)*
Sections: **About · Status**. Fields: *Opportunity name · *Account name (lookup) · *Close date (date) · Amount (currency) · Description (textarea) · Owner · *Stage (picklist) · Probability (%) (number) · *Forecast category (picklist) · Next step (text). Stages (Observed): `Qualify·Meet & Present·Propose·Negotiate·Closed Won·Closed Lost`. **To verify:** `Forecast category` values; whether `Loss reason` appears on Closed Lost.

## Task / Event (Activity)  *(To verify — New modal not yet opened)*
Expected (standard): Task → Subject · Due date · Status · Priority · Assigned to · Name(contact) · Related to · Comments. Event → Subject · Start · End · Location · Related to. **To capture dynamically.**

## Case  *(To verify — object may not be active in the trial)*
To explore: tab availability, fields (Subject, Status, Priority, Origin, Account/Contact). If not active → ⛔ + future procedure.

---

## Real picklist values (Dynamically tested, 2026-07-08)
Extracted by opening the picklists in the New modals (map to `config('crm.crm_*')`):
- **Lead — Lead status:** `New · Contacted · Nurturing · Qualified · Unqualified` (+ `Converted` only via conversion, not selectable). *(note: the real ladder includes `Qualified`; the visual Path showed a subset.)*
- **Lead — Rating:** `Urgent · Moderately urgent · Not urgent` *(NOT Hot/Warm/Cold — correction).* → maps to priority.
- **Lead — Salutation:** `Mr. · Miss · Mrs. · Dr. · Prof. · Mx.`
- **Lead — Lead source:** `Advertising · Employee Referral · External Referral · Partner · Public Relations · Seminar - Internal · Seminar - Partner · Trade Show · Web · Word of Mouth · Other`
- **Lead — Industry:** Agriculture, Apparel, Banking, Biotechnology, Chemicals, Communications, Construction, Consulting, Education, Electronics, Energy, Engineering, Entertainment, Environmental, Finance, Food & Beverage, Government, Healthcare, Hospitality, Insurance, Machinery, Manufacturing, Media, Not For Profit, Recreation, Retail, Shipping, Technology, Telecommunications, Transportation, Utilities, Other. *(shared Lead/Account as `industry`)*
- **Account — Type:** `Analyst · Competitor · Customer · Integrator · Investor · Partner · Press · Prospect · Reseller · Other`
- **Opportunity — Stage:** `Qualify · Meet & Present · Propose · Negotiate · Closed Won · Closed Lost`
- **Opportunity — Forecast category:** `Omitted · Pipeline · Best Case · Commit · Closed`
- **Opportunity closing:** "Close this opportunity" modal → required field **Stage** ("Select a closed stage" = Closed Won/Lost). **No "Loss reason" field** in the trial (would be custom → the proposed Laravel `loss_reason` stays optional/config).
- **Case:** object **available** in the org (grid present) → fields **to verify** (New Case).

## Remaining catalog TODO
- [ ] Contact: New modal (fields/required) — Observed from the record; New modal to verify
- [ ] Task/Event/Email: full composer fields (Task: Subject+Due date Observed)
- [ ] Case: fields (New Case)
