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
- **Header**: title "Lead" + big current-view name; **live count**; meta line `"{count} lead • Ordinati per {col} • Vista: {view} • Aggiornato ora"`.
- **View selector** (dropdown, Alpine) linking to `?view=…` (see §3).
- **Quick search** (`q`, GET) across `lead_name, company, role, email, phone, lead_notes, address`.
- **Filters** button → drawer (§5).
- **Sortable columns** via `?sort=col&dir=asc|desc` (whitelist columns server-side).
- **Table**: select-all + per-row checkbox, configurable columns (§6), status/priority/urgency badges, next-action cell (from service), row-action ▾ (§4).
- **Bulk bar** appears when rows selected → change status / assign owner / flag / export selected.
- **Pagination** (`withQueryString()`), **empty state**, **loading skeleton**, **error state** (degrade gracefully if table/columns missing — mirror `hasPipelineTable()` guards).

Build `index()` on top of a `filteredQuery(Request)` mirroring the existing pipeline controller's pattern (reuse its column whitelist and `contactMethods()`/`followUpOutcomes()`), extended with view + status + owner + bucket filters. **Eager-load `owner`** to avoid N+1.

## 3. Module 2 — Saved views (`?view=`)

Declare in `App\Support\LeadListViews` as `[key => ['label' => ..., 'apply' => fn(Builder $q) => ...]]`. Default = `aperti`.

| key | Label | Filter |
|---|---|---|
| `tutti` | Tutti i lead | no filter |
| `aperti` | Lead aperti *(default)* | open predicate (not vinto/perso/non_interessato) |
| `da_contattare` | Da contattare | `contacted = false` |
| `follow_up_ritardo` | Follow-up in ritardo | bucket `overdue` |
| `follow_up_oggi` | Follow-up di oggi | bucket `today` |
| `contattati` | Contattati | `contacted = true` |
| `chiusi` | Chiusi / non interessati | vinto/perso/non_interessato |
| `alta_priorita` | Alta priorità | `priority >= 4` |
| `assegnati_a_me` | Assegnati a me | `owner_id = auth id` |

Bucket-based views (`overdue/today`) may need to filter the collection via the service when not expressible in SQL; prefer SQL where possible (date comparisons on `follow_up_date`), fall back to a post-query filter for the derived buckets, keeping pagination correct (compute the filtered id set, then paginate). Document whichever approach you take.

## 4. Module — Row & bulk actions

Row ▾ menu: Apri · Segna contattato · Registra esito · Pianifica follow-up · Cambia stato · Cambia priorità · Flag/Unflag · Elimina (admin). Each posts to the matching quick-action route via a small Blade form / Alpine dropdown.

Bulk (checkboxes → `PATCH /leads/bulk` with `ids[]` + `action` + payload): `cambia_stato`, `assegna_owner`, `flag`, `unflag`, `esporta`. Validate `ids` exist; authorize; wrap in a transaction.

## 5. Module — Filters drawer

Right-side Alpine drawer, GET form (all filters are query params so views are shareable/bookmarkable, reference-style):
`stato` (multi), `priorita` (min), `owner_id`, `canale` (`contact_method`), `follow_up` bucket, `creato_da`/`creato_a` (date range), `flagged`. "Applica" submits; "Reset" clears. Show active-filter chips above the table.

## 6. Module — Configurable columns

Column catalog defined once (label, key, sortable, default-visible). User toggles visible columns in a menu; persist selection in `localStorage` via Alpine (V1). Always-visible: referente + azienda. Never hide the row-action column.

## 7. Module 4 — Lead detail (`show`)

`dashboard/crm/leads/show.blade.php`:
- **Highlights header**: azienda · referente · ruolo · stato badge · priorità badge · owner; quick-action buttons (Contatta, Registra esito, Pianifica follow-up, Cambia stato, Modifica, Chiudi).
- **Next-action panel**: from `CommercialPipelineActionService::actionFor()` — title, testo azione, canale, urgenza, due date.
- **Status ladder** (chevrons) reflecting the app's status; clicking a later stage opens "cambia stato".
- **Cards**: Anagrafica (referente, ruolo, azienda, settore, località, email, telefono, quick links tel:/mailto:/website/LinkedIn when present); Commerciale (canale, priorità, lead score, esiti, appuntamenti, valore contratto).
- **Derived timeline** (chronological, newest first): primo contatto → follow-up 1 (data/metodo/esito) → follow-up 2 → appuntamento analisi/vendita → esito vendita. Build from the pipeline fields; each item = badge + label + date + method. Empty timeline → curated empty state.
- **Note interne**: `lead_notes` display + "aggiungi nota" (appends timestamped line; V1 keeps single text field, prepend new notes with date + user).
- **Modifica** reuses the existing pipeline `form.blade.php` where practical (extend it with the new fields) rather than a second form.

## 8. Quick commercial actions (validation)

All are single-purpose `PATCH`, authorize + validate server-side, redirect `back()->with('status', …)`:
- `markContacted`: sets `contacted=true`, optional `contact_method` (in allowed list).
- `recordOutcome`: `follow_up_outcome`|`second_follow_up_outcome` ∈ `FOLLOW_UP_OUTCOMES`; sets related `*_done` + `*_appointment_set` when outcome implies it.
- `scheduleFollowUp`: `follow_up_date`/`second_follow_up_date` (date, ≥ today) + `method`; support presets +3/+7/+14 giorni computed server-side.
- `setPriority`: int 0–5.
- `setStatus`: key ∈ `crm_lead_statuses` (writes explicit `status`).
- `addNote`: string, max length; prepend to `lead_notes` with timestamp + user name.
- `close`: reason ∈ {`non_interessato`, `non_raggiungibile`, `opportunita_futura`, `perso`} → maps to `status` + sets `sales_outcome`/`follow_up_outcome` consistently.
- Reuse existing `toggleFlag` (`pipeline.flag`) if not adding a CRM-specific one.

Keep click-cost minimal: row/detail quick actions should be 1–2 clicks (dropdown → submit), no full-page form for the common cases.
