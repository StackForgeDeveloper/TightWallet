# TightWallet

**TightWallet** è un’app di gestione finanziaria personale basata su **partita doppia reale** (double-entry bookkeeping), ispirata a **GnuCash** per la solidità contabile e a **Money Manager** per la UX “di tutti i giorni”.

- **Classic**: semplice, rapida, quotidiana (per utenti non tecnici)
- **Advanced**: completa, contabile, rigorosa (per chi vuole controllo totale)

Obiettivo chiave: **web-first** (desktop via browser) + **mobile-first** (iOS/Android) con **offline-first** sul mobile e sincronizzazione automatica quando torna online.

---

## Indice

- [Perché TightWallet](#perché-tightwallet)
- [Concetti chiave](#concetti-chiave)
- [Classic vs Advanced](#classic-vs-advanced)
- [Funzionalità principali](#funzionalità-principali)
- [Architettura](#architettura)
- [Offline-first & Sync](#offline-first--sync)
- [Multi-valuta](#multi-valuta)
- [Dashboard, KPI e Budget](#dashboard-kpi-e-budget)
- [Import / Export](#import--export)
- [Database](#database)
- [Struttura repository (proposta)](#struttura-repository-proposta)
- [Roadmap](#roadmap)
- [Contribuire](#contribuire)
- [Glossario](#glossario)

---

## Perché TightWallet

Molte app “automatiche” (soprattutto con open banking) sono comode, ma spesso:
- categorizzano male e generano dati sporchi
- non hanno un modello contabile rigoroso
- non funzionano bene offline
- rendono difficile migrare i dati altrove

TightWallet parte da un’idea diversa:

> **Tu controlli i dati. L’app ti aiuta a registrarli bene e a capirli meglio.**

---

## Concetti chiave

### Conti (Accounts) e Chart of Accounts
I conti seguono la struttura contabile standard stile GnuCash:

- **Assets** (Attività): ciò che possiedi (banca, contanti, investimenti…)
- **Liabilities** (Passività): ciò che devi (carta di credito, prestiti…)
- **Equity**: patrimonio netto e saldi di apertura
- **Income**: entrate
- **Expenses**: spese

Per l’utente Classic questa struttura può essere “nascosta”, ma **esiste sempre**, perché rende solidi saldi e report.

### Partita doppia (Double-entry)
Ogni transazione ha almeno due righe (**splits**) e deve essere bilanciata:

- una o più righe in **Dare** (debit)
- una o più righe in **Avere** (credit)

Esempio: caffè 1€ pagato in contanti
- `Expenses:Coffee` **debit** 1
- `Assets:Cash` **credit** 1

Classic non deve vedere “Dare/Avere”, ma il motore sì.

### Saldi iniziali (Opening Balances)
Quando imposti un saldo iniziale, TightWallet crea una transazione di apertura con controparte `Equity:Opening Balances`.
Questo evita “scorciatoie” e mantiene il libro coerente.

---

## Classic vs Advanced

### Classic (utente semplice)
Cosa vede:
- Dashboard a colpo d’occhio (saldo, spese/entrate mese, trend)
- Lista conti principali (Banca / Carta / Contanti) con saldo
- Lista movimenti (tabellare) con filtri e ricerca
- Inbox ricorrenti (pagamenti attesi da approvare)
- Budget semplice (opzionale): obiettivo risparmio + avvisi

Cosa fa:
- Aggiunge **Spesa / Entrata / Trasferimento** in pochi tap
- Seleziona sempre:
  - **conto** (da cui paga / su cui entra)
  - **categoria** (in UI), che internamente è un conto `Expense` o `Income`
- Approva o modifica ricorrenti (anche variabili: Telepass/bollette)

Cosa non deve vedere:
- Dare/Avere
- splits multipli (se esistono, li vede “riassunti” con dettaglio apribile)

### Advanced (utente avanzato)
Cosa vede:
- Albero conti completo (Assets/Liabilities/Equity/Income/Expenses)
- Ledger per conto (movimenti del conto)
- Editor transazione completo con splits multipli
- Report contabili: Balance Sheet e Profit & Loss
- Multi-valuta completa (cambi, FX gain/loss)

> Classic e Advanced condividono lo stesso database: cambia la **vista**, non i dati.

---

## Funzionalità principali

### Inserimento “veloce ma corretto”
- Spesa / Entrata / Trasferimento (Classic)
- Transazioni complesse con ripartizioni (Advanced)

### Ricorrenti “automatici ma supervisionati” (feature chiave)
- Definisci una **Scheduled Rule** (es. Telepass, bolletta, rata)
- Ogni periodo viene generata una **Expected Instance** nell’Inbox
- Tu **approvi / modifichi / rimandi**
- Solo dopo approvazione diventa transazione reale

Strategie importo per il variabile:
- `Fixed`
- `LastMonth`
- `AverageN`
- `Estimate + Reconcile`
- `Schedule Table` (rate/prestiti)

### Payee canonico + alias (dati puliti)
Per evitare il caos “Gas/gas/metano”:
- `Payee` canonico (merchant/fornitore)
- alias e suggerimenti/autocomplete
- trend e report più affidabili

---

## Architettura

### Componenti
- **Backend API**: dominio contabile + persistenza + import + ricorrenze + KPI + sync
- **Web App**: desktop via browser (import/export, analisi, dashboard)
- **Mobile App**: UX rapida, biometria, offline-first con DB locale

---

## Offline-first & Sync

Requisito: sul mobile l’app deve funzionare senza internet.

Approccio:
- DB locale (es. SQLite)
- ogni azione genera un evento in **outbox** locale
- quando online:
  - invio eventi **idempotenti**
  - delta sync dal server
  - gestione conflitti (MVP: last-write-wins + log conflitti)

---

## Multi-valuta

Modello “alla GnuCash”:
- la **valuta è proprietà del conto**
- il libro ha una **base currency** (per reporting aggregato)
- nei `splits` si salva:
  - `amount` (in valuta del conto)
  - `value_base` (in valuta base, per aggregazione/report)

Trasferimenti cross-currency:
- richiedono un **tasso di cambio** (manuale o da lista salvata)
- opzionale: differenze su conto `FX Gain/Loss`

---

## Dashboard, KPI e Budget

### Dashboard componibile (widget/card)
Esempi di widget:
- saldo totale (in base currency)
- saldi conti principali
- spese/entrate/risparmio mese
- top categorie
- trend 6/12 mesi
- ricorrenti: “fisso del mese” + Inbox pagamenti attesi
- utilities (gas/luce/acqua): costo e (opzionale) consumo

### KPI base (MVP)
- Income (Entrate)
- Expenses (Spese)
- Net Savings (Entrate − Spese)
- Savings Rate (% risparmio)
- Burn Rate (spesa media giornaliera)
- Forecast fine mese (MVP)

### Budget
- Budget per categoria (se l’utente vuole)
- Budget “automatico”: obiettivo risparmio mensile (Savings Target)

---

## Import / Export

### Import (priorità alta)
Sorgenti iniziali:
- **GnuCash**
- **Money Manager**
- CSV / Excel (adattatori)

Pipeline import robusta:
1) upload → 2) parsing → 3) mapping → 4) normalizzazione → 5) preview → 6) commit atomico → 7) log

### Export (step successivo, ma progettato)
- CSV / Excel per transazioni e report
- PDF report mensile/annuale
- JSON backup completo

---

## Database

Target consigliato: **PostgreSQL**.

### Modello dati (overview)
Entità principali:
- Utenti e impostazioni: `users`, `book_settings`, `user_settings`, `auth_identities`
- Piano dei conti: `ledger_accounts`
- Movimenti contabili: `transactions` (testa), `splits` (righe)
- Normalizzazione merchant: `payees`, `payee_aliases`
- Multi-valuta: `exchange_rates`, `splits.value_base`, `splits.fx_rate_id`
- Ricorrenti: `scheduled_rules`, `expected_instances`, (opzionale) `loan_schedules`, `loan_schedule_items`
- Budget e obiettivi: `budgets`, `savings_targets`
- Dashboard: `dashboard_widgets`
- Import: `import_jobs`, `import_rows`
- Offline sync (server-side): `devices`, `change_events`, `operation_log`, `sync_cursors`

### Note di design importanti
- **Vincolo partita doppia**: `Σ(debit.value_base) = Σ(credit.value_base)` per ogni `transaction_id`.
  - consigliato: validazione applicativa + transazione DB; trigger opzionale solo per safety.
- **Currency**:
  - `ledger_accounts.currency` è **richiesta** per conti `asset`/`liability`
  - può essere `NULL` per `income`/`expense`
- **Carta (credit card)**:
  - modello consigliato: **Liability** (più corretto)
  - in Classic va presentata come “Carta” senza parlare di debito

### Schema SQL (DDL) – riferimento completo

> Nota: questo schema è il “target”. Nel repo andrà gestito tramite migrazioni in `db/migrations/`.

```sql
-- USERS
CREATE TABLE users (
  id              UUID PRIMARY KEY,
  email           TEXT UNIQUE NOT NULL,
  password_hash   TEXT NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- BOOK SETTINGS (base currency)
CREATE TABLE book_settings (
  user_id         UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  base_currency   TEXT NOT NULL DEFAULT 'EUR',
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- USER SETTINGS (classic/advanced, language, theme)
CREATE TABLE user_settings (
  user_id           UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  preferred_mode    TEXT NOT NULL DEFAULT 'classic' CHECK (preferred_mode IN ('classic','advanced')),
  preferred_lang    TEXT NOT NULL DEFAULT 'system', -- system|it|en
  preferred_theme   TEXT NOT NULL DEFAULT 'system', -- system|light|dark
  updated_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- (optional) identities: google/apple/etc.
CREATE TABLE auth_identities (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  provider        TEXT NOT NULL,
  provider_uid    TEXT NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(provider, provider_uid)
);

-- EXCHANGE RATES
CREATE TABLE exchange_rates (
  id              UUID PRIMARY KEY,
  rate_date       DATE NOT NULL,
  from_currency   TEXT NOT NULL,
  to_currency     TEXT NOT NULL,
  rate            NUMERIC(18,8) NOT NULL CHECK (rate > 0),
  source          TEXT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(rate_date, from_currency, to_currency)
);

-- CHART OF ACCOUNTS
CREATE TABLE ledger_accounts (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  parent_id       UUID NULL REFERENCES ledger_accounts(id) ON DELETE SET NULL,
  name            TEXT NOT NULL,
  kind            TEXT NOT NULL CHECK (kind IN ('asset','liability','equity','income','expense')),
  currency        TEXT NULL, -- REQUIRED for asset/liability; NULL ok for income/expense
  is_archived     BOOLEAN NOT NULL DEFAULT FALSE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, parent_id, name)
);

CREATE INDEX idx_ledger_user_kind ON ledger_accounts(user_id, kind);

-- PAYEES (canonical merchant) + aliases
CREATE TABLE payees (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  canonical_name  TEXT NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(user_id, canonical_name)
);

CREATE TABLE payee_aliases (
  id              UUID PRIMARY KEY,
  payee_id        UUID NOT NULL REFERENCES payees(id) ON DELETE CASCADE,
  alias           TEXT NOT NULL,
  UNIQUE(payee_id, alias)
);

-- TRANSACTIONS (header)
CREATE TABLE transactions (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  txn_date        DATE NOT NULL,
  description     TEXT NULL,
  payee_id        UUID NULL REFERENCES payees(id) ON DELETE SET NULL,
  source          TEXT NOT NULL DEFAULT 'manual'
                  CHECK (source IN ('manual','import','recurring','adjustment','opening')),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_tx_user_date ON transactions(user_id, txn_date);

-- SPLITS (double-entry lines)
-- amount: in account currency; value_base: in base currency for reporting/aggregation
CREATE TABLE splits (
  id              UUID PRIMARY KEY,
  transaction_id  UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
  account_id      UUID NOT NULL REFERENCES ledger_accounts(id) ON DELETE RESTRICT,
  direction       TEXT NOT NULL CHECK (direction IN ('debit','credit')),
  amount          NUMERIC(18,2) NOT NULL CHECK (amount >= 0),
  value_base      NUMERIC(18,2) NOT NULL CHECK (value_base >= 0),
  fx_rate_id      UUID NULL REFERENCES exchange_rates(id) ON DELETE SET NULL,
  memo            TEXT NULL
);

CREATE INDEX idx_splits_txn ON splits(transaction_id);
CREATE INDEX idx_splits_account ON splits(account_id);

-- TAGS
CREATE TABLE tags (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name            TEXT NOT NULL,
  UNIQUE (user_id, name)
);

CREATE TABLE transaction_tags (
  transaction_id  UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
  tag_id          UUID NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (transaction_id, tag_id)
);

-- ATTACHMENTS
CREATE TABLE attachments (
  id              UUID PRIMARY KEY,
  transaction_id  UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
  storage_ref     TEXT NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- UTILITIES METRICS (optional)
CREATE TABLE transaction_metrics (
  id              UUID PRIMARY KEY,
  transaction_id  UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
  metric_type     TEXT NOT NULL, -- gas_m3|electric_kwh|water_m3...
  metric_value    NUMERIC(18,4) NOT NULL,
  unit            TEXT NOT NULL,
  UNIQUE(transaction_id, metric_type)
);

-- SCHEDULED RULES (recurring)
CREATE TABLE scheduled_rules (
  id                  UUID PRIMARY KEY,
  user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name                TEXT NOT NULL,
  pay_account_id      UUID NOT NULL REFERENCES ledger_accounts(id),
  counter_account_id  UUID NOT NULL REFERENCES ledger_accounts(id),
  frequency           TEXT NOT NULL CHECK (frequency IN ('weekly','monthly','yearly','custom')),
  interval_n          INT  NOT NULL DEFAULT 1,
  day_of_month        INT  NULL CHECK (day_of_month BETWEEN 1 AND 28),
  start_date          DATE NOT NULL,
  end_date            DATE NULL,
  amount_strategy     TEXT NOT NULL CHECK (amount_strategy IN ('fixed','last_month','avg_n','estimate_reconcile','schedule_table')),
  fixed_amount        NUMERIC(18,2) NULL,
  avg_n               INT NULL,
  estimate_amount     NUMERIC(18,2) NULL,
  tolerance_amount    NUMERIC(18,2) NOT NULL DEFAULT 2.00,
  is_active           BOOLEAN NOT NULL DEFAULT TRUE,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- EXPECTED INSTANCES (Inbox items)
CREATE TABLE expected_instances (
  id              UUID PRIMARY KEY,
  rule_id         UUID NOT NULL REFERENCES scheduled_rules(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  due_date        DATE NOT NULL,
  proposed_amount NUMERIC(18,2) NOT NULL,
  status          TEXT NOT NULL DEFAULT 'pending'
                  CHECK (status IN ('pending','approved','matched','skipped','expired')),
  approved_at     TIMESTAMPTZ NULL,
  matched_txn_id  UUID NULL REFERENCES transactions(id) ON DELETE SET NULL,
  note            TEXT NULL
);

CREATE INDEX idx_expected_user_due ON expected_instances(user_id, due_date);

-- (optional, loans) piano rate importabile
CREATE TABLE loan_schedules (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  rule_id         UUID NULL REFERENCES scheduled_rules(id) ON DELETE SET NULL,
  name            TEXT NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE loan_schedule_items (
  id              UUID PRIMARY KEY,
  schedule_id     UUID NOT NULL REFERENCES loan_schedules(id) ON DELETE CASCADE,
  due_date        DATE NOT NULL,
  amount          NUMERIC(18,2) NOT NULL,
  principal       NUMERIC(18,2) NULL,
  interest        NUMERIC(18,2) NULL
);

-- BUDGETS
CREATE TABLE budgets (
  id                  UUID PRIMARY KEY,
  user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  expense_account_id  UUID NOT NULL REFERENCES ledger_accounts(id),
  period              TEXT NOT NULL, -- YYYY-MM
  amount              NUMERIC(18,2) NOT NULL CHECK (amount >= 0),
  UNIQUE (user_id, expense_account_id, period)
);

-- SAVINGS TARGET
CREATE TABLE savings_targets (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  period          TEXT NOT NULL, -- YYYY-MM
  target_amount   NUMERIC(18,2) NOT NULL CHECK (target_amount >= 0),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, period)
);

-- DASHBOARD WIDGETS (layout)
CREATE TABLE dashboard_widgets (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  widget_type     TEXT NOT NULL,
  position        INT  NOT NULL,
  config_json     JSONB NOT NULL DEFAULT '{}'::jsonb,
  UNIQUE (user_id, position)
);

-- IMPORT PIPELINE
CREATE TABLE import_jobs (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  source_type     TEXT NOT NULL CHECK (source_type IN ('gnucash','money_manager','csv','xlsx')),
  filename        TEXT NOT NULL,
  status          TEXT NOT NULL DEFAULT 'uploaded'
                  CHECK (status IN ('uploaded','parsed','mapped','previewed','committed','failed')),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  error_message   TEXT NULL
);

CREATE TABLE import_rows (
  id              UUID PRIMARY KEY,
  import_job_id   UUID NOT NULL REFERENCES import_jobs(id) ON DELETE CASCADE,
  raw_json        JSONB NOT NULL,
  row_hash        TEXT NULL,
  status          TEXT NOT NULL DEFAULT 'new'
                  CHECK (status IN ('new','duplicate','mapped','error')),
  mapped_txn_id   UUID NULL REFERENCES transactions(id) ON DELETE SET NULL,
  error_message   TEXT NULL
);

CREATE INDEX idx_import_rows_job ON import_rows(import_job_id);

-- OFFLINE SYNC SUPPORT (server)
CREATE TABLE devices (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  device_name     TEXT NULL,
  platform        TEXT NOT NULL, -- ios|android|web
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_seen_at    TIMESTAMPTZ NULL
);

CREATE TABLE change_events (
  id              BIGSERIAL PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  entity_type     TEXT NOT NULL,
  entity_id       UUID NULL,
  op_type         TEXT NOT NULL,
  payload         JSONB NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_events_user_id ON change_events(user_id, id);

CREATE TABLE operation_log (
  id              UUID PRIMARY KEY, -- operation_id from device
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  device_id       UUID NULL REFERENCES devices(id) ON DELETE SET NULL,
  op_type         TEXT NOT NULL,
  entity_type     TEXT NOT NULL,
  entity_id       UUID NULL,
  applied_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE sync_cursors (
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  device_id       UUID NOT NULL REFERENCES devices(id) ON DELETE CASCADE,
  last_event_id   BIGINT NOT NULL DEFAULT 0,
  last_sync_at    TIMESTAMPTZ NULL,
  PRIMARY KEY (user_id, device_id)
);
```

---

## Struttura repository (proposta)

```
TightWallet/
  README.md
  docs/
    product/
      spec-classic-vs-advanced.md
      recurring.md
      multicurrency.md
      offline-sync.md
      kpi.md
    architecture/
      data-model.md
      api-contracts.md
  backend/
    src/
    tests/
    Dockerfile
  web/
    src/
    tests/
  mobile/
    src/
    tests/
  db/
    migrations/
    seed/
  tools/
    importers/
      gnucash/
      money-manager/
  .github/
    workflows/
  LICENSE
```

---

## Roadmap

### Fase 1 (Core)
- ledger_accounts + transactions + splits (partita doppia)
- template conti + opening balances
- Classic/Advanced + preferenza salvata
- import GnuCash + Money Manager (staging + preview)
- dashboard base + vista tabellare + KPI base

### Fase 2 (Differenziante)
- scheduled rules + inbox (variabili incluse)
- budget obiettivo risparmio
- payee normalization + utilities trend
- offline-first mobile + sync

### Fase 3 (Espansioni)
- export completo (CSV/Excel/PDF/JSON)
- report “chiusura mese”
- AI/chat con i tuoi dati (con governance privacy)
- open banking opzionale (non requisito)

---

## Contribuire

- Apri una issue con:
  - contesto (Classic/Advanced)
  - scenario riproducibile (passi)
  - risultato atteso vs ottenuto
- PR piccole e focalizzate (una feature/bug per PR)
- Requisiti non negoziabili:
  - transazioni sempre bilanciate
  - multi-valuta coerente (amount + value_base)
  - ricorrenti e import non devono creare duplicati

---

## Glossario

- **Account/Conto**: contenitore contabile (asset, liability, income, expense…)
- **Split**: riga di una transazione (debit o credit)
- **Payee**: merchant/fornitore normalizzato (canonico)
- **Base currency**: valuta del libro per report aggregati
- **Scheduled rule**: regola programmata (ricorrenza)
- **Expected instance**: pagamento atteso in Inbox
- **Offline-first**: funziona senza rete; sync quando torna online
- **Ledger**: lista movimenti di un conto
