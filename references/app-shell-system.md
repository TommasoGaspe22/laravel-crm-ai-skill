# App shell & navigation system

**Scope:** the app shell (global header, navigation, utility bar). Source: **Observed** (real screenshots of the "Sales" app).

## Observed
- **Left app rail** (Sales app switcher): Home, Contacts, Accounts, Sales, Service, Marketing, Commerce, Your Account, DevOps Center — this is the **app switcher**, not the CRM nav. Target app: **do not replicate** (assuming it already has a dashboard sidebar).
- **Global header:** logo · centered **global search** ("Search…") · right icons (help, setup ⚙, notifications 🔔 with badge, avatar/user menu).
- **Object tab bar:** Lead · Contacts · Accounts · Opportunities · Products · Price Books · Calendar · Analytics · Invoices — each with a ▾ (recent records + New). Active tab underlined. "More" overflow if there are too many.
- **App name** on the left ("Sales").
- **Utility bar** at the bottom ("To Do List").
- **States:** hover/active/focus on controls; dense typography; semantic colors; colored object icons.

## Proposed Laravel design (Proposed for Laravel)
- **Reuse the existing dashboard shell** (`layouts/dashboard.blade.php`: sidebar + header + auth). Add:
  - CRM **object nav** (Lead/Account/Contact/Opportunity/Activity) as a strip/tabs with active-state (`request()->routeIs('dashboard.crm.*')`), each with a "recent + New" dropdown.
  - **Global search** in the header → cross-object `q` endpoint (Lead/Account/Contact/Opportunity) — see search in `modules.md`.
  - User menu, notifications (V2), help.
- **Component** `x-crm.app-shell` (or a layout extension): header + object-nav + content slot. Breadcrumb `x-crm.breadcrumb` (object ▸ record).
- **Responsive:** collapsible sidebar (already present), object-nav → menu on mobile; horizontal-scroll tables.
- **Accessibility:** nav with `aria-current`, focus states, skip-link, labeled search.

## Priority
- **V1:** CRM object-nav + global search + breadcrumb + reuse of the existing shell. **V2:** recent-items dropdown per tab, notifications, global actions (quick New). **V3:** multi-area app switcher, command palette.
- **Do not replicate:** app rail switcher, docked utility bar, built-in AI assistant/chat integration.

## Open questions → `open-questions-and-assumptions.md`
Global search (scope, results); "recent" dropdown per tab; command palette (pattern, not specific to the reference org).
