# Contact record page

**Source:** Observed (real record + creation via conversion). Base: `record-page-system.md`. Fields: `field-catalog.md` (Contact).

- **Header:** `Contact` / name (e.g. "Mario CRM-SKILL-Lead New 001"). Actions: **New Opportunity · Edit · ▾**. No Path.
- **Sections (Observed):** `About` (Full name, Account name [lookup], Title, Reports to [self lookup], Description, Contact owner) · `Get in Touch` (Phone, Email, Mailing address) · `History`.
- **Activity:** composer + timeline (a contact is the "who"/Name on activities).
- **Related list (Observed):** **Opportunities · Cases · Files** (+ Activities). The contact's role on opportunities = the "Contact Roles" related list on the Opportunity side.
- **Creation via conversion (Tested):** email/phone transferred from the Lead; linked to the created Account.
- **Laravel:** `CrmContact` `belongsTo account`, self `reports_to`, `hasMany opportunities` (as primary contact) + `belongsToMany opportunities` via roles (`crm_opportunity_contact_roles`, V2).
- **To verify:** full New Contact fields; "Reports to" (hierarchy); contact roles on opportunities.
