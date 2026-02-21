# TightWallet — Tecnologie consigliate (stack di implementazione)

Questa nota traduce i requisiti di TightWallet (partita doppia “vera”, import serio, ricorrenti con Inbox, multi‑valuta, **offline‑first + sync**) in un set di tecnologie pragmatico e scalabile.

---

## Stack consigliato (pragmatico, solido, “da prodotto”)

### Backend
- **.NET 8/9 + ASP.NET Core**
  - Tipizzazione forte e ottimo dominio applicativo (regole contabili, validazioni, invarianti)
  - Performance e tooling maturi
- **Persistenza**
  - **EF Core** per la parte CRUD e consistenza transazionale
  - **Dapper** (opzionale) per query/report complessi e ottimizzati
- **Database: PostgreSQL**
  - Integrità, indici, JSONB (import rows, config widget), NUMERIC per importi
- **Job & schedulazione**
  - **Hangfire** (consigliato) o **Quartz.NET** per:
    - generare le *Expected Instances* (Inbox ricorrenti)
    - gestire pipeline import (parse → mapping → preview → commit)
    - housekeeping / notifiche / riconciliazioni

**API contract**
- **REST + OpenAPI/Swagger** (MVP-friendly, standard)
- Eviterei GraphQL nell’MVP: aumenta complessità su caching/sync/import senza un guadagno immediato.

---

### Web App
- **Angular 17+** (coerente con ecosistema già usato)
- UI kit: **Angular Material** *oppure* **PrimeNG**
  - Material: essenziale e consistente
  - PrimeNG: più componenti “enterprise” pronti
- Grafici/dashboard: **ECharts** o **Chart.js**
  - ECharts spesso eccelle su dashboard ricche e interattive

---

### Mobile (offline-first vero)
Qui la scelta impatta tantissimo su UX e sincronizzazione.

**Consiglio #1 (forte): Flutter**
- Flutter + DB locale **SQLite**
- ORM/DB layer: **drift** (ottimo) oppure alternative (isar/realm, a scelta)
- Pro: UI molto fluida, ottimo controllo offline-first

**Consiglio #2: React Native (se vuoi massimizzare riuso TS)**
- DB locale: SQLite + WatermelonDB/Realm (scelta da valutare con cura)

**Consiglio #3: .NET MAUI (se vuoi restare full C#)**
- Pro: C# end‑to‑end
- Contro: ecosistema/UX a volte più delicati; offline/sync va comunque progettato bene

**Scelta consigliata complessiva**
> **Angular (web) + .NET (backend) + Flutter (mobile)**

---

## Componenti “difficili” e tech che semplificano la vita

### Offline-first & Sync (il cuore sporco)
**Target**: mobile sempre utilizzabile senza rete.

- DB locale: **SQLite**
- Sync protocol: **operazioni idempotenti + change feed**
  - `operation_id` (UUID) lato client
  - `operation_log` lato server (non riapplicare la stessa operazione)
  - `change_events` append‑only per scaricare delta
  - `sync_cursors.last_event_id` per riprendere da dove eri rimasto
- Conflitti:
  - MVP: **last‑write‑wins + log conflitti**
  - Evoluzione: risoluzione guidata in UI + regole per entità critiche (es. account tree)

In .NET: endpoint tipo:
- `POST /sync/push`
- `GET  /sync/pull?cursor=...`

---

### Import (GnuCash / Money Manager / CSV)
- Parser dedicati + staging su DB:
  - `import_jobs`, `import_rows`
- Storage file: **S3-compatible**
  - AWS S3 / MinIO / (equivalente Azure Blob)
- Job runner: Hangfire/Quartz
- Deduplica:
  - hash riga (`row_hash`)
  - regole su (data, importo, payee, descrizione, conto) con tolleranze configurabili
- UX import consigliata:
  - mapping e preview prima del commit
  - commit atomico
  - report errori e possibilità di “fix & retry”

---

### Contabilità e soldi (precisione numerica)
- In DB: **NUMERIC** (mai float)
- In .NET: **decimal**
- Libreria “money” (opzionale ma utile): **NodaMoney**
  - riduce bug su currency/rounding e rende il codice più esplicito

---

### Observability (quando qualcosa non torna, devi saperlo)
- **OpenTelemetry** (tracing/metrics)
- **Serilog** (logging strutturato)
- **Sentry** (o equivalente) per error tracking su web/mobile/backend

---

## DevOps / Deploy (minimal ma serio)
- **Docker** per backend + db (+ redis se lo usi)
- **GitHub Actions** (CI):
  - build + test
  - lint/format
  - check migrazioni
- Ambienti: dev / staging / prod
- Hosting: va bene quasi tutto purché Postgres sia affidabile:
  - Render / Fly.io / Azure App Service / AWS ECS (scegli quello più comodo)
- **Terraform** (opzionale) quando vuoi infra ripetibile

---

## Alternative coerenti (se vuoi ridurre varietà di stack)

### Opzione A — Full TypeScript
- Backend: **NestJS + Prisma**
- Web: Angular o React
- Mobile: React Native  
**Pro**: linguaggio unico  
**Contro**: la parte contabile/sync resta complessa; perdi parte del rigore/tooling tipico .NET (dipende dal team).

### Opzione B — Full .NET
- Backend: .NET
- Web: **Blazor**
- Mobile: **MAUI**  
**Pro**: un ecosistema unico  
**Contro**: se vuoi Angular, Blazor è un cambio grosso.

---

## Scelta finale “da battaglia” per TightWallet

**Backend .NET + Postgres + Hangfire (+ Redis opzionale) + Web Angular + Mobile Flutter + SQLite locale + Sync idempotente.**

Questo set ti dà:
- contabilità rigorosa senza compromessi
- import e ricorrenti gestibili e debuggabili
- offline-first reale
- evoluzione graduale senza riscritture dolorose

---

## Prossimo passo tecnico consigliato
Definire:
1) **confini dei moduli** (Domain / Sync / Import / Reporting)  
2) **API minima** (OpenAPI) per coprire Classic + Advanced senza duplicare logica
