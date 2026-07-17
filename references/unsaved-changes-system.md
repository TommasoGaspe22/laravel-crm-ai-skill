# Unsaved changes system

**Scope:** unsaved-change detection + warning. Source: **Observed** ("Cancel/Save" bar appears while editing; close warning **to verify**).

## Observed / Deduced
- Inline edit / modal: editing ≥1 field shows a **`Cancel · Save`** bar ("dirty" state).
- **Deduced/To verify:** closing the modal (X/Cancel) with pending edits → warning "You'll lose your changes?" (the reference org's standard behavior). To be tested.

## Proposed Laravel design (Proposed for Laravel)
- **Alpine `dirty` tracking:** every form/inline-edit keeps a `dirty` flag (comparing initial↔current values).
- **Close warning:** on close/X/navigation while `dirty` → `confirm()` / `x-crm.confirm-dialog` dialog ("You have unsaved changes. Leave without saving?"). `beforeunload` for browser navigation.
- **Save bar:** `Cancel` (restores initial values, `dirty=false`) · `Save` (submit).
- **Implementation risk:** false positives (normalize values before comparing); don't annoyingly block the user; dialog accessibility.

## Priority
- **V1:** dirty tracking + Cancel/Save bar + warning on closing a modal with pending edits. **V2:** `beforeunload`, draft autosave. **V3:** draft recovery.

## Open questions → `open-questions-and-assumptions.md`
Q5: unsaved-changes warning (text, trigger) — to test (open modal, edit, close).
