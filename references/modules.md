# V1 modules — build spec (routes, controllers, views, actions)

Canonical prefix: **`/dashboard/crm`** (inside the existing `auth` + `dashboard.` group, inherits layout/nav/roles). Add a redirect `/crm/leads → dashboard.crm.leads.index` to honor the brief's short URL.

## 0. Shared service (do first)

`App\Support\CommercialPipelineActionService`
- Move `actionForEntry()`, `datedFollowUpAction()`, `baseAction()` out of `CommercialPipelineTodayController` verbatim.
- Public API: `actionFor(CommercialPipelineEntry $e, ?Carbon $today = null): ?array` and `bucketFor($e): string`.
- Update the Today controller to call the service. **Its existing tests must stay green** — pure refactor, no behavior change.
- Buckets (unchanged): `overdue, today, upcoming, first_contact, analysis_pending, stalled, sales_pending, flagged`. Tones: `danger, warn, neutral, accent, positive`.

## 1. Routes

```php
Route::prefix('crm')->as('crm.')->group(function () {
    Route::get('/leads', [LeadCrmController::class, 'index'])->name('leads.index');
    Route::get('/leads/export', [LeadCrmController::class, 'export'])->name('leads.export');
    Route::get('/leads/{entry}', [LeadCrmController::class, 'show'])->name('leads.show');

    // quick actions (each minimal, validated, policy-guarded, returns back() with status)
    Route::patch('/leads/{entry}/contacted',  [LeadActionController::class, 'markContacted'])->name('leads.contacted');
    Route::patch('/leads/{entry}/outcome',    [LeadActionController::class, 'recordOutcome'])->name('leads.outcome');
    Route::patch('/leads/{entry}/follow-up',  [LeadActionController::class, 'scheduleFollowUp'])->name('leads.followup');
    Route::patch('/leads/{entry}/priority',   [LeadActionController::class, 'setPriority'])->name('leads.priority');
    Route::patch('/leads/{entry}/status',     [LeadActionController::class, 'setStatus'])->name('leads.status');
    Route::patch('/leads/{entry}/note',       [LeadActionController::class, 'addNote'])->name('leads.note');
    Route::patch('/leads/{entry}/close',      [LeadActionController::class, 'close'])->name('leads.close');
    Route::patch('/leads/{entry}/flag',       [LeadActionController::class, 'toggleFlag'])->name('leads.flag'); // or reuse pipeline.flag
    Route::patch('/leads/bulk',               [LeadActionController::class, 'bulk'])->name('leads.bulk');
});
```

Controllers: `App\Http\Controllers\Dashboard\Crm\LeadCrmController` (read: index/show/export) and `LeadActionController` (writes). Route-model-bind `{entry}` to `CommercialPipelineEntry`. `authorize()` in every method.

## 2. Module 1 — Lead list view (`index`)

Renders `dashboard/crm/leads/index.blade.php`. Server-side, paginated (`config('crm.dashboard.page_sizes')`, `default_per_page`).

Must include:
- **Header**: title "Leads" + big current-view name; **live count**; meta line `"{count} leads • Sorted by {col} • View: {view} • Updated now"`.
- **View selector** (dropdown, Alpine) linking to `?view=…` (see §3).
- **Quick search** (`q`, GET) across `lead_name, company, role, email, phone, lead_notes, address`.
- **Filters** button → drawer (§5).
- **Sortable columns** via `?sort=col&dir=asc|desc` (whitelist columns server-side).
- **Table**: select-all + per-row checkbox, configurable columns (§6), status/priority/urgency badges, next-action cell (from service), row-action ▾ (§4).
- **Bulk bar** appears when rows selected → change status / assign owner / flag / export selected.
- **Pagination** (`withQueryString()`), **empty state**, **loading skeleton**, **error state** (degrade gracefully if table/columns missing — mirror `hasPipelineTable()` guards).

Build `index()` on top of a `filteredQuery(Request)` mirroring the existing pipeline controller's pattern (reuse its column whitelist and `contactMethods()`/`followUpOutcomes()`), extended with view + status + owner + bucket filters. **Eager-load `owner`** to avoid N+1.

## 3. Module 2 — Saved views (`?view=`)

Declare in `App\Support\LeadListViews` as `[key => ['label' => ..., 'apply' => fn(Builder $q) => ...]]`. Default = `open`.

| key | Label | Filter |
|---|---|---|
| `all` | All leads | no filter |
| `open` | Open leads *(default)* | open predicate (not won/lost/not_interested) |
| `to_contact` | To contact | `contacted = false` |
| `follow_up_overdue` | Overdue follow-up | bucket `overdue` |
| `follow_up_today` | Follow-up today | bucket `today` |
| `contacted` | Contacted | `contacted = true` |
| `closed` | Closed / not interested | won/lost/not_interested |
| `high_priority` | High priority | `priority >= 4` |
| `assigned_to_me` | Assigned to me | `owner_id = auth id` |

Bucket-based views (`overdue/today`) may need to filter the collection via the service when not expressible in SQL; prefer SQL where possible (date comparisons on `follow_up_date`), fall back to a post-query filter for the derived buckets, keeping pagination correct (compute the filtered id set, then paginate). Document whichever approach you take.

## 4. Module — Row & bulk actions

Row ▾ menu: Open · Mark Contacted · Log Outcome · Schedule Follow-up · Change Status · Change Priority · Flag/Unflag · Delete (admin). Each posts to the matching quick-action route via a small Blade form / Alpine dropdown.

Bulk (checkboxes → `PATCH /leads/bulk` with `ids[]` + `action` + payload): `change_status`, `assign_owner`, `flag`, `unflag`, `export`. Validate `ids` exist; authorize; wrap in a transaction.

## 5. Module — Filters drawer

Right-side Alpine drawer, GET form (all filters are query params so views are shareable/bookmarkable, reference-style):
`status` (multi), `priority` (min), `owner_id`, `channel` (`contact_method`), `follow_up` bucket, `created_from`/`created_to` (date range), `flagged`. "Apply" submits; "Reset" clears. Show active-filter chips above the table.

## 6. Module — Configurable columns

Column catalog defined once (label, key, sortable, default-visible). User toggles visible columns in a menu; persist selection in `localStorage` via Alpine (V1). Always-visible: contact + company. Never hide the row-action column.

## 7. Module 4 — Lead detail (`show`)

`dashboard/crm/leads/show.blade.php`:
- **Highlights header**: company · contact · role · status badge · priority badge · owner; quick-action buttons (Contact, Log Outcome, Schedule Follow-up, Change Status, Edit, Close).
- **Next-action panel**: from `CommercialPipelineActionService::actionFor()` — title, action text, channel, urgency, due date.
- **Status ladder** (chevrons) reflecting the app's status; clicking a later stage opens "change status".
- **Cards**: Profile (contact, role, company, industry, location, email, phone, quick links tel:/mailto:/website/LinkedIn when present); Sales (channel, priority, lead score, outcomes, appointments, contract value).
- **Derived timeline** (chronological, newest first): first contact → follow-up 1 (date/method/outcome) → follow-up 2 → analysis/sales appointment → sales outcome. Build from the pipeline fields; each item = badge + label + date + method. Empty timeline → curated empty state.
- **Internal notes**: `lead_notes` display + "add note" (appends timestamped line; V1 keeps single text field, prepend new notes with date + user).
- **Edit** reuses the existing pipeline `form.blade.php` where practical (extend it with the new fields) rather than a second form.

## 8. Quick commercial actions (validation)

All are single-purpose `PATCH`, authorize + validate server-side, redirect `back()->with('status', …)`:
- `markContacted`: sets `contacted=true`, optional `contact_method` (in allowed list).
- `recordOutcome`: `follow_up_outcome`|`second_follow_up_outcome` ∈ `FOLLOW_UP_OUTCOMES`; sets related `*_done` + `*_appointment_set` when outcome implies it.
- `scheduleFollowUp`: `follow_up_date`/`second_follow_up_date` (date, ≥ today) + `method`; support presets +3/+7/+14 days computed server-side.
- `setPriority`: int 0–5.
- `setStatus`: key ∈ `crm_lead_statuses` (writes explicit `status`).
- `addNote`: string, max length; prepend to `lead_notes` with timestamp + user name.
- `close`: reason ∈ {`not_interested`, `unreachable`, `future_opportunity`, `lost`} → maps to `status` + sets `sales_outcome`/`follow_up_outcome` consistently.
- Reuse existing `toggleFlag` (`pipeline.flag`) if not adding a CRM-specific one.

Keep click-cost minimal: row/detail quick actions should be 1–2 clicks (dropdown → submit), no full-page form for the common cases.
