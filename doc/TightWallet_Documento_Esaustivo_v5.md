# TightWallet – Documento ESAUSTIVO (Classic & Advanced)  
**Visione • UX • Funzionalità • Import/Export • Multi‑valuta • Offline‑First • KPI • Budget • Roadmap • Database**

> Versione: **v5 (esaustiva)**  
> Scopo: documento **comprensibile** sia da una persona non tecnica sia da un informatico, senza perdere dettagli.

---

## 1) Perché la versione precedente è “più corta” (cosa era cambiato)

Hai notato bene: la v4 era più corta perché avevo **compattato** (riassunto) diverse sezioni per mettere in evidenza subito **Classic vs Advanced**.  
Non ho “cambiato idea” sul prodotto: ho solo **ridotto** testo ripetuto e alcuni dettagli erano stati spostati o accorciati.

In particolare, nella v4 erano stati **ridotti / sintetizzati** rispetto alla v3:
- la parte **posizionamento** (perché non open-banking) e alcuni esempi di marketing/approccio
- dettagli su **trigger pagamenti Apple Pay / Google Pay** (cosa realisticamente fattibile e cosa no)
- dettagli su **Liquid Glass** e scelte UI (rimasti, ma meno estesi)
- roadmap con “must / nice-to-have” era più corta
- alcune note su **export** (rimasto “previsto”, ma meno dettagliato)
- parte **AI/chat con i dati** era stata citata più brevemente
- DB: era rimasto coerente ma alcune tabelle “nice to have” non erano ripetute nella stessa estensione

Qui sotto trovi una versione **integrata**: include tutto ciò che c’era prima + chiarimento Classic/Advanced + aggiunte (multi‑valuta più rigorosa, budget automatico, offline sync) **senza diminuire**.

---

## 2) Obiettivo (in una frase)

**TightWallet** è un’app di gestione soldi moderna che usa la **partita doppia** (come GnuCash) ma con UX semplice:  
- **Classic** per l’uso quotidiano (veloce, senza contabilità “in faccia”)  
- **Advanced** per chi vuole il controllo contabile completo (albero conti, ledger, report)  
Con **import** immediato da GnuCash e Money Manager, **dashboard componibile**, **KPI**, **budget**, **multi‑valuta**, e **offline‑first** con sincronizzazione trasparente.

---

## 3) Principi di prodotto (non negoziabili)

1. **Partita doppia vera**: ogni transazione è bilanciata (Dare = Avere).  
2. **Conti strutturati**: Assets / Liabilities / Equity / Income / Expenses (come GnuCash).  
3. **Classic e Advanced condividono gli stessi dati**: cambia la vista, non la base dati.  
4. **Import subito, export dopo**: prima portiamo dentro la storia; l’export viene come step successivo ma è progettato fin dall’inizio.  
5. **Web + Mobile**:  
   - Desktop: web app (browser)  
   - Mobile: app nativa/ibrida che chiama API  
6. **Offline‑first**: l’app mobile deve funzionare anche senza Internet, con sync automatico quando torna online.  
7. **Tema chiaro/scuro**: segue il device, con override manuale.  
8. **Monetizzazione semplice**: Free con ads, Pro 2€/mese senza ads + funzionalità avanzate.

---

## 4) Concetti chiave spiegati “da persona normale”

### 4.1 Cos’è un “conto” (Account)
Un conto è un contenitore di soldi o valore. Esempi:
- “Banca”
- “Contanti”
- “Carta”
- “Prestito Findomestic”
- “Spese: Gas”
- “Entrate: Stipendio”

Sì: anche “Spese Gas” e “Entrate Stipendio” sono conti (in contabilità si fa così).

### 4.2 Cos’è una transazione a partita doppia
Ogni movimento ha **due lati**:
- da dove escono i soldi
- dove vanno (categoria/spesa, entrata, altro conto)

Esempio (spesa di 3€ pagata in contanti):
- Tolgo 3€ da Contanti  
- Li metto su Spese: Caffè

È un modo rigoroso per evitare “buchi” nei conteggi.

---

## 5) Setup iniziale (fondamentale per l’utente Classic)

Qui sta il punto critico che hai evidenziato: anche l’utente semplice deve avere **conti configurati**.

### 5.1 Primo avvio: Wizard (60–120 secondi)
Obiettivo: partire subito senza “creare conti a mano”.

Step consigliati:
1) **Valuta base del libro** (es. EUR)  
2) **Conti che possiedi** (toggle + nome):
   - Banca (EUR)
   - Contanti (EUR)
   - Carta (EUR)
   - (Opzionale) Cash USD / Bank USD se serve
3) (Opzionale) Import storico: GnuCash / Money Manager / CSV  
4) Scelta modalità iniziale: Classic (default) o Advanced

### 5.2 Template “Chart of Accounts” creato automaticamente
TightWallet crea un albero standard (sempre modificabile), con sottoconti essenziali:

**Assets (Attività)**
- Cash on Hand (Contanti)
- Bank Accounts (Banca)
- (opzionale) Investments
- (opzionale) Fixed Assets

**Liabilities (Passività)**
- Credit Card
- Loans Payable
- Other Liabilities

**Equity**
- Opening Balances (IMPORTANTISSIMO)
- Retained Earnings

**Income**
- Salary
- Other Income

**Expenses**
- Utilities (Gas/Luce/Acqua)
- Groceries
- Transport (Telepass/Carburante)
- Subscriptions
- Health
- Leisure
- Taxes
- Other Expenses

> L’utente Classic non deve “vedere” per forza tutto questo albero, ma deve esistere.

### 5.3 Saldi iniziali: Opening Balances (Equity)
Quando l’utente imposta un saldo iniziale:
- Assets:Banca +1.000  
- Equity:Opening Balances +1.000 (lato opposto)

Questo serve a non “barare” coi saldi: è contabilità corretta e sblocca report affidabili.

---

## 6) Classic vs Advanced (finalmente chiarissimo)

### 6.1 Classic (utente semplice) – cosa vede
Schermate principali:
1) **Dashboard/Home** (colpo d’occhio)  
   - saldo totale
   - spese mese
   - entrate mese
   - risparmio mese
   - trend base
2) **Conti (semplificati)**  
   - elenco conti principali: Banca, Carta, Contanti  
   - (opzionale) “Debiti” se attivo: Prestiti, Carta come debito  
   - click su un conto → movimenti di quel conto
3) **Movimenti** (vista tabellare)  
   - lista transazioni con filtri e ricerca
4) **Ricorrenti/Inbox**  
   - pagamenti attesi da approvare (anche variabili)
5) **Budget semplice**  
   - obiettivo di risparmio + avvisi (opzionale)

### 6.2 Classic – cosa può fare
- Aggiungere **Spesa / Entrata / Trasferimento** in pochi tap  
- Scegliere sempre almeno:
  - **conto** (da cui paga / su cui entra)
  - **categoria** (in UI, ma internamente è un conto Expense/Income)
- Modificare e cancellare transazioni
- Approvare ricorrenti variabili (Telepass/bollette)
- Importare dati con preview
- Gestire conti “principali” (rinomina, archivia, saldo iniziale)

### 6.3 Classic – cosa NON vede (o vede tradotto)
- Dare/Avere
- splits multipli (se esistono, vede un “riassunto”)
- tree completo Assets/Liabilities/Equity (può essere nascosto dietro “Avanzate”)

---

### 6.4 Advanced (utente avanzato) – cosa vede
1) **Accounts Tree completo** (stile GnuCash/New Cash)  
   - Assets / Liabilities / Equity / Income / Expenses  
   - con sottoconti
2) **Ledger per conto**  
   - tutti i movimenti interni del conto selezionato
3) **Editor transazione con splits**  
   - più righe, ripartizioni, contabilità piena
4) **Report contabili**
   - Balance Sheet (Stato patrimoniale)
   - Profit & Loss (Conto economico)
5) **Multi‑valuta completa**
   - exchange rates, cross-currency, FX gain/loss

### 6.5 Advanced – cosa può fare
- creare/modificare conti e sottoconti
- scegliere il tipo conto (asset/liability/equity/income/expense)
- transazioni complesse (splits multipli)
- riconciliazione
- gestione cambi e differenze cambio

---

### 6.6 Punto cruciale: stessi dati, due viste
- Una transazione creata in Classic è una transazione “vera” (in Advanced la vedi completa).
- Una transazione creata in Advanced con 5 splits, in Classic appare come:
  - importo totale + descrizione + “dettagli” apribili.

---

## 7) Vista conti “alla New Cash” (richiesta specifica)

### 7.1 Classic: vista conti per categorie (semplice)
- Sezione “I miei soldi” (Assets principali): Banca, Contanti, Carta
- Sezione “I miei debiti” (Liabilities): Prestiti, Carta (se modellata come debito)
- Sezione “Altro” (se serve): Investimenti ecc.

Click su un conto → movimenti del conto (ledger semplificato)

### 7.2 Advanced: albero completo
- Assets → sottoconti
- Liabilities → sottoconti
- Equity
- Income
- Expenses

Click su un conto → ledger completo con filtri.

---

## 8) Logica “Assets positivi / Liabilities negativi” (spiegazione chiara)

### 8.1 Cosa significa per l’utente
- Assets: ciò che possiedi (saldo positivo)
- Liabilities: ciò che devi (saldo “negativo” nel senso che riduce il patrimonio)

### 8.2 Perché è importante
Perché i calcoli di patrimonio e report hanno senso solo così:
- Patrimonio Netto = Assets − Liabilities

### 8.3 Dettaglio contabile (per informatici / advanced)
- Assets aumentano con **debit**
- Liabilities aumentano con **credit**
- Income aumenta con **credit**
- Expenses aumentano con **debit**
- Equity spesso aumenta con **credit** (dipende dal conto)

Classic non deve conoscere i termini, ma il motore sì.

---

## 9) Decisione di prodotto: Carta come Asset o Liability (IMPORTANTE)

Ci sono due modelli:

### Modello A — Carta come Liability (più corretto)
- La carta rappresenta un debito verso l’issuer fino a quando paghi il saldo.
- Quando spendi con carta:
  - aumenti la liability “Credit Card”
  - aumenti la spesa “Expense:…”
- Quando paghi il saldo carta:
  - diminuisci liability carta
  - diminuisci asset banca

Pro: contabilità corretta  
Contro: per Classic va “nascosta bene” per non confondere.

### Modello B — Carta come Asset (più semplice)
- La carta è vista come “portafoglio digitale”.
Pro: facilissimo per Classic  
Contro: non è contabilità rigorosa, soprattutto con saldo carta e rate.

**Proposta consigliata (coerente con GnuCash):** usare il modello A (liability) ma presentarlo in Classic in modo semplice (“Carta” con saldo e movimenti) senza parlare di debito.

---

## 10) Inserimento movimenti (Classic) – esempi concreti

### 10.1 Spesa semplice
Spendo 10€ di supermercato con carta:
- UI Classic: Importo 10, conto “Carta”, categoria “Spese → Alimentari”
- Motore:
  - Debit: Expenses:Groceries 10
  - Credit: (Asset o Liability carta, a seconda del modello) 10

### 10.2 Trasferimento
Sposto 50€ da banca a contanti:
- Debit: Assets:Cash 50
- Credit: Assets:Bank 50

### 10.3 Entrata
Stipendio 2.000€ su banca:
- Debit: Assets:Bank 2000
- Credit: Income:Salary 2000

---

## 11) Ricorrenti: la feature che elimina la “lista mensile”

### 11.1 Problema reale
“Ogni mese devo ricordarmi cosa pago” → noioso, soggetto a errori.

### 11.2 Soluzione: Scheduled Rules + Inbox (automatico supervisionato)
- Definisci una regola: Telepass, Gas, Rata Findomestic, Abbonamento.
- Ogni periodo genera una “istanza attesa” nell’Inbox.
- Tu approvi/modifichi/posticipi.

### 11.3 Pagamenti variabili: strategie importo
- Fixed
- LastMonth
- AverageN
- Estimate + Reconcile (stimo e poi aggiusto)
- Schedule Table (piano rate importabile)

### 11.4 Matching con import (futuro)
Quando importi estratto conto:
- match su payee + data window + tolleranza importo
- auto-collegamento all’istanza attesa

---

## 12) Normalizzazione descrizioni (Gas/gas/metano) e trend utilities

### 12.1 Problema
Se scrivi ogni volta un nome diverso, i grafici “Gas” non funzionano.

### 12.2 Soluzione
- Payee canonico “GAS” con alias
- Autocomplete mentre digiti
- “Preferiti/like” per descrizioni ricorrenti

### 12.3 Extra utile: metriche bolletta (opzionale)
Salvare:
- consumo (m³ o kWh)
- costo unitario (€/m³)
Permette grafici più utili del solo “quanto ho pagato”.

---

## 13) Dashboard (a colpo d’occhio) e vista tabellare

### 13.1 Dashboard componibile (card/widget)
- layout personalizzabile (drag&drop)
- ogni widget configurabile (periodo, conti, categorie, tag)

Widget tipici:
- saldo totale (in base currency)
- saldi per conti principali
- spese/entrate/risparmio del mese
- top categorie
- trend 6/12 mesi
- ricorrenti: “fisso del mese”
- inbox pagamenti attesi
- utilities: andamento gas/luce/acqua

### 13.2 Vista elenco (tabellare)
- filtri: periodo, conto, categoria, tag, payee, importo
- ricerca testuale
- editing rapido

---

## 14) KPI – elenco chiaro con significato

KPI “base” (Classic):
- Income (entrate)
- Expenses (spese)
- Net Savings (risparmio netto)
- Savings Rate (% risparmio)
- Burn Rate (spesa media giornaliera)
- Forecast fine mese
- Top categorie + trend

KPI “avanzati” (Advanced):
- Balance Sheet completo
- Profit&Loss completo
- multi‑valuta con FX gain/loss
- utilities: costo/consumo/costo unitario

---

## 15) Budget (anche automatico)

### 15.1 Budget classico per categoria
- “Ristoranti 200€ al mese”
- semaforo + notifiche

### 15.2 Budget “obiettivo risparmio”
- “Questo mese voglio risparmiare 50€”
- alert quando il trend non lo permette

---

## 16) Multi‑valuta “alla GnuCash” (spiegata semplice)

Regola:
- la valuta è del **conto**
- il libro ha una **base currency** per report totali
- per trasferimenti cross‑currency serve un tasso

Decisione MVP:
- per trasferimenti tra valute chiedi il cambio (manuale o da lista)
- salva il tasso usato (per report corretti)

---

## 17) Apple Pay / Google Pay: cosa è realistico fare
Obiettivo: aprire TightWallet in “quick add” dopo un pagamento.

MVP realistico:
- deep link / intent / scorciatoia
- (spesso) non puoi leggere automaticamente importo/merchant per limiti OS, ma puoi ridurre attrito.

Step successivi (se fattibile):
- integrazioni con notifiche (attenzione privacy/permessi)
- PayPal: login + import movimenti (futuro)

---

## 18) Web + Mobile + Offline‑First (seamless)

### 18.1 Architettura
- Backend API con dominio contabile
- Web app per desktop
- Mobile app con DB locale + sync

### 18.2 Offline sync (concetto)
- outbox eventi locali
- invio con idempotenza
- delta sync dal server
- conflitti MVP: last-write-wins + log

---

## 19) Login e lingua
- Login MVP: email + password
- Lingue: IT + EN (default = lingua sistema, override manuale)
- Preferenze salvate: modalità Classic/Advanced, tema, lingua, valuta base

---

## 20) Monetizzazione
- Free con pubblicità e limiti
- Pro 2€/mese: no ads + storico illimitato + analytics avanzate + export completo + backup

---

## 21) Roadmap (step incrementali)

**Fase 1 – Core**
- chart of accounts + opening balances
- Classic/Advanced + preferenze
- import GnuCash/Money Manager
- dashboard base + vista tabellare
- KPI base

**Fase 2 – Differenziante**
- scheduled + inbox + strategie variabili
- budget obiettivo risparmio
- payee normalization + utilities trend
- offline-first mobile + sync

**Fase 3 – Espansioni**
- export completo (Excel/PDF/JSON)
- AI/chat con i dati (privacy e governance)
- open banking opzionale

---

# 22) Database – schema completo (PK/FK) in SQL

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

-- OFFLINE SYNC SUPPORT
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

## 23) Nota operativa (da dev a dev)
La cosa che vi farà “spaccare” non sarà la torta 3D:  
sarà **(1)** la struttura contabile coerente, **(2)** ricorrenti variabili con inbox, **(3)** multi‑valuta fatta bene, **(4)** offline seamless.  
Se queste quattro cose sono solide, tutto il resto diventa incrementale.
