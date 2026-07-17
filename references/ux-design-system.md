# CRM — UX & design system rules

Goal: feel as professional and information-dense as a polished enterprise CRM, but faster and simpler for non-technical sales users. Reuse the existing dashboard shell and CSS vocabulary — don't fork a new design language. (This spec uses English copy throughout; localize it if your app targets another language.)

## Layout & shell
- Live inside `layouts/dashboard.blade.php` (existing sidebar + header + auth). Add a sidebar link **"CRM"** (or under a "Sales" group) with active state via `request()->routeIs('dashboard.crm.*')`, near "Today"/"Pipeline".
- Content width = full dashboard content area. The list view is table-first; the detail view is a two-column layout (main cards + right rail with next-action/timeline) collapsing to one column on narrow screens.
- Put CRM CSS in `public/css/crm.css` (loaded on CRM pages), following the class patterns already in `public/css/*.css` (e.g. score/priority classes referenced by the model accessors: `score-fast/mid/low/empty`, `priority-0..5`). **Reuse those accessors** (`getScoreClassAttribute`, `getPriorityClassAttribute`) for badges.

## Table (list view)
- Dense but readable: compact row height, sticky header, zebra or hover highlight, horizontal scroll on small screens (the app already warns mobile users about horizontal scroll — reuse `config('crm.brand.dashboard_notice_mobile')`).
- Sticky first column (contact) is a nice-to-have, not required.
- Numeric/date columns right-or-left aligned consistently; truncate long text with a title tooltip.
- Row is clickable to open detail (but checkboxes, links, and the action ▾ stop propagation).

## Badges (clear, scannable)
- **Status**: colored pill per `crm_lead_statuses` (new=neutral, contacted=info, nurturing=accent, appointment=warn/positive, won=positive/green, lost=danger/red, not_interested=muted).
- **Priority**: use `priority-{0..5}` classes; higher = stronger. Show 4–5 as "high".
- **Lead score**: `score-fast/mid/low` (≥8 / ≥5 / else).
- **Urgency / bucket** (from service): `overdue` = red with "N days overdue", `today` = amber "today", `upcoming` = neutral, `first_contact` = accent "to contact". These drive the strongest visual emphasis — an operator must spot overdue + today at a glance.

## Visual priority rules (what must pop)
1. Overdue follow-ups (red).
2. Today's actions (amber).
3. High-priority never-contacted leads (accent).
4. Flagged rows (flag icon + subtle row tint).
Everything else stays calm. No decorative color.

## States (all required)
- **Empty**: curated per view (e.g. "No overdue follow-ups. Great work." with a subtle icon and a link to another view). Never a blank table.
- **Loading / skeleton**: show shimmer rows on navigation where a spinner would otherwise flash (Alpine `x-cloak` + skeleton markup). Server-rendered pages: keep skeletons for the async bits only.
- **Error / unavailable**: mirror the existing `pipelineUnavailable` graceful degradation — if the table or a new column is missing, render an informative panel, not a 500.
- **Missing/invalid data**: null follow-up date → "no follow-up date"; null owner → "Unassigned"; null email/phone → hide the quick link, don't render broken `mailto:`/`tel:`.

## Interactions (Alpine only)
- Filters drawer: `x-data` open/close, focus-trap, Esc to close, backdrop click to close.
- Column toggle menu: persists to `localStorage`.
- Bulk select: header checkbox toggles all visible; a reactive count drives the bulk bar.
- Row action ▾ and detail quick actions: small dropdowns; each action is a `<form method=POST>` with `@method('PATCH')` + `@csrf`, or an Alpine-submitted form. Keep actions to 1–2 clicks.
- No client-side data grid, no virtualization, no external JS libs.

## Accessibility
- Semantic `<table>` with `<th scope="col">`; sortable headers are `<button>`s with `aria-sort`.
- Every interactive control has a visible label or `aria-label`; icon-only buttons get `aria-label` (mirror the reference org's "Show Actions", "Filters", etc.).
- Visible focus states on all controls; drawer and dropdowns keyboard-operable; contrast meets WCAG AA (badges included — don't rely on color alone; include text).
- Checkbox column header labeled "Select all".

## Responsive
- Desktop-first (sales work on laptops). Tablet: table scrolls horizontally, filters drawer full-height. Mobile: essential but functional — stack the detail cards, allow table horizontal scroll, keep quick actions reachable.

## Copy & tone
- Operational, imperative for actions ("Contact", "Schedule follow-up", "Log outcome"). Match the next-action phrasing already produced by the service ("Follow-up overdue by N days", "Contact the lead via {channel}").
- No fake charts, no empty dashboards, no numbers without real data behind them.
