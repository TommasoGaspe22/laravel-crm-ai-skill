# Form & modal system

**Scope:** anatomy and behavior of create/edit/convert/confirm modals, with a Laravel proposal (`x-crm.record-modal` component + Form Request). Source: **Observed** (New Lead, New Opportunity, Convert) + **Dynamically tested** (empty-save validation on New Lead).

## Index
1. Common modal anatomy · 2. New/Edit record · 3. Validation & errors · 4. Convert · 5. Confirm/Delete · 6. Unsaved changes · 7. Laravel · 8. Priority · 9. Open questions

## 1. Common anatomy (Observed)
- **Trigger:** `New`/`Edit`/row action/record action button.
- Dark modal **overlay**, centered content, `X` top-right.
- **Header:** title `Create {Object}` / `Edit {Object}`. Note `* = Required Information` top-right.
- **Body:** **sections** with grey headers grouping the fields; internal scroll if long; 2-column layout on desktop.
- **Footer:** `Cancel` · (`Save & New`) · `Save` (primary). On conversion: `Cancel · Convert`.
- **Fields:** text, textarea, date (calendar icon), number, currency, **picklist** (combobox ▾ with `role=listbox/option`), **lookup** (magnifier search, "N matches"), checkbox, owner (User lookup).

## 2. New / Edit — per-object specifics

### New Lead (Observed + Dynamically tested)
- Sections: **About · Get in Touch · Segment**.
- Fields: see `field-catalog.md` (Lead). Compound **Full name** (Salutation+First+Last).
- **Blocking required fields (Tested):** `Last name`, `Company`. `Lead status` is `*` but has a default → doesn't block.
- Picklists: `Lead status`, `Rating`, `Lead source`, `Industry`.

### New Opportunity (Observed)
- Sections: **About** (Name*, Account* lookup, Close date*, Amount, Description, Owner) · **Status** (Stage*, Probability %, Forecast category*, Next step).
- Footer: `Cancel · Save & New · Save`.

### New Account / Contact (To verify — capture the New modal)
- Account: Account name* + Type (picklist) + Website + Phone + addresses + Parent account (self lookup).
- Contact: Full name* + Account name (lookup) + Title + Email + Phone + Reports To (self lookup).

## 3. Validation & errors (Dynamically tested — New Lead)
- **Empty-save:** required fields show an inline error **"Fill in this field."** under the field, red border, focus on the first invalid one; the save is blocked, the modal stays open.
- **Observed/Deduced:** email format validated; dates via date-picker; numeric number/currency.
- **To verify:** cross-field validation, max lengths, email/URL format messages, server-side errors (validation rule) → not testable without custom fields/rules.

## 4. Convert modal (Observed — Lead → 3 records)
Title **"Convert Lead"**. Three sections with **Create / Choose** radios: Account · Contact · Opportunity (+ "Don't create an opportunity" checkbox). Bottom: `*Record owner`, `*Converted status` (default "Qualified"). Detail in `lead-conversion-system.md`.

## 5. Confirm / Delete (To verify)
Expected: confirmation dialog with a message + `Cancel`/`Delete`. To test dynamically (by deleting a demo record) → document the text, cascade behavior, toast.

## 6. Unsaved changes (To verify)
Expected: a warning when closing a modal with unsaved changes. To test (open, edit, close with X).

## 7. Proposed Laravel design
- **Component** `x-crm.record-modal`: props `title`, `:sections`, `:footer`, a field slot; Alpine `modal` (focus-trap, Esc, backdrop). Fields via `x-crm.field.*` (`field-catalog.md`/`component-catalog.md`).
- **Validation:** `FormRequest` per object (e.g. `StoreLeadRequest`) → rules `required`, `email`, `numeric`, `date`, `in:{picklist}`; localized messages ("Fill in this field." → `:attribute is required`). Show inline errors under the field (mirrors the reference org).
- **Actions:** `Save` (store/update), `Save & New` (redirect to create), `Cancel` (close). Toast on success (`session('status')`).
- **Unsaved changes:** Alpine `dirty` flag + `beforeunload`/confirmation on close.
- **Implementation risk:** compound Name → separate fields in Laravel; async lookup → dedicated endpoint with debounce + policy; picklist → shared config/enum with validation.

## 8. Priority
- **V1:** New/Edit Lead/Account/Contact/Opportunity, inline required validation, Convert, Confirm/Delete, toast.
- **V2:** Save & New, unsaved-changes warning, rich async lookup, collapsible sections.
- **V3:** multiple record types, advanced dependent/conditional fields.
- **Do not replicate:** complex server-driven dynamic forms, per-profile page-layout assignment.

## 9. Open questions (→ `open-questions-and-assumptions.md`)
Q4 complete per-object validation · Q5 unsaved-changes · New Account/Contact modal · Confirm/Delete text+cascade · picklist values (Type, Forecast category, Source, Industry).
