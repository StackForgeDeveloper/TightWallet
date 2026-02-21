# TightWallet – Documento Funzionale Chiaro (Classic vs Advanced) + Architettura + KPI + DB

> Versione: v4  
> Obiettivo di questa revisione: rendere **cristallino** cosa vede/fa l’utente **Classic (semplice)** e cosa vede/fa l’utente **Advanced (avanzato)**, mantenendo la logica “alla GnuCash” (conti + partita doppia).

---

## 0) Obiettivo in una frase
Un’app moderna “alla GnuCash” (conti + partita doppia) che permetta di **importare subito** i dati esistenti (GnuCash/Money Manager), offra una UI **semplice ma solida**, con una modalità **Advanced** per chi vuole controllo contabile completo, dashboard componibile, KPI, budget, multi‑valuta e offline-first.

---

## 1) Principio chiave: i conti esistono sempre (anche per l’utente semplice)

Anche l’utente Classic, quando registra una spesa, deve scegliere **un conto** da cui escono i soldi (es. “Carta”, “Banca”, “Contanti”).  
Quindi:
- i conti **devono essere pre-configurati** (template iniziale)
- la UI Classic **nasconde** la complessità, ma non la elimina

**Conclusione:** la struttura contabile “Assets/Liabilities/Equity/Income/Expenses” è sempre presente nel modello dati. Cambia *solo* quanto la UI la espone.

---

## 2) Setup iniziale (fondamentale per evitare “è tutto manuale”)

### 2.1 Primo avvio: Wizard di configurazione (Classic-friendly)
Al primo avvio, TightWallet crea una configurazione base e guida l’utente con poche domande:

1) **Valuta base del libro** (es. EUR)  
2) **Conti che possiedi** (toggle + nome):
   - Banca (EUR)
   - Carta (EUR) *(a scelta: come Liability o come Asset; vedi 4.3)*
   - Contanti (EUR)
   - (opzionale) Seconda valuta: Cash USD, ecc.
3) (opzionale) Import storico: GnuCash / Money Manager / CSV

> Obiettivo: l’utente in 60 secondi può già inserire movimenti.

### 2.2 Template Chart of Accounts (creato automaticamente)
TightWallet crea un albero di conti “standard” (personalizzabile) con le 5 macro-categorie:

- **Assets** (positivi)
- **Liabilities** (negativi)
- **Equity**
- **Income**
- **Expenses**

Con sottoconti tipici (ridotti per Classic, espandibili per Advanced).

---

## 3) Classic vs Advanced: cosa vede e cosa può fare

### 3.1 Classic (utente semplice) – “uso quotidiano senza stress”
**Cosa vede**
- **Home/Dashboard** (card): saldo totale, spese mese, entrate mese, risparmio, trend base
- **Conti**: elenco conti principali (Banca/Carta/Contanti) con saldo
- **Movimenti**: lista transazioni filtrabile (vista tabellare)
- **Inbox Ricorrenti**: pagamenti attesi da approvare/modificare
- **Budget semplice**: obiettivo risparmio + alert (opzionale)

**Cosa può fare**
- Aggiungere **Spesa / Entrata / Trasferimento** in 2–3 tap
- Scegliere:
  - conto (da cui paga / su cui entra)
  - “categoria” (che internamente è un conto Expense/Income)
- Gestire conti **principali** (rinomina, archivia) senza vedere “Dare/Avere”
- Approvare ricorrenti variabili (Telepass/bollette) dall’Inbox
- Importare dati (con preview) e vedere subito dashboard

**Cosa NON vede (o vede solo in modo “tradotto”)**
- Dare/Avere
- Splits multipli
- Albero completo di conti contabili (Assets/Liabilities/Equity ecc.) *può essere mostrato in forma semplificata*

---

### 3.2 Advanced (utente avanzato) – “contabilità vera”
**Cosa vede**
- **Albero completo conti** (Assets/Liabilities/Equity/Income/Expenses) con sottoconti
- **Ledger**: movimenti per conto (libro mastro)
- **Transazioni con splits** (righe multiple) e Dare/Avere
- Report contabili:
  - Balance Sheet (Stato patrimoniale)
  - Profit & Loss (Conto economico)
- Multi-valuta completa (cambi, FX gain/loss)

**Cosa può fare**
- Creare/modificare:
  - conti e sottoconti
  - tipi di conti (asset/liability/equity/income/expense)
- Registrare transazioni complesse:
  - splits multipli
  - pagamenti parziali
  - ripartizione spesa su più categorie
- Riconciliazione
- Gestione FX e trasferimenti cross-currency

---

### 3.3 Punto di incontro (importante)
Classic e Advanced lavorano sugli stessi dati.
- Se una transazione è creata in Classic, in Advanced si vede come transazione contabile completa.
- Se una transazione è creata in Advanced (con splits multipli), in Classic appare in modo “riassunto” (es. importo totale + note).

---

## 4) Struttura conti e logica “positivi/negativi” (Assets vs Liabilities)

### 4.1 Convenzione di segno (mentale, non da far soffrire l’utente)
- **Assets**: saldo “positivo” = soldi/valore che possiedi (banca, cash)
- **Liabilities**: saldo “negativo” = debiti/obblighi (carta di credito a debito, prestiti)
- **Equity**: serve a inizializzare e rappresenta patrimonio netto

Nel calcolo:
- Assets aumentano con **debit**, diminuiscono con **credit**
- Liabilities aumentano con **credit**, diminuiscono con **debit**
- Income aumenta con **credit**
- Expenses aumentano con **debit**

### 4.2 Visualizzazione conti “alla New Cash / GnuCash”
**Classic – vista conti semplificata**
- Schermata “Conti” con sezioni:
  - I miei soldi (Assets principali): Banca, Carta, Contanti
  - I miei debiti (Liabilities, se attive): Carta a saldo, Prestiti
- Clic su un conto → lista movimenti di quel conto

**Advanced – vista conti completa**
- Schermata “Accounts Tree”:
  - Assets (con sottoconti)
  - Liabilities (con sottoconti)
  - Equity
  - Income
  - Expenses
- Clic su conto → ledger + filtri + riconciliazione

### 4.3 Decisione di prodotto: “Carta” come Asset o Liability?
Due modelli:

1) **Carta come Liability (più corretto)**  
   - La carta rappresenta un debito verso l’issuer
   - Pagamento carta = Bank (asset) → Credit Card (liability)

2) **Carta come Asset (più semplice)**  
   - La carta è vista come “contenitore” da cui escono soldi

**Proposta pragmatica**
- scegliere un modello unico (consigliato: liability)  
- in Classic mostrarlo in modo semplice (“Carta” con saldo), senza parlare di liability

---

## 5) Inserimento transazioni

### 5.1 Classic: 3 tipi + auto-scritture
- **Spesa**: Conto (asset) + Categoria (expense) + importo
- **Entrata**: Conto (asset) + Categoria entrata (income) + importo
- **Trasferimento**: conto A → conto B (se valute diverse, serve cambio)

### 5.2 Advanced: editor splits
- transazioni con più righe
- vincolo: debit = credit (in base currency)

---

## 6) Ricorrenti variabili: “automatico ma supervisionato”
- Scheduled rules → generano istanze attese (Inbox)
- Strategie importo: Fixed, LastMonth, AverageN, Estimate+Reconcile, Schedule Table

---

## 7) Dashboard e KPI

### 7.1 KPI base (Classic)
- Income, Expenses, Net Savings, Savings Rate
- Burn Rate + Forecast fine mese (MVP)
- Top categorie + trend

### 7.2 KPI avanzati (Advanced)
- Balance Sheet completo
- P&L completo
- FX gain/loss
- Utilities trend

---

## 8) Multi‑valuta (2 livelli)
Classic: valuta per conto + base currency + cambio manuale o salvato.  
Advanced: tassi storici + FX gain/loss + reporting totale coerente.

---

## 9) Offline-first + Sync
DB locale + outbox eventi + sync automatico quando online, idempotenza e conflitti.

---

## 10) Login e monetizzazione
- Login MVP: email + password
- Free con ads → Pro 2€/mese no ads + extra

---

# 11) Bozza DB (PK/FK, multi‑valuta, ricorrenze, payee, sync)

```sql
CREATE TABLE users (
  id              UUID PRIMARY KEY,
  email           TEXT UNIQUE NOT NULL,
  password_hash   TEXT NOT NULL,
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
  updated_at        TIMESTAMPTZ NOT NULL DEFAULT now()
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

-- OFFLINE SYNC (server support)
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
