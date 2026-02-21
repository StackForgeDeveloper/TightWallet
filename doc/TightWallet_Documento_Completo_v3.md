# TightWallet – Documento completo (Visione, Funzionalità, Architettura, KPI, Budget, Multi‑Valuta, Offline, DB)

> Versione: v3 (integra anche multi‑valuta “alla GnuCash”, budget più automatico, offline‑first e tutti gli spunti iniziali)

---

## 0) Obiettivo in una frase
Costruire una app moderna “alla GnuCash” (**conti + partita doppia**) che consenta **import** dai tool già usati (GnuCash e Money Manager), offra **dashboard componibile**, **KPI/grafici**, funzioni **offline** con sync trasparente, e abbia un modello di monetizzazione semplice (**ads → 2€/mese no‑ads**).

---

## 1) Posizionamento e principi

### 1.1 Cosa vogliamo (e cosa no)
**Vogliamo**
- Un sistema **a partita doppia vero** (non “solo categorie”).
- **Web app** (desktop) + **app mobile** (iOS/Android) che usa gli stessi endpoint.
- **Import/Export** (Excel/CSV) per portabilità (import prima, export come step successivo).
- **Dashboard a colpo d’occhio**, personalizzabile con card + vista tabellare.
- **Tema chiaro/scuro** seguendo il dispositivo.
- Integrazione UI curata (tenere d’occhio **Liquid Glass** Apple e tema scuro).
- **Offline-first** sul mobile (e possibilmente PWA su web), con sync “seamless”.

**Non vogliamo (per ora)**
- Dipendere dall’open banking/PSD2 come requisito (può diventare opzione futura).
- Obbligare l’utente a capire Dare/Avere in modalità base.

### 1.2 Modalità d’uso (scelta salvata)
- **Classic** (Base): inserimento semplice, niente termini contabili.
- **Advanced**: mostra splits, ledger, report contabili.
- La scelta Classic/Advanced è una **preferenza utente** persistente (non va risettata ad ogni avvio).

---

## 2) Fondamenta contabili: conti e partita doppia

### 2.1 Chart of Accounts (conti tipici)
Struttura consigliata (personalizzabile):

**Balance Sheet (Bilancio)**
- **Assets** (Attività): Cash, Bank, Investments, Fixed Assets…
- **Liabilities** (Passività): Credit Card, Loans, Accounts Payable…
- **Equity** (Patrimonio netto): *Opening Balances*, Retained Earnings…

**Profit & Loss**
- **Income** (Entrate): Salary, Interest, Other Income…
- **Expenses** (Spese): Utilities, Groceries, Transport, Subscriptions, Taxes…

> Nota: in questa logica, le “categorie” sono **conti Expense/Income** (stile GnuCash).  
> È il modo più robusto per avere reporting serio e dati coerenti.

### 2.2 Opening Balances (inizializzazione corretta)
Quando crei un conto con saldo iniziale, si registra una transazione “Apertura”:

- Dare: `Assets:Bank` 1.000 EUR  
- Avere: `Equity:Opening Balances` 1.000 EUR

Questo evita scorciatoie e mantiene il sistema “quadrato”.

### 2.3 Tipi di transazione (semplificati per Classic)
- **Spesa**: da un Asset (banca/carta/contanti) verso un Expense
- **Entrata**: da un Income verso un Asset
- **Trasferimento**: Asset → Asset (o Liability, nei casi di rimborso debiti)

---

## 3) Multi‑valuta (modello corretto “alla GnuCash”)

### 3.1 Regola mentale (semplice ma precisa)
- **La valuta è proprietà del conto (Account)**.
- Le categorie/Expense/Income **non impongono** la valuta.
- Esiste una **valuta base del libro** (per reporting totale).

Esempio:
- `Assets:Cash EUR` (valuta EUR)
- `Assets:Cash USD` (valuta USD)
- `Expenses:Coffee` (neutra: è un conto expense, non “EUR” o “USD”)

### 3.2 Transazione in valuta diversa
Se sei negli USA e paghi caffè 3 USD con `Cash USD`:

- Dare: `Expenses:Coffee` +3.00 USD  
- Avere: `Assets:Cash USD` -3.00 USD

Il reporting può convertire in EUR (valuta base del book) usando un tasso di cambio.

### 3.3 Reporting: “book currency”
Impostiamo una **base currency** (tipicamente EUR).
Per calcolare “patrimonio totale” con conti in USD/EUR:
- o converti in EUR con tasso di cambio (oggi o storico)
- o mostri per valuta (se l’utente preferisce)

### 3.4 Trasferimenti tra valute (cross‑currency)
Esempio: da `Cash EUR` a `Cash USD`:

- `Cash EUR` -100 EUR  
- `Cash USD` +108 USD  
- tasso: 1 EUR = 1.08 USD

Per rendere il bilancio coerente in base currency:
- si salva il tasso usato
- eventuali differenze vanno in un conto `Income/Expense:FX Gain/Loss` (opzionale, ma corretto)

### 3.5 Scelte MVP (per non complicarsi subito)
- Valuta per conto: **obbligatoria** per Asset/Liability.
- Base currency per utente/book: **obbligatoria**.
- Per cross‑currency: MVP richiede **inserire il cambio** (manuale o da lista salvata).  
  (Alternativa “ignorare il cambio” = reporting totale sbagliato: sconsigliato)

---

## 4) Inserimento dati (stile GnuCash ma UX moderna)

### 4.1 Classic (Base)
Schermata “Aggiungi”:
- Tipo: Spesa / Entrata / Trasferimento
- Importo (in valuta del conto selezionato)
- Conto pagante / ricevente
- Categoria (Expense/Income account)
- Data + descrizione facoltative
- Tag facoltativi

Obiettivo: registrare un movimento in **pochi secondi**.

### 4.2 Advanced
- Editor transazione con più righe (splits multipli)
- Visualizzazione Dare/Avere
- Ledger per conto
- Report: Balance Sheet / Profit&Loss

---

## 5) Import / Export (portabilità)

### 5.1 Import (priorità alta)
Sorgenti iniziali:
- **GnuCash**
- **Money Manager**

Pipeline robusta:
1) upload → 2) parsing → 3) mapping → 4) normalizzazione → 5) preview → 6) commit atomico → 7) log

**Deduplica e riconciliazione**
- hash (data + importo + descrizione + conto)
- suggerimenti merge quando trova “quasi duplicati”

### 5.2 Export (step successivo, ma previsto)
- CSV / Excel per transazioni e report
- PDF report mensile/annuale
- JSON per backup completo

---

## 6) Il “dolore” principale: pagamenti mensili da ricordare

### 6.1 Scheduled Rules + Inbox (automatico supervisionato)
- Crei una regola (Telepass, bolletta gas, rata Findomestic).
- Ogni mese appare una **istanza attesa** in Inbox.
- Tu approvi/modifichi/posticipi.
- Solo dopo approvazione diventa transazione reale.

Questo sposta la fatica da “ricordarsi tutto” a “revisionare”.

### 6.2 Pagamenti variabili: strategie importo
Per ogni regola:
- **Fixed**: importo fisso
- **LastMonth**: propone l’ultimo importo
- **AverageN**: propone media ultimi N mesi
- **Estimate + Reconcile**: stima ora, aggiusta quando arriva il valore reale
- **Schedule Table**: tabella rate (utile per finanziamenti con rate variabili)

### 6.3 Riconciliazione con import (futuro)
Quando importi movimenti banca/carta:
- match con istanze attese (merchant, finestra date, tolleranza importo)
- istanza diventa “matched” automaticamente

---

## 7) Normalizzazione descrizioni (Gas/gas/metano) + trend “Utilities”
Problema: descrizioni incoerenti spezzano i grafici.

Soluzione:
- “Payee canonico” (es. GAS) + alias
- Autocomplete e suggerimenti
- Possibilità di “preferire”/fissare un nome ricorrente

Extra utile (Utilities):
- opzionale: salvare metrica (m³/kWh) nella transazione per grafici di consumo oltre al costo.

---

## 8) Dashboard componibile (a colpo d’occhio) + vista tabellare

### 8.1 Dashboard (card/widget)
- apertura immediata
- layout personalizzabile (drag&drop, abilita/disabilita)
- configurazione per periodo/conti/categorie/tag

Widget tipici:
- Saldo totale (in base currency) + saldi principali
- Spese mese / Entrate mese / Risparmio mese
- Top categorie mese
- Trend 6/12 mesi
- Ricorrenti (quanto “fisso” hai questo mese)
- Inbox (pagamenti attesi)
- Utilities: andamento GAS / luce / acqua (costo e, se disponibile, consumo)

### 8.2 Vista elenco (tabellare)
- filtri: periodo, conto, categoria, tag, payee, importo
- ricerca su descrizione/payee
- editing rapido (Classic) o editor completo (Advanced)

---

## 9) KPI e grafici (spiegati, senza esagerare)

> Calcolo KPI per periodo e filtri. In multi‑valuta: per KPI aggregati si usa **base currency** + tassi salvati.

### 9.1 KPI base
- **Income**: totale entrate nel periodo
- **Expenses**: totale spese nel periodo
- **Net Savings**: Income − Expenses
- **Savings Rate**: (NetSavings / Income) × 100 (se Income=0 → N/A)

### 9.2 KPI operativi
- **Burn Rate**: Expenses / Days
- **Forecast Expenses**: BurnRate × DaysInMonth
- **Forecast Balance (MVP)**: CurrentBalance − BurnRate × RemainingDays
- **Recurring vs Variable**:
  - RecurringExpenses (da regole ricorrenti / categorie marcate)
  - VariableExpenses = Expenses − RecurringExpenses
- **Normalized monthly (annuali)**: Amount / CoverageMonths
- **Category share**: spesa per categoria + percentuale
- **Trend**: serie mensile di Income/Expenses/NetSavings

### 9.3 KPI per “Utilities”
- trend costo GAS (€/mese)
- (opzionale) trend consumo GAS (m³/mese)
- costo unitario (€/m³) se avete metriche

---

## 10) Budget (anche “automatico”, senza che l’utente debba configurare tutto)

### 10.1 Budget classico (per categoria)
- budget mensile per alcune categorie (es. ristoranti 200€)
- semaforo + notifiche

### 10.2 Budget “obiettivo risparmio” (più automatico)
Caso d’uso: “questo mese voglio risparmiare 50€”.

Logica:
- l’utente imposta `TargetSavings = 50€`
- il sistema calcola in tempo reale:
  - risparmio attuale nel mese
  - margine residuo
  - alert quando il trend compromette l’obiettivo

Questo funziona anche se non hai budget per categoria.

---

## 11) Tema, UI e piattaforme (Liquid Glass + scuro)
- Tema **auto** (segue OS), con override manuale.
- Dark mode curato (contrasto, leggibilità, “glow” non eccessivo).
- Tenere in considerazione estetiche Apple moderne (Liquid Glass) per:
  - palette
  - effetti blur/trasparenze
  - icona app

---

## 12) Web + Mobile + Offline‑first (richiesta esplicita)

### 12.1 Architettura
- **Backend API**: partita doppia, import, ricorrenze, KPI, sync.
- **Web app** (browser): esperienza desktop, analisi comoda, import/export.
- **Mobile app**: UI + device features + offline.

### 12.2 Offline “seamless”
- Mobile lavora su DB locale (es. SQLite).
- Ogni modifica genera eventi in **Outbox** locale.
- Quando torna online:
  - invio eventi (idempotenti)
  - pull aggiornamenti server (delta)
  - gestione conflitti (MVP: last-write-wins + log)

---

## 13) Trigger pagamenti (Apple Pay / Google Pay) e integrazioni
- Idea: quando paghi con Apple Pay / Google Pay → apri TightWallet in “quick add”.
- Implementazione realistica MVP:
  - scorciatoie / intent / deep link (dipende da OS)
  - l’autocompilazione completa è spesso limitata dalle API pubbliche
- (futuro) PayPal: login + import movimenti PayPal (integrazione separata, non MVP)

---

## 14) Login e profilo (MVP semplice)
- Inizialmente: **email + password**.
- Strutturare backend/DB per estendere a:
  - email con codice
  - Google/Apple
  - biometria per sblocco (client-side)
  - PayPal (futuro)

---

## 15) Monetizzazione
- **Free**: pubblicità + limiti (es. storico limitato, alcuni widget limitati, export base).
- **Pro 2€/mese**: no ads + storico illimitato + advanced analytics + import/export completo + backup.

---

## 16) Roadmap “step incrementali” (must / nice‑to‑have)

### Fase 1 (Must‑Have – core)
- Partita doppia + chart of accounts con Equity/Opening Balances
- Classic/Advanced + preferenza salvata
- Import GnuCash + Money Manager (staging + preview)
- Dashboard base + vista tabellare
- KPI base (Income/Expenses/Savings) + trend

### Fase 2 (Must‑Have – differenziante)
- Scheduled rules + Inbox (variabili incluse)
- Budget “obiettivo risparmio”
- Normalizzazione payee + trend utilities (GAS ecc.)
- Offline-first mobile + sync

### Fase 3 (Nice‑to‑Have)
- Export completo (Excel/PDF/JSON), report mensile “chiusura mese”
- Chat “con i tuoi dati” (AI) con attenzione a privacy/burocrazia
- Open banking / aggregazione conti (opzionale, non requisito)

---

# 17) Bozza DB completa (PK/FK, multi‑valuta, offline sync, budget)

> Target: PostgreSQL.  
> Nota: per multi‑valuta “vera” conviene salvare nei `splits` sia l’**amount** (valuta del conto) sia il **value_base** (valuta base del book) usato per bilanciare e fare reporting.

```sql
CREATE TABLE users (
  id              UUID PRIMARY KEY,
  email           TEXT UNIQUE NULL,
  password_hash   TEXT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE book_settings (
  user_id         UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  base_currency   TEXT NOT NULL DEFAULT 'EUR',
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE user_settings (
  user_id           UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  preferred_mode    TEXT NOT NULL DEFAULT 'classic' CHECK (preferred_mode IN ('classic','advanced')),
  preferred_lang    TEXT NOT NULL DEFAULT 'system',
  preferred_theme   TEXT NOT NULL DEFAULT 'system',
  default_currency  TEXT NOT NULL DEFAULT 'EUR',
  updated_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE auth_identities (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  provider        TEXT NOT NULL,
  provider_uid    TEXT NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(provider, provider_uid)
);

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

CREATE TABLE ledger_accounts (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  parent_id       UUID NULL REFERENCES ledger_accounts(id) ON DELETE SET NULL,
  name            TEXT NOT NULL,
  kind            TEXT NOT NULL CHECK (kind IN ('asset','liability','equity','income','expense')),
  currency        TEXT NULL,
  is_archived     BOOLEAN NOT NULL DEFAULT FALSE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, parent_id, name)
);

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

CREATE TABLE attachments (
  id              UUID PRIMARY KEY,
  transaction_id  UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
  storage_ref     TEXT NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE transaction_metrics (
  id              UUID PRIMARY KEY,
  transaction_id  UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
  metric_type     TEXT NOT NULL,
  metric_value    NUMERIC(18,4) NOT NULL,
  unit            TEXT NOT NULL,
  UNIQUE(transaction_id, metric_type)
);

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

CREATE TABLE budgets (
  id                  UUID PRIMARY KEY,
  user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  expense_account_id  UUID NOT NULL REFERENCES ledger_accounts(id),
  period              TEXT NOT NULL,
  amount              NUMERIC(18,2) NOT NULL CHECK (amount >= 0),
  UNIQUE (user_id, expense_account_id, period)
);

CREATE TABLE savings_targets (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  period          TEXT NOT NULL,
  target_amount   NUMERIC(18,2) NOT NULL CHECK (target_amount >= 0),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, period)
);

CREATE TABLE dashboard_widgets (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  widget_type     TEXT NOT NULL,
  position        INT  NOT NULL,
  config_json     JSONB NOT NULL DEFAULT '{}'::jsonb,
  UNIQUE (user_id, position)
);

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

-- OFFLINE SYNC support
CREATE TABLE devices (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  device_name     TEXT NULL,
  platform        TEXT NOT NULL,
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

CREATE TABLE operation_log (
  id              UUID PRIMARY KEY,
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

## 18) Nota finale (pragmatica)
Il rischio più grande non è “scrivere codice”, ma:
- partire senza una struttura contabile chiara (Assets/Liabilities/Equity/Income/Expenses)
- sottovalutare multi‑valuta e ricorrenze variabili (che sono la vera differenza)
- fare troppe feature prima di avere un core solido

Se il core è giusto, dashboard/KPI/budget diventano *conseguenze naturali*.
