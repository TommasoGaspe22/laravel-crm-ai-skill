# Validation & error system

**Scope:** validations, error messages, error states. Source: **Dynamically tested** (empty-save on New Lead) + **Observed**.

## Tested / Observed
- **Inline required (Tested):** saving with empty required fields → **"Fill in this field."** error under the field, red border, focus on the first invalid field, save blocked, modal stays open. (Lead: Last name, Company.)
- **Observed/Deduced:** email/URL format validated; dates via date-picker; numeric number/currency; picklist with allowed values.
- **To verify:** cross-field validation, max lengths, format messages, server-side errors (custom validation rule → not testable in the demo org without touching config: **⛔** document from prior knowledge), save errors (error toast), concurrency conflicts.

## Proposed Laravel design (Proposed for Laravel)
- **FormRequest** per object/action: `required`, `email`, `url`, `numeric`, `date` (≥ today where relevant), `in:{picklist}`, `max:`; localized messages ("Fill in this field." → `The :attribute field is required.`).
- **Inline errors** under the field (mirrors the reference org); a summary at the top of the modal if >1.
- **Server/action errors:** error toast (`x-crm.toast`); no 500s → catch + a readable message.
- **Concurrency:** optimistic lock (`updated_at`) → 409 with a "record modified by someone else" message.
- **Business rules** (mirror the reference org's validation rules): in `FormRequest`/Action (e.g. don't close an opportunity without a loss reason) — explicit, tested domain logic.
- **Implementation risk:** ALWAYS validate server-side (never trust the client); sanitization; messages that don't leak sensitive details.

## Priority
- **V1:** required + type + inline picklist, localized messages, error toast, no 500s. **V2:** cross-field, optimistic lock, rich business rules. **V3:** configurable (validation-rule-like) rules.

## Open questions → `open-questions-and-assumptions.md`
Q4: complete validation per object; NV2 (server-side validation rule/Flow not inspectable → infer).
