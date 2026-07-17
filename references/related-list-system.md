# Related list system

**Scope:** related lists on record pages. Source: **Observed** (real related lists per object).

## Observed
- Card in the right/related column with **title + (count)**, a few compact rows, a **"View All"** link, and **quick-create** (e.g. "New Contact"/"New Opportunity" from the Account header).
- Related lists per object: **Account** → Contacts · Opportunities · Cases (+ Activities, Files). **Contact** → Opportunities · Cases · Files. **Opportunity** → Contact Roles · Files · Invoices. **Lead** → Files.
- **Tested:** creating an opportunity from the Account context (related/action) pre-links the account (more reliable than the standalone form).
- **To verify:** inline actions on related rows (edit/delete), related-list pagination, columns shown, "View All" → filtered list view.

## Proposed Laravel design (Proposed for Laravel)
- **Component** `x-crm.related-list`: props `title`, `:rows`, `:columns`, `createRoute`, `viewAllRoute`. Shows the first N rows + count + "View All" (→ the related object's list view filtered on the parent) + quick-create (modal with the parent pre-filled).
- **Data:** Eloquent relations (`hasMany`/`belongsToMany`) with `withCount` for badges; eager-load to avoid N+1; limit + "view all".
- **Quick-create:** reuse `x-crm.record-modal` with the foreign key preset (like "New Opportunity" from Account).
- **Polymorphic:** Activities/Files/Notes as polymorphic related lists (`what` morph) → a generic related list for any object.
- **Implementation risk:** N+1 on many related lists; permissions (show only visible records); expensive counts (cache/withCount).

## Priority
- **V1:** main related lists (Account→Contacts/Opportunities; Opportunity→Contact Roles; +Activities/Files) with count, first rows, quick-create, "view all".
- **V2:** inline row actions, related-list pagination, configurable columns.
- **V3:** custom related lists, related-of-related.

## Open questions → `open-questions-and-assumptions.md`
Inline actions/pagination on related lists; related-list columns; "Contact Roles" (M:N opportunity-contact).
