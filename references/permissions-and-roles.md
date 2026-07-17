# Permissions & roles

**Scope:** modello autorizzazioni CRM. Fonte: **Osservato** (azioni "Cambia titolare", owner "Alias titolare"; org demo mono-utente) + **Proposto per Laravel**. Multi-utente reale **⛔ Da verificare** (demo con 1 utente).

## Osservato / Dedotto (org di riferimento)
- Ogni record ha un **Titolare (Owner)**; azione **Cambia titolare** (singola e bulk).
- L'org di riferimento ha sharing model complesso (org-wide defaults, ruoli, sharing rules, team) → **Non replicare** in V1 (troppo pesante, non ispezionabile nella demo).
- Nella demo tutto è visibile all'unico utente admin.

## Modello Laravel proposto (Proposto per Laravel)
Riusa i ruoli esistenti `admin`/`sales` (`config('crm.user_roles')`, `User::isAdmin()/isSales()`, middleware `role`).

### Ruoli CRM
- **admin:** tutto (view/create/update/delete/convert/assign/bulk su tutti i record; gestione viste condivise).
- **sales:** view/create/update/convert su tutti i record commerciali; **delete** solo admin (sales chiude/archivia); **assign owner** admin (o sales→se stesso); bulk admin.
- (V3) **manager/team lead:** vede il team; **read-only** per reporting.

### Policy (una per oggetto: `CrmAccountPolicy`, `CrmContactPolicy`, `CrmOpportunityPolicy`, `CrmActivityPolicy`, + `CommercialPipelineEntryPolicy`)
| Abilità | admin | sales |
|---|---|---|
| viewAny/view | ✔ | ✔ (tutti; opz. solo propri se `owner_scope`) |
| create | ✔ | ✔ |
| update | ✔ | ✔ |
| delete | ✔ | ✘ (no; solo close/archive) |
| convert (lead) | ✔ | ✔ |
| changeOwner/assign | ✔ | self only (config) |
| bulk | ✔ | ✘ |
| manage shared views | ✔ | ✘ |

- **Scope "Assegnati a me":** filtro (`owner_id = auth id`), non restrizione hard (salvo config `owner_scope=true` per limitare sales ai propri record).
- **Gate/middleware:** route CRM sotto `auth`; azioni admin-only via `role:admin` o policy; ogni controller `->authorize()`.
- **Field-level / dati sensibili:** V3 (field-level security). V1: PII (email/telefono) visibile a staff; non esporre in log.
- **Audit accessi:** log di view/edit/delete/convert/export (chi/quando/cosa) → `audit_events` (`data-model-and-migrations.md`), soprattutto per export ed eliminazioni.

## Priorità
- **V1:** owner per record, policy per oggetto (admin vs sales), delete admin-only, scope "assegnati a me", audit azioni sensibili.
- **V2:** team/manager, owner-scope configurabile, condivisione viste, audit ricco.
- **V3:** field-level security, sharing rules, impersonation (solo se necessario — **Non replicare** di default).
- **Non replicare:** org-wide defaults/sharing rules complesse come nell'org di riferimento.

## Open questions → `open-questions-and-assumptions.md`
NV1 (multi-utente/sharing non testabile in demo mono-utente) → verificare con ruoli Laravel + test; scope sales (tutti vs propri) = decisione team.
