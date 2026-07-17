# Inline edit system

**Scope:** inline field editing on the record page and list view. Source: **Observed/Tested** (2026-07-08: the pencil opens inline edit with a "Cancel/Save" bar; save-selector calibration in progress).

## Observed / Dynamically tested (2026-07-08)
- Every field row on the record page shows a **pencil ✎** (title "Edit {Field}") on hover/focus.
- Clicking ✎ → the field becomes an **editable input** in place (text/number/picklist/lookup depending on type). E.g. `Account name` becomes editable with an **X** to clear the lookup.
- Editing shows a **save bar** at the bottom with **`Cancel · Save`** (batched: multiple edits, one Save). → also an **unsaved-changes** mechanism (`unsaved-changes-system.md`).
- **Tested:** inline-edited an opportunity's **Amount** to **€30,000** → saved and persisted (visible in the record, list, and Kanban/split). **Multi-field batch** edit (Amount + Description) via the Save bar.
- On the list view: **"Edit List"** mode (toolbar) enables inline cell editing (mass inline edit). **To verify** in detail.
- **To verify:** inline validation (required field cleared), non-editable fields (formula/rollup), inline picklist/lookup editing, conflicts/concurrency.

## Proposed Laravel design (Proposed for Laravel)
- **Component** `x-crm.detail-section` with editable field rows (Alpine `inlineEdit`): click ✎ → swap value↔input; `dirty` state per row; floating `Cancel/Save` bar when ≥1 field is dirty.
- **Endpoint** `PATCH /crm/{obj}/{id}` (partial, only changed fields) with a `FormRequest` (same rules as the full form, but `sometimes`), `update` policy, JSON response (updated value + any per-field errors) to update the UI without a reload.
- **Types:** text/number/date/currency inline; picklist → combobox; lookup → async search (dedicated endpoint).
- **Validation:** per-field errors shown under the cell; if a required field is cleared → save is blocked.
- **Implementation risk:** concurrency (optimistic lock via `updated_at`); non-editable computed fields; XSS on text; field-level permissions → V2/V3.
- **Accessibility:** ✎ is a `button` with `aria-label`; the input receives focus; Esc cancels; the save bar is keyboard-reachable.

## Priority
- **V1:** inline edit on the record page for simple fields (text/number/date/picklist), batch save bar, basic validation.
- **V2:** inline edit on the list view ("Edit List"), inline lookup, mass edit on multi-selection.
- **V3:** field-level security, conditional editing.
- **Do not replicate:** inline edit on computed/formula fields.

## Open questions → `open-questions-and-assumptions.md`
Q3 inline edit (batch/validation/non-editable fields/list mass edit) — to test in detail.
