# Table system (data grid)

**Scope:** la tabella densa delle list view. Fonte: **Osservato/Testato** (liste popolate).

## Osservato
- `role="grid"`, header sticky, densità compatta, zebra/hover.
- **Header:** checkbox select-all + colonne; ogni header con **sort** (asc/desc, uno attivo, `aria-sort`) + menu **azioni colonna** ("Mostra azioni colonna": ordina, a capo/taglia testo, larghezza).
- **Riga:** checkbox selezione + celle + **✎ inline edit** su celle editabili + **azione riga ▾** (Modifica/Elimina/Cambia titolare/…). Prima cella (nome) = link al record.
- **Celle:** link, badge (stato/fase), avatar/owner (alias titolare), date formattate, testo troncato con tooltip.
- **Modalità "Modifica elenco"** → edit inline massivo delle celle.
- **Ordinamento** (Testato indirettamente): meta line "Ordinati per {col}".

## Proposta Laravel (Proposto per Laravel)
- **Componente** `x-crm.data-table`: props `:columns` (label, key, sortable, align, formatter, editable), `:rows`, `:sort`, `selectable`, `:rowActions`. Sort server-side via link `?sort=&dir=`; selezione + menu via Alpine.
- **Densità/colonne**: `column-management-system.md`. **Ordinamento**: whitelist colonne (anti-injection); indici DB sulle colonne ordinabili; una sort attiva (V1), multi (V2).
- **Formatter** per tipo (data/valuta/badge/link/owner). Troncamento + `title`.
- **Performance:** paginazione server-side (`pagination-and-loading-system.md`); eager-load relazioni mostrate (owner/account); niente N+1.
- **Accessibilità:** `<th scope=col>`, header sortabili `<button aria-sort>`, checkbox etichettate.

## Priorità
- **V1:** tabella densa, sort singolo, badge/formatter, link, azione riga, select+bulk, responsive (scroll orizzontale). **V2:** inline edit celle, multi-sort, resize colonne, sticky prima colonna. **V3:** virtualizzazione/data-grid lib (Tabulator) se necessario.

## Open questions
Multi-sort; resize persistente; virtualizzazione con molti record (→ `pagination-and-loading-system.md`).
