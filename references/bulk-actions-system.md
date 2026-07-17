# Bulk actions system

**Scope:** mass actions on a multi-selection in list views. Source: **Observed** (select-all + row checkboxes; object actions like "Change Owner", "Send Email"); bulk details **to verify** dynamically.

## Observed / Dynamically tested (2026-07-08)
- Header **select-all checkbox** + per-row checkbox → multi-selection. **Tested:** selected 2 rows → **"2 items selected"** appears below the title.
- With rows selected, the **header object actions become mass actions** (Tested, Lead): **Change Owner · Send Email · Add to Campaign** (+ New/Import). **Edit List** → mass inline edit.
- **Row action ▾ (Tested, Lead):** `Edit · Delete · Change Owner · Edit Labels`.
- **To verify:** selection limits (e.g. 200), bulk confirmation, partial outcome, mass inline cell edit, where mass "Delete" lives (probably in the ▾ overflow).

## Proposed Laravel design (Proposed for Laravel)
- **UI:** `x-crm.bulk-action-bar` that appears when `selectedIds.length > 0` (Alpine reactive), with actions: **change status/stage**, **assign owner**, **flag/unflag**, **delete** (admin), **export selected**, **mass edit** (V2).
- **Endpoint** `PATCH /crm/{obj}/bulk` with `ids[] + action + payload`: validates the `ids` exist, **authorizes every record** (policy), runs in a **transaction**, returns the outcome (succeeded/failed) + a summary toast.
- **Limits:** selection cap (e.g. 200) → beyond that, chunk/queue job (V2). Prevent timeouts on SQLite.
- **Implementation risk:** per-record authorization (not just global); partial outcome (report of failed rows); idempotency; audit for every changed record.

## Priority
- **V1:** select-all + bulk change status/assign owner/flag/delete/export, in a transaction, with confirmation + toast. **V2:** mass inline edit, bulk email sending, chunk/queue for large selections. **V3:** custom bulk actions, campaigns.
- **Do not replicate (V1):** real bulk email sending, add to campaign.

## Open questions → `open-questions-and-assumptions.md`
Q6: contextual bulk bar, selection limits, mass inline edit, partial outcome — to test.
