# Testing strategy (Laravel)

**Scope:** strategia di test per il CRM. Fonte: **Proposto per Laravel**. Riusa i pattern `tests/Feature` esistenti (pipeline ha già test su accesso/CRUD/import/export/bulk).

## Livelli & cosa coprire
- **Unit:** accessor (`computed_status`, `is_closed/is_won`), enum/config, `CommercialPipelineActionService` (bucket/next-action per fixture), `FilterQueryBuilder` (traduzione operatori→SQL), `TimelineBuilder`.
- **Feature (HTTP):**
  - **List view:** ogni vista/filtro, ricerca `q`, sort (whitelist rifiuta colonne ignote — anti-injection), paginazione query-string, stato vuoto, tabella mancante → no-500.
  - **Record page:** show autorizzato, inline edit (`PATCH` parziale), related counts.
  - **Form/validazione:** required inline, tipi, picklist `in:`, email/date; 422/redirect-errors; no partial write.
  - **Conversione:** nuovo account · account esistente (dedup) · senza opportunità · dati minimi · rollback su errore (mock failure) · stato lead=convertito · trasferimento relazioni · idempotenza.
  - **Stage transition:** transizioni valide/invalide, chiusura won/lost + motivo persa, audit scritto.
  - **Attività/timeline:** log task/call/nota → compare in timeline; completamento; scopes upcoming/overdue.
  - **Bulk:** applica solo agli id dati, transazione, autorizzazione per-record, esito parziale.
  - **Saved views/filtri:** serializzazione, whitelist campi.
  - **Policy/permessi:** guest→login; sales vs admin (delete/bulk/assign 403 per sales); scope "assegnati a me"; convert.
- **Data integrity/migration:** up/down su SQLite; additive non distruttive; FK/indici; backfill; soft delete + ripristino.
- **Regression:** i test esistenti (Oggi/pipeline/import/export) restano verdi dopo l'estrazione del service e le migration additive.
- **Browser/E2E (V2):** flussi critici (crea lead → converti → opportunità → cambia fase → attività) con Playwright/Dusk.
- **Accessibility:** aria/focus/contrasto sui componenti chiave (assertion su markup + audit manuale).
- **Performance:** liste con N record (seed factory), assert su n° query (no N+1, es. `assertDatabaseQueryCount`), indici usati; target: list < X query, kanban < Y.

## Factory & seed
`Crm*Factory` + factory per pipeline entry; seeder demo **separato** dai dati reali (dati fittizi, mai PII reale). Seed di test per ogni scenario (stati/fasi/attività).

## Comandi
```bash
composer test        # php artisan test (config:clear prima)
./vendor/bin/pint    # style
npm run build        # Vite/Tailwind
php artisan test --filter=Crm
```

## Definition of tested-enough (per feature)
Happy path + validazione + autorizzazione + edge (vuoto/mancante) coperti; regression esistente verde; `pint` pulito; build ok; verifica manuale del flusso (`verification-before-completion`).

## Priorità
- **V1:** unit service/accessor, feature (viste/validazione/conversione/stage/policy/bulk), migration, regression. **V2:** E2E, performance, accessibility automatizzata. **V3:** load test, contract test API.
