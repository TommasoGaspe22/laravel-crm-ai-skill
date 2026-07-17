# Lead conversion system

**Scope:** the "Convert Lead" flow → Account + Contact + Opportunity. Source: **Dynamically tested** (converted `CRM-SKILL-Lead New 001` on 2026-07-08) + **Observed** (modal). Includes field mapping and the transactional Laravel blueprint.

## Index
1. Trigger · 2. Modal (anatomy) · 3. Tested behavior · 4. Field mapping · 5. Laravel · 6. Tests · 7. Priority · 8. Open questions

## 1. Trigger (Observed)
**`Convert`** button in the Lead record page's action header (first button, before `Change Owner`/`Edit`). Available as long as the lead isn't already converted.

## 2. "Convert Lead" modal (Observed)
Three sections, each with a **Create / Choose** radio:
- **Account:** `Create` (`*Account name` prefilled from `Company`) — *or* `Choose` (search "Search for matching accounts", shows N matches → dedup).
- **Contact:** `Create` (name prefilled from First+Last name) — *or* `Choose` (shows "N contact match found").
- **Opportunity:** `Create` (name prefilled = `{Company}`) + checkbox **"Don't create an opportunity upon conversion"** — *or* `Choose`.
- Bottom: `*Record owner` (owner), `*Converted status` (picklist; default in the ladder = the last status "Converted"/"Qualified").
- Footer: `Cancel · Convert`. Helper: *"We'll turn this lead into three new records."*

## 3. Dynamically tested behavior (2026-07-08)
Accepting the defaults (all "Create") and clicking **Convert**:
- **Success:** **"Your lead has been converted"** screen (flag illustration) with **3 cards**: ACCOUNT · CONTACT · OPPORTUNITY, each with key fields (Account: Phone/Website/Address/Owner; Contact: Title/Phone/Email; Opportunity: Name/Close date/Amount/Owner).
- **Records created:** Account `CRM-SKILL-Manufacturing Co 001`, Contact `Mario CRM-SKILL-Lead New 001` (email `mario.rossi@crm-skill.test` **transferred**, phone `3000000001` **transferred**), Opportunity `CRM-SKILL-Manufacturing Co 001` with **auto close date = 2026-09-30** (default ~end of quarter/+N days).
- **Dedup not enforced:** the company already existed as an Account, but with the "Create" default a **duplicate Account** was created (the reference org doesn't force "Choose"; it only offers the search). → **Implementation risk:** without explicit dedup, duplicates get created.
- **Lead post-conversion:** stays accessible (the reference org's detail view), status → Converted; fields become read-only; the `Convert` action is no longer available.
- Post-conversion footer: `New Task` / `Go to Lead`. Helper "Click the opportunity name to move this deal forward".
- **To verify:** whether open Notes/Files/Activities transfer from the Lead to the new records; audit/timeline on the created records; behavior with "Don't create opportunity"; conversion with "Choose" an existing account (dedup).

## 4. Field mapping (Observed/Deduced)
| Lead | → Account | → Contact | → Opportunity |
|---|---|---|---|
| Company | Account name | Account name (rel) | Opportunity name (`{Company}`), Account (rel) |
| First/Last name | — | First/Last name | — |
| Email | — | Email ✔(transferred) | — |
| Phone | Phone | Phone ✔ | — |
| Title | — | Title | — |
| Website/Industry/#Employees/Revenue | Account (matching fields) | — | — |
| Owner | Owner | Owner | Owner |
| — | — | — | Close date (auto), Stage (default 1st) |

## 5. Laravel blueprint (Proposed for Laravel)
`ConvertLead` Action (in a DB transaction):
```
DB::transaction(function() use ($lead, $opts) {
  $account = $opts->accountId ? CrmAccount::find($opts->accountId)
            : CrmAccount::firstOrCreate(['name'=>$lead->company], [...map...]); // optional dedup
  $contact = $opts->contactId ? CrmContact::find($opts->contactId)
            : CrmContact::create([...map from lead, 'account_id'=>$account->id]);
  $opp = null;
  if (!$opts->skipOpportunity) $opp = CrmOpportunity::create([
     'name'=>$opts->oppName ?? $lead->company, 'account_id'=>$account->id,
     'primary_contact_id'=>$contact->id, 'stage'=>'qualify',
     'close_date'=>$opts->closeDate ?? now()->addDays(90), 'owner_id'=>$lead->owner_id]);
  $lead->update(['status'=>'converted','converted_at'=>now(),
     'converted_account_id'=>$account->id,'converted_contact_id'=>$contact->id,'converted_opportunity_id'=>$opp?->id]);
  // transfer polymorphic activities/notes/files from the lead to the new records
  // write a "conversion"-type crm_activities entry on lead+account+opp (audit/timeline)
});
```
- **Dedup (Proposed):** offer a match on `company`/`email` (like "Choose") + a "use existing" option; configurable default (avoids blind duplicates — unlike the reference org).
- **Policy:** `convert` allowed for staff owner/admin; validate owner + converted status.
- **Rollback:** transaction → if any create fails, full rollback (no orphaned records).
- **Audit:** a `crm_activities`/`audit_events` "Lead converted" row linked to the lead, account, and opp.
- **Implementation risk:** transferring polymorphic relations (activities/notes/files) must happen inside the transaction; handle a missing owner; timezone on close_date.

## 6. Recommended tests (Laravel)
Feature: conversion with a new account · with an existing account (dedup) · without an opportunity · minimal data · rollback on error (mock failure) · activity/note transfer · policy (unauthorized user → 403) · post-conversion lead status = converted · idempotency (don't re-convert).

## 7. Priority
- **V1:** conversion creates Account+Contact+Opportunity in a transaction, with a basic dedup option and "no opportunity", audit.
- **V2:** rich dedup UI (multiple matches), full note/file/activity transfer, configurable field mapping.
- **V3:** automatic conversion rules, advanced merge.
- **Do not replicate:** complex batch mass-conversion (as in the reference org), unless genuinely needed.

## 8. Open questions → `open-questions-and-assumptions.md`
Note/File/Activity transfer (to verify); dedup with "Choose" (to verify); `Converted status` values; "Don't create opportunity" behavior.
