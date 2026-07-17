# Lead record page

**Source:** Observed/Tested (real record + conversion). Common base: `record-page-system.md`. Fields: `field-catalog.md` (Lead).

- **Header:** `Lead` / name (e.g. "Ms. Jane Doe"). Actions: **Convert · Change Owner · Edit · ▾**.
- **Path (Lead status):** `New → Contacted → Nurturing → Unqualified → Converted` (two-step commit, `stage-transition-system.md`). `Converted` reachable only via conversion.
- **Sections (left, Observed):** `About` (Lead status, Full name, Company, Title, Website, Description, Lead owner, Rating) · `Get in Touch` (Phone, Email, Address) · `Segment` (Number of employees, Annual revenue, Lead source, Industry) · `History`.
- **Activity:** composer + timeline (`activity-and-timeline-system.md`).
- **Related list (right):** `Files`. (Before conversion the Lead has few relations.)
- **Key action — Convert** (`lead-conversion-system.md`): creates Account+Contact+Opportunity.
- **Post-conversion (Tested):** status → Converted, fields read-only, `Convert` no longer available.
- **Laravel:** on `commercial_pipeline_entries` + `/crm/leads/{id}` (existing Lead module, `modules.md`) extended with a status Path + Convert action. Inline edit + timeline + quick actions.
- **To verify:** transferring notes/files/activities on conversion; History/audit.
