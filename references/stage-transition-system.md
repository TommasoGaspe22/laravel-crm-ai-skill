# Stage / status transition system (Path)

**Scope:** the **Path** component and status/stage transitions on Lead (Lead status) and Opportunity (Stage). Source: **Dynamically tested** (2026-07-08: moved an opportunity Qualify → Propose via Path) + **Observed** (Lead path).

## Index
1. Path (anatomy) · 2. Transition (tested) · 3. Rules · 4. Laravel · 5. Priority · 6. Open questions

## 1. Path — anatomy (Observed/Tested)
- Horizontal **chevron** bar: completed stages (green/tinted), current (dark), future (grey).
- **Lead** (Lead status): `New → Contacted → Nurturing → Unqualified → Converted`.
- **Opportunity** (Stage): `Qualify → Meet & Present → Propose → Negotiate → Closed` (the **"Closed"** chevron groups Closed Won/Lost; selecting it opens the closed-status choice — **to verify**).
- **"Guidance for Success"** panel below the Path: guidance text for the selected stage (Observed on Opp Propose: *"Make the offer. Does the quote cover…"*). → in the target CRM, per-stage guidance text (config), useful for sales onboarding.
- Primary button on the right, **depends on the selection**:
  - Current stage selected → **"Mark Status as Complete"** (advances to the next one).
  - A different stage selected → **"Mark as Current Stage"** (sets it as current).

## 2. Dynamically tested transition
- Clicking the **"Propose"** chevron → the stage gets selected (shows guidance) and the button becomes **"Mark as Current Stage"**.
- Clicking the button → **Stage updated to Propose** (verified: the record page highlights the current stage = Propose **and** the list view shows `Stage = Propose` **and** Kanban places the card in the Propose column). No required field for this transition.
- **Closing (Dynamically tested):** selecting the **"Closed"** chevron turns the button into **"Select Stage"** → opens the **"Close this opportunity"** modal with a **required "Stage" field** = picklist *"Select a closed stage…"* (**Closed Won / Closed Lost**) → `Cancel · Save`. **The trial does NOT show a "Loss reason" field** (it would be a custom field). → in Laravel `loss_reason` stays optional/config, required only if the team wants it on "Lost".

## 3. Rules (Observed/Deduced)
- Selection ≠ commit: the button is needed to confirm (two steps). → avoids accidental changes.
- Changing stage also updates **Probability** and **Forecast category** (the reference org ties them to the stage) — **to verify** the exact values.
- Closing (Won/Lost) is terminal; the opportunity ends up closed (`is_closed`/`is_won`).

## 4. Laravel blueprint (Proposed for Laravel)
- **Component** `x-crm.path` (see `component-catalog.md`): props `:steps`, `current`, `:guidance`, `:onSelect`. Two-step: select → confirm.
- **Action** `ChangeStage(opportunity, toStage, extra?)`:
  - validates `toStage ∈ config('crm_opportunity_stages')`;
  - if `toStage` is terminal (won/lost) → requires fields (e.g. `loss_reason` for lost) via a modal;
  - updates `stage`, derives `probability` (stage→default-probability map, manual override allowed), `is_closed`/`is_won`;
  - writes a "stage change" `crm_activities` entry (timeline/audit): `{from, to, user, at}`;
  - `update` policy; transaction; toast.
- **Lead status** works the same way: `ChangeLeadStatus` Action; the "Converted" status is reached **only** via conversion (not manually selectable) — see `lead-conversion-system.md`.
- **Implementation risk:** transition validation (allow skipping stages? the reference org allows it); required fields on closing; audit; concurrency (optimistic lock).

## 5. Priority
- **V1:** Lead + Opportunity Path with two-step commit, stage change with timeline audit, Won/Lost closing with loss reason.
- **V2:** per-stage guidance (config text), auto-probability per stage, required-field validation per transition.
- **V3:** configurable transition rules (blocks, approvals).
- **Do not replicate:** complex approval processes, custom multi-object paths.

## 6. Open questions → `open-questions-and-assumptions.md`
Won/Lost closing modal + required fields; stage→probability/forecast mapping; whether the reference org allows skipping stages; required fields per transition.
