# Activity & timeline system

**Scope:** activities (Task, Call, Event, Email, Note), composer, and record timelines. Source: **Dynamically tested** (2026-07-08: "Log a Call" and "New Task" on an opportunity → appeared in the timeline) + **Observed** (composer, timeline filters).

## Index
1. Composer · 2. Activity types · 3. Timeline · 4. Laravel model · 5. Priority · 6. Open questions

## 1. Composer (Observed/Tested)
On the record page, middle column, a button row: **Email · New Task · Log a Call · New Event** (each with ▾). They open a **modal/composer**.

### "Log a Call" (Tested — real fields)
| Field | UI | Notes |
|---|---|---|
| Subject | text with presets (magnifier) | default "Call"; picklist of frequent subjects |
| Comments | textarea | tip "Ctrl+." for quick text |
| Name | lookup (Lead/Contact) | "who" — the related person |
| Related To | lookup (Account/Opportunity/…) | "what" — pre-filled with the current record |
| — | footer | `Cancel · Save` |

### "New Task" (Tested — fields)
Subject · **Due Date** (date) · (Name, Related To, Priority, Status, Assigned To — standard). → the Task model has a polymorphic `who` (Name) + `what` (Related To).
### Event / Email: **to verify** in detail (Event: Start/End/Location/Invitees; Email: recipients/subject/body).

## 2. Activity types (Observed/Deduced)
`task` · `call` · `event` · `email` (logged email) · `note` (a separate "Notes" object). Every activity has **who** (Lead/Contact) and **what** (Account/Opportunity/Lead) → polymorphic relations.

## 3. Timeline (Tested)
- Middle panel below the composer. Toggle "Show only activities with insights". **Filter:** "Within 2 months • All Activities • All Types" + gear (customizable).
- Groups: **"Upcoming & Overdue"** (future/overdue) + past by month. `Refresh · Expand All · Show All Activities`.
- **Tested:** after "Log a Call", a **"Call"** item appears in the timeline (type icon + subject + "Show More Actions" menu). Item with an empty subject → "Details for [No Subject]".
- Empty state: "No activities to show. Get started by sending an email, scheduling a task…".

## 4. Laravel model (Proposed for Laravel)
Single `crm_activities` table (replaces the *derived* timeline from earlier phases):
```
id, type(task|call|event|email|note), subject, body/comments,
status(open|completed|nullable), priority(nullable),
due_at(nullable), start_at(nullable), end_at(nullable),   // task/event
who_type+who_id (morph: lead/contact, nullable),          // "Name"
what_type+what_id (morph: account/opportunity/lead),      // "Related To"
owner_id(FK users), created_by, timestamps
```
Model `CrmActivity`: `morphTo who`, `morphTo what`, `belongsTo owner`. Scopes: `upcoming()`, `overdue()`, `completed()`, `ofType()`, `forRecord($model)`.
- **Timeline** on a record = `CrmActivity::forRecord($record)->orderByDesc(coalesce(due_at,start_at,created_at))` grouped by month, with an "upcoming/overdue" section on top. Component `x-crm.activity-panel` + `x-crm.timeline` (see `component-catalog.md`).
- **Composer** = `LogActivity` Action (validation: subject required for tasks; valid dates; consistent who/what). Toast on save.
- **System events vs. activities:** distinguish audit (owner/status/stage change, conversion) → `audit_events` table/rows **or** `crm_activities` of type `system` with a flag. Both (activities + events) show up in the timeline, as in the reference org.
- **Follow-up:** a follow-up = a `task`/`call` with a future `due_at`; bucket (overdue/today/upcoming) via the existing `CommercialPipelineActionService` / query scope.
- **Implementation risk:** who/what polymorphic relations; timezone on due_at/start_at; N+1 in the timeline (eager-load who/what/owner); permissions (see `permissions-and-roles.md`).

## 5. Priority
- **V1:** `crm_activities` with task/call/note, composer (subject+comments+who+what+due), record timeline with "upcoming/overdue", task completion, system events in the timeline (conversion, stage/owner/status change).
- **V2:** events/calendar (FullCalendar), logged email, reminders/notifications, advanced timeline filters, "Ctrl+." quick text.
- **V3:** email/telephony integration, AI-generated insights.
- **Do not replicate:** AI conversation insights, proprietary integrations.

## 6. Open questions → `open-questions-and-assumptions.md`
Full Event/Email fields; the Note object (where it lives, related list); the difference between audit and activities in the reference org's timeline; timeline filter customization; completing/cancelling/deleting activities (to test).
