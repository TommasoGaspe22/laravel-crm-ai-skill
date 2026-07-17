# Activity & timeline system

**Scope:** attività (Task, Chiamata, Evento, Email, Nota), composer e timeline sui record. Fonte: **Testato dinamicamente** (2026-07-08: "Registra una chiamata" e "Nuova operazione" su un'opportunità → comparse in timeline) + **Osservato** (composer, filtri timeline).

## Indice
1. Composer · 2. Tipi attività · 3. Timeline · 4. Modello Laravel · 5. Priorità · 6. Open questions

## 1. Composer (Osservato/Testato)
Sul record page, colonna centrale, riga di pulsanti: **Email · Nuova operazione (Task) · Registra una chiamata (Call) · Nuovo evento (Event)** (ognuno con ▾). Aprono una **modale/composer**.

### "Registra una chiamata" (Testato — campi reali)
| Campo | UI | Note |
|---|---|---|
| Oggetto | text con valori predefiniti (lente) | default "Chiamata"; picklist di oggetti frequenti |
| Commenti | textarea | tip "Ctrl+." per testo veloce |
| Nome | lookup (Lead/Referente) | "chi" — la persona correlata |
| Correlato a | lookup (Account/Opportunità/…) | "cosa" — pre-compilato col record corrente |
| — | footer | `Annulla · Salva` |

### "Nuova operazione" (Task) (Testato — campi)
Oggetto · **Scadenza** (data) · (Nome, Correlato a, Priorità, Stato, Assegnato a — standard). → il modello Task ha `who` (Nome) + `what` (Correlato a) polimorfici.
### Evento / Email: **Da verificare** in dettaglio (Evento: Inizio/Fine/Luogo/Invitati; Email: destinatari/oggetto/corpo).

## 2. Tipi attività (Osservato/Dedotto)
`task` (operazione) · `call` (chiamata) · `event` (evento) · `email` (email registrata) · `note` (nota, oggetto separato "Note"). Ogni attività ha **who** (Lead/Contact) e **what** (Account/Opportunity/Lead) → relazioni polimorfiche.

## 3. Timeline (Testato)
- Pannello centrale sotto il composer. Toggle "Mostra solo le attività con approfondimenti". **Filtro:** "Entro 2 mesi • Tutte le attività • Tutti i tipi" + gear (personalizzabile).
- Gruppi: **"Imminenti e scadute"** (future/scadute) + passate per mese. `Aggiorna · Espandi tutto · Mostra tutte le attività`.
- **Testato:** dopo "Registra una chiamata", in timeline compare un item **"Chiamata"** (icona tipo + oggetto + menu "Mostra più azioni"). Item con oggetto vuoto → "Dettagli per [Nessun oggetto]".
- Empty state: "Nessuna attività da mostrare. Inizia inviando un'email, pianificando un'operazione…".

## 4. Modello Laravel (Proposto per Laravel)
Tabella unica `crm_activities` (sostituisce la timeline *derivata* dei fase precedenti):
```
id, type(task|call|event|email|note), subject, body/comments,
status(open|completed|nullable), priority(nullable),
due_at(nullable), start_at(nullable), end_at(nullable),   // task/event
who_type+who_id (morph: lead/contact, nullable),          // "Nome"
what_type+what_id (morph: account/opportunity/lead),      // "Correlato a"
owner_id(FK users), created_by, timestamps
```
Model `CrmActivity`: `morphTo who`, `morphTo what`, `belongsTo owner`. Scopes: `upcoming()`, `overdue()`, `completed()`, `ofType()`, `forRecord($model)`.
- **Timeline** su un record = `CrmActivity::forRecord($record)->orderByDesc(coalesce(due_at,start_at,created_at))` raggruppata per mese, con sezione "imminenti/scadute" in cima. Componente `x-crm.activity-panel` + `x-crm.timeline` (vedi `component-catalog.md`).
- **Composer** = Action `LogActivity` (validazione: subject required per task; date valide; who/what coerenti). Toast su salvataggio.
- **Eventi di sistema vs attività:** distinguere audit (cambio owner/stato/fase, conversione) → tabella/righe `audit_events` **oppure** `crm_activities` di tipo `system` con flag. In timeline si mostrano entrambi (attività + eventi), come nell'org di riferimento.
- **Follow-up:** un follow-up = un `task`/`call` con `due_at` futuro; bucket (overdue/today/upcoming) via il `CommercialPipelineActionService` esistente / query scope.
- **Rischio implementativo:** relazioni polimorfiche who/what; timezone su due_at/start_at; N+1 in timeline (eager-load who/what/owner); permessi (vedi `permissions-and-roles.md`).

## 5. Priorità
- **V1:** `crm_activities` con task/call/note, composer (subject+comments+who+what+due), timeline su record con "imminenti/scadute", completamento task, eventi di sistema in timeline (conversione, cambio fase/owner/stato).
- **V2:** eventi/calendario (FullCalendar), email registrata, reminder/notifiche, filtri timeline avanzati, "Ctrl+." quick text.
- **V3:** integrazione email/telefonia, Einstein-like insights.
- **Non replicare:** conversation insights AI, integrazioni proprietarie.

## 6. Open questions → `open-questions-and-assumptions.md`
Campi completi Evento/Email; oggetto Nota (dove vive, related list); differenza audit vs attività in timeline dell'org di riferimento; personalizzazione filtri timeline; completamento/annullamento/eliminazione attività (Da testare).
