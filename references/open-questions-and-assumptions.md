# Open questions & assumptions

Tracks everything that's **uncertain**, **not yet tested**, or **assumed**. Every item has: status, why, and how to resolve it (test or decision). Update every cycle. If research is interrupted, this file + `definition-of-done.md` let you resume without redoing the analysis.

## Current assumptions (to confirm with dynamic testing)
| # | Assumption | Basis | How to confirm |
|---|---|---|---|
| AS1 | The org's Opportunity stages are `Qualify→Meet & Present→Propose→Negotiate→Closed Won→Closed Lost` | Observed (New Opp picklist) | Create 1 opp per stage and verify Path/Kanban |
| AS2 | The Lead status ladder is `New→Contacted→Nurturing→Unqualified→Converted` | Observed (record Path) | Create a lead and move its status along the Path |
| AS3 | Conversion creates Account+Contact+Opportunity in a single transaction | Observed (modal + helper text) | Convert a demo lead and verify the 3 records + timeline |
| AS4 | Account related lists are Contacts/Opportunities/Cases | Observed | Populate and verify counts/actions |
| AS5 | Lead record sections are About/Get in Touch/Segment/History | Observed (layout) | Verify fields per section with inline edit |

## Open questions (not yet observed/tested)
| # | Question | Area | Priority |
|---|---|---|---|
| Q1 | How does the **filter builder** behave: operators per type, AND/OR, groups, relative dates? | filter-builder | high |
| Q2 | Does **Kanban** support real drag&drop + stage-change confirmations? Column totals? | kanban | high |
| Q3 | **Inline edit** behavior: batch save bar, validation, non-editable fields? | inline-edit | high |
| Q4 | Required **validation** + error messages on every modal (Lead/Account/Contact/Opp)? | validation | high |
| Q5 | **Unsaved changes**: warning when closing a modal with pending edits? | unsaved-changes | medium |
| Q6 | **Bulk actions** available per row/multi-select (change owner, delete, etc.)? | bulk-actions | high |
| Q7 | **Saved views**: create/clone/rename/share/filter — full flow? | saved-views | medium |
| Q8 | **Notes & Files**: composer, attachments, related list, where do they live? | activity/files | medium |
| Q9 | **Cases**: object available? fields? Account/Contact relationship? | cases | low |
| Q10 | **Split/dual view**: layout and behavior? | split-view | low |
| Q11 | Full Lead fields + picklists (Rating, Lead source, Industry, # of employees)? | field-catalog | high |
| Q12 | Forecast category values? Does a loss reason exist on Closed Lost? | opportunity | medium |
| Q13 | **Pagination** behavior with many records (>50)? | pagination | medium |

## Elements potentially NOT verifiable in the demo (⛔ — justify + future procedure)
| # | Element | Why | Future verification procedure |
|---|---|---|---|
| NV1 | Real multi-user permissions/sharing | The demo has 1 user only; can't create users/roles | Document from prior knowledge of the reference org + test in a multi-user org or via proposed Laravel roles |
| NV2 | Server-side automation/Flow/validation rules | Off-limits to touch and not always visible | Infer from UI behavior; mark as "To verify" |
| NV3 | Forecast/Quotes/Products if not enabled | Feature not active in the trial | Mark "To verify"; propose an optional Laravel model (V3) |
| NV4 | Performance with thousands of records | Can't realistically populate | Define targets + Laravel performance tests in `testing-strategy.md` |

## Resume notes (for continuity across sessions)
- Reference-org session: state saved in `scratchpad/sess/session-state.json` (reusable while valid; needs a new OTP from the user once it expires).
- Automation toolkit: scripts in `scratchpad/crmdriver/` (Playwright, `channel: chrome`).
- **Next unverified item:** create the first Lead dataset + test the New-Lead modal (see `definition-of-done.md` → Next cycle).
