# Roadmap V1 / V2 / V3

**Scope:** prioritizzazione motivata di ogni funzione. Fonte: sintesi di tutti i `*-system.md`. Regola: non mettere tutto in V1; motivare.

## V1 — CRM operativo essenziale (indispensabile al team commerciale)
- **Oggetti:** Lead (su pipeline esistente), Account, Referente, Opportunità (+ Attività base). Relazioni + **conversione Lead**.
- **List view:** tabella densa, ricerca, sort singolo, **filtri curati** (stato/fase/owner/canale/bucket/date/flag) via query-string + chip, **viste predefinite** + pin default, azioni riga, **select+bulk essenziali** (cambia stato/assegna/flag/elimina/esporta), paginazione, stati vuoto/loading/errore. Vista **"per fase" raggruppata** (Opportunità).
- **Record page:** header+azioni, **detail sections + inline edit**, **Path** (Lead status, Opp fase) con commit + audit, **activity panel + timeline** (task/call/nota), related list principali, **elimina con conferma**.
- **Conversione:** Account+Referente+Opportunità in transazione, dedup base, "no opportunità", audit.
- **Quick actions:** contattato, esito, follow-up (+3/7/14), priorità, stato, nota, chiudi (won/lost+motivo).
- **Trasversali:** policy owner (admin vs sales, delete admin-only, "assegnati a me"), validazione inline, toast, ricerca globale base, app-shell riuso + object-nav, audit azioni sensibili, seed demo, test.
- *Motivazione:* copre il lavoro quotidiano (chi contattare, avanzare trattative, loggare attività) col minimo attrito; sfrutta dati/engine esistenti.

## V2 — CRM avanzato (produttività e controllo)
- **Kanban drag&drop** (SortableJS) + somme colonna + chiusura won/lost modale.
- **Filter builder** generico (campo/operatore/valore, date relative, salva vista, OR), **saved views utente** (CRUD+condivisione base).
- **Colonne configurabili + resize** (per vista), **mass inline edit**, **split view**.
- **Attività ricche:** eventi/calendario (FullCalendar), email registrata, **reminder/notifiche**, filtri timeline.
- **Note/File** ricchi, related list con azioni inline+paginazione.
- **Unsaved-changes** completo, ottimistic lock, business rules ricche.
- **Ruoli team/manager**, owner-scope configurabile, audit ricco, ricerca (Scout), cache conteggi, jobs/queue per bulk/notifiche.
- *Motivazione:* aumenta efficienza e governance quando il volume/uso cresce; introduce le uniche 2 dipendenze (SortableJS, FullCalendar) solo quando i dati le giustificano.

## V3 — Enterprise / opzionale
- **Catalogo prodotti/listini/line items**, forecasting avanzato, **ruoli referenti M:N**, quote/preventivi.
- **Field-level security**, sharing rules/team avanzati, viste condivise granulari, impersonation (se necessario).
- **API pubblica**, integrazioni (email/telefonia/calendar esterni), import wizard ricco, virtualizzazione data-grid.
- *Motivazione:* valore solo per organizzazioni strutturate; alto costo/complessità.

## Non replicare (dalla piattaforma di riferimento)
Flow/Process Builder/automation builder · Report/Dashboard builder · AI assistant integrato · sharing rules/org-wide defaults complessi · app switcher/utility bar · multi-currency avanzata · page-layout editor per profilo · conversation insights · integrazioni chat embedded.
*Motivazione:* funzioni-piattaforma non necessarie al flusso commerciale del target app, troppo pesanti/proprietarie o sostituibili con soluzioni più semplici.

## Criteri di promozione V1→V2→V3
Una funzione sale di priorità solo se: (a) risolve un attrito reale osservato dal team, (b) i dati la giustificano (es. Kanban dopo che stage è stabile), (c) il costo/rischio è sostenibile sullo stack (Blade/Alpine/SQLite/Aruba).
