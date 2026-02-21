# TightWallet – Documento Analitico (Funzionalità + Architettura + KPI + Bozza DB)

> Obiettivo: app “semplice per tutti” con base **a partita doppia** in stile **GnuCash**, ma moderna; import dati da altri programmi (inizialmente **GnuCash** e **Money Manager**), dashboard componibile dall’utente, web-first con API per app mobile.

---

## 1) Visione e principi

**TightWallet** è un *libro mastro personale moderno*:
- **Inserimento dati** coerente con GnuCash (conti + scritture, non “magia” bancaria obbligatoria).
- **Partita doppia** come fondamento: ogni movimento è sempre bilanciato.
- **Semplicità in UI**: l’utente vede “spesa/entrata/trasferimento”; il sistema mantiene la contabilità corretta.
- **Portabilità dati**: import (subito) da altri software; export (secondo momento) per evitare lock-in.
- **Multi‑piattaforma**: web app sul desktop (browser) + app mobile che chiama gli stessi endpoint.

---

## 2) Logica “alla GnuCash” (ma user‑friendly)

### 2.1 Concetti base (tradotti in UX semplice)
- **Account/Conti**: banca, carta, contanti, debiti, fondo emergenza, ecc.
- **Categorie** (spese/entrate) **modellate come conti** di tipo Expense/Income (come GnuCash).
- **Transazione**: un evento con data/descrizione che contiene una o più **righe** (split) in Dare/Avere.
- **Vincolo**: somma Dare = somma Avere (sempre).

### 2.2 UX minima per utente “non nerd”
Schermata “Aggiungi”:
- Tipo: **Spesa / Entrata / Trasferimento**
- Importo
- Conto (es. “Carta”)
- Categoria (es. “Trasporti → Telepass”)
- Data e descrizione (facoltative)

Dietro le quinte:
- Spesa: `Expense:Telepass (Dare)` + `Assets:Carta (Avere)`
- Entrata: `Assets:Banca (Dare)` + `Income:Stipendio (Avere)`
- Trasferimento: `Assets:Banca (Dare)` + `Assets:Contanti (Avere)`

### 2.3 Nerd mode (opzionale)
- Visualizzazione **splits** e conti contabili (Dare/Avere).
- Libro giornale / libro mastro.
- Report contabili (conto economico, stato patrimoniale).

---

## 3) Import dati (primo obiettivo operativo)

### 3.1 Requisito
Importare dati già esistenti per “partire subito” con dashboard e analisi.

### 3.2 Strategia: pipeline di import con “staging” + preview
1. **Caricamento file** (upload) o collegamento a sorgente.
2. **Parsing** verso un formato interno “staging” (righe grezze).
3. **Mapping**: colonne → campi (data/importo/descrizione/conti/categorie).
4. **Normalizzazione**: segni, valute, formati data, categorie.
5. **Preview**: elenco transazioni che verranno create + eventuali warning.
6. **Commit**: creazione transazioni/splits in modo atomico.
7. **Log**: report errori, duplicati, righe scartate.

### 3.3 Sorgenti iniziali
- **GnuCash**: import di conti + transazioni + splits.
- **Money Manager**: import da export CSV/DB tramite adapter dedicato.

> Export “verso fuori” (CSV/Excel/PDF/JSON) resta un secondo step, ma il DB è pensato per farlo senza dolore.

---

## 4) Il problema “che ti dà noia”: pagamenti mensili da ricordare

Questa è una feature ad alto valore: **processo automatico supervisionato**.

### 4.1 Tipi di pagamenti ricorrenti
1. **Fissi** (stesso importo): abbonamenti, affitto.
2. **Variabili prevedibili**: bollette, Telepass (quasi sempre cambia).
3. **Variabili strutturate**: rate/finanziamenti che cambiano quota interessi/capitale.

### 4.2 Soluzione: “Transazioni Programmate” + “Inbox di Revisione”
- L’utente crea una **Regola Ricorrente** (scheduled transaction):
  - nome (es. “Telepass”)
  - conto di pagamento (es. Carta)
  - categoria/controconto (Trasporti → Pedaggi)
  - periodicità (mensile)
  - **strategia importo** (vedi sotto)
- Ogni periodo il sistema genera una **Istanza Attesa** (expected instance) che finisce in una Inbox:
  - l’utente la **approva**, la **modifica** (importo/data), o la **rimanda**
  - se approvata, diventa una transazione reale (partita doppia)

### 4.3 Strategie importo (per gestire il variabile)
Per ogni regola ricorrente, scegli una strategia:

- **Fixed**: importo fisso.
- **LastMonth**: propone l’ultimo importo e tu lo aggiusti (Telepass/bollette).
- **AverageN**: media degli ultimi N mesi.
- **Estimate + Reconcile**: inserisci una *stima*; quando arriva l’importo reale:
  - il sistema registra la differenza su un conto “Aggiustamenti” **oppure** aggiorna la transazione (scelta UX).
- **Planned Schedule** (prestiti): importi pre-caricati (tabella rate), anche importabile.

### 4.4 Riconciliazione automatica (quando importi estratti conto)
Quando importate movimenti (banca/carta):
- matching tra transazione importata e istanza attesa (merchant/descrizione + importo vicino + finestra date)
- se match:
  - istanza diventa “matched”
  - collegamento alla transazione reale
  - gestione differenze importo (se strategia lo prevede)

Risultato: non devi più “ricordarti tutto”, ma **revisionare**.

---

## 5) Dashboard componibile (utente sceglie cosa vedere)

### 5.1 Layout a card (widget)
- aggiungi/rimuovi widget, riordina, salva layout
- ogni widget configurabile (periodo, conto, categoria, tag)

Widget tipici:
- Saldo totale + saldi conti principali
- Spese del mese + confronto col mese precedente
- Top categorie mese
- Ricorrenti: previsione “spesa fissa del mese”
- Budget: stato per categoria (semaforo)
- Trend 6/12 mesi
- Inbox pagamenti attesi

### 5.2 Vista tabellare (power user)
- elenco transazioni filtrabile (data, conto, categoria, tag, importo, sorgente)
- ricerca su descrizione
- (export dalla vista: secondo step)

---

## 6) Architettura piattaforma (web-first + API)

### 6.1 Struttura consigliata
- **Web app** = UI principale (desktop + mobile web)
- **API backend** = dominio + persistenza (auth, conti, transazioni, import, ricorrenze, KPI)
- **App mobile** = client che usa le API + integrazioni device (notifiche, widget)

Vantaggi:
- una sola logica “seria” (partita doppia)
- mobile più semplice (solo chiamate API)
- evoluzione più rapida

---

## 7) KPI (spiegati e come calcolarli)

> KPI calcolati per periodo e filtro (tutti i conti o subset). Con partita doppia, si calcolano aggregando i conti Income/Expense/Asset/Liability.

### 7.1 KPI base
**Totale Entrate (Income)**  
- Cosa misura: entrate nel periodo  
- Calcolo: somma movimenti sui conti `income`  
- Formula logica: `Income = Σ credit su conti income`

**Totale Spese (Expenses)**  
- Cosa misura: spese nel periodo  
- Calcolo: somma movimenti sui conti `expense`  
- Formula logica: `Expenses = Σ debit su conti expense`

**Risparmio Netto (Net Savings)**  
- `NetSavings = Income - Expenses`

**Savings Rate**  
- `SavingsRate = (NetSavings / Income) * 100` (se Income=0 → N/A)

### 7.2 KPI operativi (quelli “utili davvero”)
**Burn Rate (spesa media giornaliera)**  
- `BurnRate = Expenses / Days`  
- Days = giorni del mese (consuntivo) o trascorsi (mese corrente)

**Forecast spesa fine mese**  
- `ForecastExpenses = BurnRate * DaysInMonth`

**Forecast saldo fine mese (MVP)**  
- `ForecastBalance = CurrentBalance - (BurnRate * RemainingDays)`  
- Evoluzione: aggiungere entrate attese + ricorrenti fissi

**Ricorrenti vs variabili**  
- richiede flag ricorrente o derivazione dalle scheduled_rules  
- `RecurringExpenses = Σ spese ricorrenti`  
- `VariableExpenses = Expenses - RecurringExpenses`  
- `%Ricorrenti = (RecurringExpenses / Expenses) * 100`

**Costo mensile normalizzato (annuali/periodiche)**  
- `NormalizedMonthly = Amount / CoverageMonths`  
- utile per stime e budget “realistici”

**Distribuzione per categoria**  
- `CatSpend[c] = Σ spese in categoria c`  
- `% = CatSpend[c] / Expenses * 100`

**Trend mensile (6/12 mesi)**  
- serie di Income/Expenses/NetSavings per mese + delta %

---

## 8) Categorie consigliate (per partire bene)

### Entrate
- Stipendio (netto/bonus)
- Rimborsi (note spese/sanitario)
- Vendite (usato/prestazioni)
- Interessi/Rendite
- Regali/Cashback

### Spese
- Casa (affitto/mutuo, bollette, condominio, manutenzione)
- Alimentari (supermercato, spesa rapida)
- Trasporti (carburante, Telepass/pedaggi, parcheggi, mezzi, manutenzione)
- Abbonamenti & servizi (streaming, telefonia, software)
- Salute (farmacia, visite, palestra)
- Tempo libero (ristoranti, eventi, hobby)
- Shopping (abbigliamento, elettronica, casa)
- Viaggi (trasporti, alloggi, spese)
- Tasse & commissioni (bancarie, imposte)
- Donazioni
- Altro (da usare poco, con suggerimento riclassifica)

### Trasferimenti
- Giroconto / trasferimento tra conti
- Accantonamento verso fondi

---

# 9) Bozza database (tabelle, colonne, PK/FK) – completa

## 9.1 DDL (PostgreSQL) – bozza

```sql
-- USERS
CREATE TABLE users (
  id              UUID PRIMARY KEY,
  email           TEXT UNIQUE NOT NULL,
  password_hash   TEXT NOT NULL,
  is_pro          BOOLEAN NOT NULL DEFAULT FALSE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- LEDGER ACCOUNTS (piano dei conti unificato stile GnuCash)
CREATE TABLE ledger_accounts (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  parent_id       UUID NULL REFERENCES ledger_accounts(id) ON DELETE SET NULL,
  name            TEXT NOT NULL,
  kind            TEXT NOT NULL CHECK (kind IN ('asset','liability','equity','income','expense')),
  currency        TEXT NULL,               -- utile per asset/liability (es. EUR)
  is_archived     BOOLEAN NOT NULL DEFAULT FALSE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, parent_id, name)
);

-- TRANSACTIONS (testa)
CREATE TABLE transactions (
  id              UUID PRIMARY KEY,
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  txn_date        DATE NOT NULL,
  description     TEXT NULL,
  source          TEXT NOT NULL DEFAULT 'manual' CHECK (source IN ('manual','import','recurring','adjustment')),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_transactions_user_date ON transactions(user_id, txn_date);

-- SPLITS (righe: partita doppia)
CREATE TABLE splits (
  id              UUID PRIMARY KEY,
  transaction_id  UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
  account_id      UUID NOT NULL REFERENCES ledger_accounts(id) ON DELETE RESTRICT,
  direction       TEXT NOT NULL CHECK (direction IN ('debit','credit')),
  amount          NUMERIC(18,2) NOT NULL CHECK (amount >= 0),
  memo            TEXT NULL
);

CREATE INDEX idx_splits_txn ON splits(transaction_id);
CREATE INDEX idx_splits_account ON splits(account_id);

-- Vincolo Dare=Avere: consigliato in validazione applicativa + transazione DB.
-- (opzionale) Trigger di check per transazioni "complete".

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

-- SCHEDULED RULES (ricorrenze)
CREATE TABLE scheduled_rules (
  id                  UUID PRIMARY KEY,
  user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name                TEXT NOT NULL,
  pay_account_id      UUID NOT NULL REFERENCES ledger_accounts(id), -- es. Assets:Carta
  counter_account_id  UUID NOT NULL REFERENCES ledger_accounts(id), -- es. Expense:Telepass / Liability:Loan
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

-- EXPECTED INSTANCES (Inbox pagamenti attesi)
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

CREATE INDEX idx_expected_user_duedate ON expected_instances(user_id, due_date);

-- (opzionale, per prestiti) piano rate importabile
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

-- BUDGETS (su conti expense)
CREATE TABLE budgets (
  id                  UUID PRIMARY KEY,
  user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  expense_account_id  UUID NOT NULL REFERENCES ledger_accounts(id),
  period              TEXT NOT NULL, -- 'YYYY-MM'
  amount              NUMERIC(18,2) NOT NULL CHECK (amount >= 0),
  UNIQUE (user_id, expense_account_id, period)
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

-- IMPORT JOBS (pipeline import + staging)
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
  row_hash        TEXT NULL, -- hash per duplicati (date+amount+desc)
  status          TEXT NOT NULL DEFAULT 'new'
                  CHECK (status IN ('new','duplicate','mapped','error')),
  mapped_txn_id   UUID NULL REFERENCES transactions(id) ON DELETE SET NULL,
  error_message   TEXT NULL
);

CREATE INDEX idx_import_rows_job ON import_rows(import_job_id);
```

---

## 10) Roadmap (coerente con la vostra strategia)

**Step 1 – Fondamenta (GnuCash-like)**
- ledger_accounts + transactions + splits
- UI semplice per inserimento
- import GnuCash + Money Manager con staging/preview

**Step 2 – “Non voglio ricordarmi ogni mese”**
- scheduled_rules + expected_instances (Inbox)
- matching da import e gestione variabili

**Step 3 – Dashboard componibile + KPI**
- widgets + configurazione
- KPI base + trend + budget

**Step 4 – Export/report**
- export CSV/Excel/PDF/JSON
- chiusura mese + report
