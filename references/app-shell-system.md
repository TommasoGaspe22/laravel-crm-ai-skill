# App shell & navigation system

**Scope:** shell applicativa (header globale, navigazione, utility). Fonte: **Osservato** (screenshot reali dell'app "Vendite").

## Osservato
- **App rail sinistra** (app switcher Sales): Pagina iniziale, Referenti, Account, Vendite, Servizio, Marketing, Commerce, Il tuo account, DevOps Center — è lo **switcher app**, non la nav CRM. Target app: **non replicare** (assumendo si abbia già una sidebar dashboard).
- **Header globale:** logo · **ricerca globale** centrale ("Cerca…") · icone destra (help, setup ⚙, notifiche 🔔 con badge, avatar/menu utente).
- **Barra tab oggetti:** Lead · Referenti · Account · Opportunità · Prodotti · Listini prezzi · Calendario · Analytics · Fatture — ognuna con ▾ (record recenti + Nuovo). Tab attiva sottolineata. Overflow "Altro" se troppe.
- **App name** a sinistra ("Vendite").
- **Utility bar** in basso ("To Do List").
- **Stati:** hover/active/focus sui controlli; tipografia densa; colori semantici; icone oggetto colorate.

## Proposta Laravel (Proposto per Laravel)
- **Riusare la shell dashboard esistente** (`layouts/dashboard.blade.php`: sidebar + header + auth). Aggiungere:
  - **Object nav** CRM (Lead/Account/Referenti/Opportunità/Attività) come strip/tab con active-state (`request()->routeIs('dashboard.crm.*')`), ognuna con dropdown "recenti + Nuovo".
  - **Ricerca globale** in header → endpoint `q` cross-oggetto (Lead/Account/Contact/Opportunità) — vedi ricerca in `modules.md`.
  - Menu utente, notifiche (V2), help.
- **Componente** `x-crm.app-shell` (o estensione del layout): header + object-nav + content slot. Breadcrumb `x-crm.breadcrumb` (oggetto ▸ record).
- **Responsive:** sidebar collassabile (già presente), object-nav → menu su mobile; tabelle scroll orizzontale.
- **Accessibilità:** nav con `aria-current`, focus states, skip-link, ricerca etichettata.

## Priorità
- **V1:** object-nav CRM + ricerca globale + breadcrumb + riuso shell esistente. **V2:** dropdown recenti per tab, notifiche, azioni globali (Nuovo rapido). **V3:** app switcher multi-area, command palette.
- **Non replicare:** app rail switcher, utility bar docked, Agentforce/Slack.

## Open questions → `open-questions-and-assumptions.md`
Ricerca globale (scope, risultati); dropdown "recenti" per tab; command palette (pattern, non specifico dell'org di riferimento).
