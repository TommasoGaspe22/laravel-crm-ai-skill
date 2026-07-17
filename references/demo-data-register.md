# Demo data register (`CRM-SKILL-`)

**Scope:** register of ALL demo records created in the reference org for this research. Update on every creation/edit/deletion. Needed for repeatable tests and for the **final cleanup**.

## Rules (mandatory)
- Every record created has the **`CRM-SKILL-`** prefix in its name/primary label.
- Fake emails only: `@example.test`, `@crm-skill.test`, `@crm-demo.test`.
- Fake phone numbers/URLs/files. No real personal data.
- Don't touch records you didn't create. Don't send real emails/invites.
- Final cleanup: delete all `CRM-SKILL-` records (procedure at the bottom).

## Naming convention
`CRM-SKILL-{Object} {Variant} {NNN}` — e.g. `CRM-SKILL-Lead New 001`, `CRM-SKILL-Company Manufacturing 001`, `CRM-SKILL-Opportunity Invoice Automation`.

## Record register

| ID | Type | Name | Test goal | Status | Relations | Flows tested | Keep? | Notes |
|---|---|---|---|---|---|---|---|---|
| 00Qbl000005Rh9GEAS | Lead | CRM-SKILL-Lead New 001 | list view, record page, conversion | New | Company CRM-SKILL-Manufacturing Co 001 | creation (Tested) | yes | Name=Mario, fake email/phone |
| 00Qbl000005RjCfEAK | Lead | CRM-SKILL-Lead Contacted 002 | Contacted status, inline edit | Contacted | — | creation + set status (Tested) | yes | Name=Luca |
| 00Qbl000005RjHVEA0 | Lead | CRM-SKILL-Lead Unqualif 004 | Unqualified status | Unqualified | — | creation + set status (Tested) | yes | Name=Sara |
| 00Qbl000005RjJ7EAK | Lead | CRM-SKILL-Lead Minimal 005 | minimal data (required only) | New (default) | — | minimal-data creation (Tested) | yes | Last name + Company only |
| *(failed)* | Lead | CRM-SKILL-Lead Nurturing 003 | Nurturing status | — | — | flaky picklist → recreate or set via inline edit | — | retry |

**Observation (Dynamically tested):** creation via the reference org's direct lead-creation URL is reliable; the `Lead status` picklist is intermittent (org-side re-render); blocking required fields = Last name + Company (msg "Fill in this field."). Lead ID prefix `00Q`.

### Accounts created (Dynamically tested)
| ID | Name | Type | Notes |
|---|---|---|---|
| 001bl000008DwzKAAS | CRM-SKILL-Manufacturing Co 001 | Customer | fake phone + website; used for opportunities |
| 001bl000008DvaHAAS | CRM-SKILL-Retail Co 002 | Prospect | used for opportunities |
| 001bl000008DxQlAAK | CRM-SKILL-No-Relations Co 003 | — | "no contacts/opportunities" case |

Account ID prefix `001`. New Account fields (Observed): Account name* (only required), Type (picklist), Phone, Website, Parent account (lookup), addresses.

### ⚠️ Known blocker: creating Opportunities via automation
**Implementation risk / tooling note:** the New Opportunity modal has an **Account lookup** that, when driven via Playwright, destabilizes the form (after interacting with the lookup, the other fields stop responding). Multiple attempts (clicking an option, keyboard ArrowDown+Enter, short/long search, creating from the Account record) failed intermittently.
**Workaround adopted:** create Opportunities via the **Lead conversion flow** (Lead→Account+Contact+Opportunity), which is also a key flow to test. Manual creation is the alternative.
**Still observed:** fields, sections (About/Status), stages (`Qualify→…→Closed Lost`), Forecast category auto-set from Stage — enough for `field-catalog`/`opportunity-record-page`.

### Conversion performed (Dynamically tested, 2026-07-08)
`CRM-SKILL-Lead New 001` (00Qbl000005Rh9GEAS) **converted** → created:
- Account `CRM-SKILL-Manufacturing Co 001` (**duplicate** — the reference org didn't force "Choose"; ID still to retrieve)
- Contact `Mario CRM-SKILL-Lead New 001` (email/phone transferred)
- Opportunity `CRM-SKILL-Manufacturing Co 001` (auto close date 2026-09-30, initial stage)
Lead → status **Converted** (read-only). Exact IDs of the 3 records: **still to retrieve** (query or navigate from the cards).

### Session 2 (2026-07-08) — additional dynamic tests
- **Lead 003 Nurturing exists** (the save had actually succeeded despite the error report): visible in AllOpenLeads.
- **Lead 002 conversion** (`00Qbl000005RjCfEAK` Contacted) performed → 2nd Opportunity created.
- **Opportunities present (2):** `CRM-SKILL-Manufacturing Co 001-` (ID **006bl000005LLplAAG**, now **Propose**) + `CRM-SKILL-Retail Co 002-` (Qualify). Both closing 2026-09-30, Amount empty.
- **Stage transition tested** (Tested): Manufacturing opp Qualify→Propose via Path ("Mark as current stage").
- **Kanban tested** (Tested): "All opportunities" board with columns per stage, counts + € sum, cards with drag handle.
- **Filters panel** (Observed): "Filter by owner", "Add filter", "Remove all" — only on real views, NOT on "Recent".

## Dataset status (updated)
- **Leads: 5** (00Q… Rh9G converted / RjCf converted / RjHV / RjJ7 / +Nurturing003) · **Accounts: 3 + 2 (from conversions, dup)** · **Contacts: 2** (from conversions) · **Opportunities: 2** (from conversions; Propose+Qualify) · Activities: 0.
- **Cleanup note:** includes the duplicate Accounts + Contacts + Opportunities generated by the 2 conversions. Cleanup query by name containing `CRM-SKILL-` on every object.
- Next: create **Activities** (Task/Call/Event/Note) on a record + timeline; test **inline edit** and **row/bulk actions** + **deletion**; set an opp Amount (for Kanban sums); test Kanban drag&drop + Won/Lost closing.

## Planned dataset (to create)

**Leads** (statuses + variants): New · Working · Contacted · Qualified · Unqualified · Unreachable · Converted; future/today/overdue follow-up; high/low priority; with/without owner; with/without activities; with notes/files/email/phone; minimal/full data; long text; potential duplicate.

**Accounts**: no contacts · 1 contact · N contacts · no opps · open/won/lost opps · future/past activities · notes/files · minimal/full data.

**Contacts**: CEO · CFO · CIO · Operations · Admin · IT; with/without activities/email/phone/opportunities; minimal/full.

**Opportunities**: one per real stage (`Qualify → Meet & Present → Propose → Negotiate → Closed Won → Closed Lost`); different/missing amounts; different probabilities; past/future close date; next step; loss reason; with/without activities; linked account+contact; minimal/full.

**Activities**: Task today/overdue/future/completed; planned/completed call; logged email; future/past event; note; + system events (owner/status/priority/stage change, conversion, win/loss, attached file).

## Dataset status
- Leads: 0/28 · Accounts: 0/13 · Contacts: 0/14 · Opportunities: 0/20 · Activities: 0/21. **Total: 0.**

## ✅ CLEANUP DONE (2026-07-08)
All `CRM-SKILL-` records **deleted** (verified: empty lists + converted records "Failed to load"): 4 direct Leads + 2 converted Leads + 4 Accounts + 2 Contacts + Opportunities (cascaded from Account deletion). The pre-existing non-CRM-SKILL lead (a real name, ID 00Qbl000005RNI3EAO) was **preserved** (not created by this research). Demo org clean. **Observed (cleanup):** deleting an Account **cascades** to linked Opportunities+Contacts → confirms the relational model and flags an implementation risk around cascading deletes in Laravel.

## Final cleanup procedure (for future use)
1. In every list view (Lead, Account, Contact, Opportunity, Activity, Case): filter/sort by name containing `CRM-SKILL-`.
2. Select all → bulk **Delete** action (or row by row if bulk isn't available).
3. Verify that cascading deletes (linked opportunities/activities) don't leave orphans; delete any leftovers.
4. Empty the Recycle Bin if accessible, to remove permanently.
5. Confirm in this file: cleanup date, deleted count, leftovers.
**Implementation risk:** bulk deletion and cascading need verification; note the real behavior observed during cleanup (this also doubles as a valid test for `bulk-actions-system.md`).
