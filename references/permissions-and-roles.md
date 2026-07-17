# Permissions & roles

**Scope:** the CRM authorization model. Source: **Observed** ("Change Owner" actions, "Owner Alias" owner; single-user demo org) + **Proposed for Laravel**. Real multi-user behavior **⛔ to verify** (single-user demo).

## Observed / Deduced (reference org)
- Every record has an **Owner**; **Change Owner** action (single and bulk).
- The reference org has a complex sharing model (org-wide defaults, roles, sharing rules, teams) → **Do not replicate** in V1 (too heavy, not inspectable in the demo).
- In the demo everything is visible to the single admin user.

## Proposed Laravel model (Proposed for Laravel)
Reuse the existing `admin`/`sales` roles (`config('crm.user_roles')`, `User::isAdmin()/isSales()`, `role` middleware).

### CRM roles
- **admin:** everything (view/create/update/delete/convert/assign/bulk on all records; shared-view management).
- **sales:** view/create/update/convert on all sales records; **delete** admin-only (sales closes/archives instead); **assign owner** admin-only (or sales→self); bulk admin-only.
- (V3) **manager/team lead:** sees the team; **read-only** for reporting.

### Policy (one per object: `CrmAccountPolicy`, `CrmContactPolicy`, `CrmOpportunityPolicy`, `CrmActivityPolicy`, + `CommercialPipelineEntryPolicy`)
| Ability | admin | sales |
|---|---|---|
| viewAny/view | ✔ | ✔ (all; optionally own-only if `owner_scope`) |
| create | ✔ | ✔ |
| update | ✔ | ✔ |
| delete | ✔ | ✘ (no; close/archive only) |
| convert (lead) | ✔ | ✔ |
| changeOwner/assign | ✔ | self only (config) |
| bulk | ✔ | ✘ |
| manage shared views | ✔ | ✘ |

- **"Assigned to me" scope:** a filter (`owner_id = auth id`), not a hard restriction (unless `owner_scope=true` config limits sales to their own records).
- **Gate/middleware:** CRM routes under `auth`; admin-only actions via `role:admin` or policy; every controller `->authorize()`.
- **Field-level / sensitive data:** V3 (field-level security). V1: PII (email/phone) visible to staff; don't expose it in logs.
- **Access audit:** log view/edit/delete/convert/export (who/when/what) → `audit_events` (`data-model-and-migrations.md`), especially for exports and deletions.

## Priority
- **V1:** per-record owner, per-object policy (admin vs sales), delete admin-only, "assigned to me" scope, sensitive-action audit.
- **V2:** team/manager, configurable owner-scope, view sharing, rich audit.
- **V3:** field-level security, sharing rules, impersonation (only if needed — **do not replicate** by default).
- **Do not replicate:** org-wide defaults/complex sharing rules like the reference org's.

## Open questions → `open-questions-and-assumptions.md`
NV1 (multi-user/sharing not testable in the single-user demo) → verify with Laravel roles + tests; sales scope (all vs own) = a team decision.
