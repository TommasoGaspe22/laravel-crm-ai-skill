# Account record page

**Source:** Observed/Tested (real record + deletion). Base: `record-page-system.md`. Fields: `field-catalog.md` (Account) + `data-model-and-migrations.md`.

- **Header:** `Account` / name. Actions: **org-chart** icon · **New Contact · New Opportunity · Edit · ▾**. No Path (Account has no stages).
- **Sections (Observed):** `About` (Account name, Website, Type, Description, Parent account [self lookup], Account owner) · `Get in Touch` (Phone, Billing address, Shipping address) · `History`.
- **Activity:** composer + timeline.
- **Related list (Observed):** **Contacts · Opportunities · Cases** (+ Activities, Files). Quick-create "New Contact"/"New Opportunity" live here (Tested: creating an opp from the account context works more reliably than the standalone form).
- **Deletion (Tested):** `Delete` action → "Delete this account?" dialog (title "Delete Account") → confirm → redirect to the list. Cascade to Contacts/Opportunities **to verify**.
- **Laravel:** `CrmAccount` (hub) → `hasMany contacts/opportunities/activities`; `belongsTo parent`. Related list with counts + quick-create. See `data-model-full.md`.
- **To verify:** full New Account fields (Type picklist values); deletion cascade; parent-account hierarchy (org chart).
