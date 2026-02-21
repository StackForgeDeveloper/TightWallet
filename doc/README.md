# TightWallet

TightWallet è un’app di gestione finanziaria personale basata su **partita doppia reale** (double-entry bookkeeping), ispirata a GnuCash ma progettata con un’esperienza moderna:

- **Classic**: semplice, rapida, quotidiana (per “normali esseri umani”)
- **Advanced**: completa, contabile, rigorosa (per chi vuole controllo totale)

Il progetto è pensato per essere **web-first** (desktop via browser) e **mobile-first** (iOS/Android), con una caratteristica fondamentale: **offline-first** sul mobile con sincronizzazione automatica quando torna online.

---

## Indice

- [Perché TightWallet](#perché-tightwallet)
- [Concetti chiave](#concetti-chiave)
- [Classic vs Advanced](#classic-vs-advanced)
- [Setup iniziale](#setup-iniziale)
- [Ricorrenti “automatici ma supervisionati”](#ricorrenti-automatici-ma-supervisionati)
- [Multi-valuta](#multi-valuta)
- [Dashboard, KPI e Budget](#dashboard-kpi-e-budget)
- [Import / Export](#import--export)
- [Architettura (Web + API + Mobile)](#architettura-web--api--mobile)
- [Offline-first & Sync](#offline-first--sync)
- [Modello dati (high level)](#modello-dati-high-level)
- [Database (schema: overview)](#database-schema-overview)
- [Sicurezza & privacy](#sicurezza--privacy)
- [Monetizzazione](#monetizzazione)
- [Roadmap](#roadmap)
- [Struttura repository (proposta)](#struttura-repository-proposta)
- [Contribuire](#contribuire)
- [Glossario](#glossario)

---

## Perché TightWallet
### Ispirazioni: GnuCash **e** Money Manager (App Store)

TightWallet nasce con due “bussole”:

- **GnuCash** per il *motore contabile*: partita doppia rigorosa, albero conti (Assets/Liabilities/Equity/Income/Expenses), multi‑valuta corretta e report contabili affidabili.
- **Money Manager** (l’app “money mgr” disponibile su App Store) per l’*esperienza d’uso quotidiana*: inserimento rapido, schermata conti chiara, ricorrenti pratici, budget e grafici immediati.

In pratica: **sotto** la solidità di GnuCash, **sopra** la comodità “da app” di Money Manager.

**Comportamenti “stile Money Manager” che vogliamo replicare**
- **Schermata Conti immediata**: lista conti con saldo, raggruppati in sezioni (es. Assets / Debiti) e accesso rapido ai movimenti del conto.
- **Quick Add super veloce**: aggiungi spesa/entrata/trasferimento con pochi tap.
- **Ricorrenti pratici**: non solo “automatici”, ma con logica che aiuta davvero nella vita reale (Inbox + approvazione, soprattutto per importi variabili).
- **Budget “semplice”**: anche se l’utente non imposta budget per categoria, l’app deve riuscire a dare feedback utili (es. obiettivo risparmio mensile).
- **Grafici chiari**: trend mensili, top categorie, confronto entrate/spese, vista tabellare filtrabile.
- **Pulizia dati**: aiutare l’utente a usare descrizioni coerenti (payee canonico + alias) per evitare caos tipo “Gas/gas/metano”.

> Nota: oltre ai comportamenti UI, TightWallet prevede **import** dai dati di Money Manager (e GnuCash) per partire subito con uno storico reale.


Molte app moderne “si collegano alla banca” e fanno tutto automaticamente. È comodo, ma spesso:
- ti ritrovi dati sporchi / categorizzazioni strane
- non hai un modello contabile rigoroso
- ti senti “spettatore” invece che protagonista
- non puoi lavorare bene offline
- se cambi app, migrare i dati è un incubo

TightWallet punta a una filosofia diversa:

> **Tu controlli i dati. L’app ti aiuta a registrarli bene e a capirli meglio.**

Questa scelta abilita cose “da app seria”:
- report coerenti (stato patrimoniale, conto economico)
- gestione debiti (liabilities) fatta bene
- multi-valuta corretta
- ricorrenti variabili (Telepass, bollette, rate) gestite in modo intelligente

---

## Concetti chiave

### Conti (Accounts) e albero contabile
TightWallet usa la struttura contabile standard (come GnuCash):

- **Assets** (Attività): ciò che possiedi (banca, contanti, investimenti…)
- **Liabilities** (Passività): ciò che devi (carta di credito, prestiti…)
- **Equity**: patrimonio netto e saldi di apertura
- **Income**: entrate
- **Expenses**: spese

L’utente Classic non deve “studiarsela”, ma questa struttura **deve esistere** perché:
- anche per una spesa semplice, devi scegliere *da quale conto* escono i soldi
- le liabilities hanno una logica diversa (saldo “negativo” / debito)
- i report e i KPI hanno senso solo se i dati sono coerenti

### Partita doppia (Double-entry)
Ogni transazione ha almeno due righe (split):
- un lato “da dove escono”
- un lato “dove vanno”

Esempio: caffè da 1€ pagato in contanti:
- `Expenses:Coffee` +1€
- `Assets:Cash` -1€

Il sistema garantisce che **la somma dei Dare = somma degli Avere** (bilanciamento).

---

## Classic vs Advanced

### Classic (utente semplice)
**Cosa vede**
- Dashboard “a colpo d’occhio” (saldo, spese/entrate mese, risparmio, trend)
- Lista conti principali (Banca/Carta/Contanti) con saldo
- Lista movimenti (tabellare) con filtri e ricerca
- Inbox Ricorrenti (pagamenti attesi da approvare)
- Budget semplice (opzionale): obiettivo risparmio + avvisi

**Cosa fa**
- Aggiunge **Spesa / Entrata / Trasferimento** in pochi tap
- Seleziona sempre:
  - **conto** (da cui paga o su cui entra)
  - **categoria** (in UI), che internamente è un conto `Expense` o `Income`
- Approva/modifica ricorrenti anche variabili (Telepass/bollette)
- Importa storico e ottiene subito dashboard e trend

**Cosa non deve vedere**
- Dare/Avere
- splits multipli (se esistono, li vede “riassunti”)

### Advanced (utente avanzato)
**Cosa vede**
- Albero conti completo (Assets/Liabilities/Equity/Income/Expenses)
- Ledger per conto (movimenti del conto)
- Editor transazione completo con splits multipli
- Report contabili: Balance Sheet e Profit & Loss
- Multi-valuta completa (cambi, FX gain/loss)

**Cosa fa**
- Gestisce conti e sottoconti
- Registra transazioni complesse (ripartizioni, pagamenti parziali)
- Riconcilia, analizza, fa reporting “da contabilità vera”

> Nota: Classic e Advanced condividono lo stesso database. È solo una differenza di UI/permessi.

---

## Setup iniziale

Per evitare che l’utente “debba crearsi tutto a mano”, TightWallet prevede un **wizard di avvio**:

1) Valuta base del libro (es. EUR)  
2) Conti posseduti (toggle + nome):
   - Banca (EUR)
   - Contanti (EUR)
   - Carta (EUR)
   - (opzionale) Cash USD / Bank USD  
3) Import storico (opzionale): GnuCash / Money Manager / CSV  
4) Modalità iniziale: Classic (default) o Advanced

### Opening Balances (saldi di apertura)
Quando imposti un saldo iniziale su un conto, TightWallet crea automaticamente una transazione “Apertura” tramite `Equity:Opening Balances`.
Questo rende i saldi e i report coerenti fin dal primo giorno.

---

## Ricorrenti “automatici ma supervisionati”

Problema reale: “ogni mese devo ricordarmi cosa pagare e inserire una lista a mano”.

Soluzione TightWallet:
- definisci una **Scheduled Rule** (Telepass, Gas, rata…)
- ogni periodo viene creata una **Expected Instance** nell’Inbox
- tu **approvi / modifichi / posticipi**
- solo dopo approvazione diventa una transazione reale

### Pagamenti variabili: strategie importo
Ogni regola può usare una strategia:
- `Fixed` (importo fisso)
- `LastMonth` (propone ultimo importo)
- `AverageN` (media ultimi N)
- `Estimate + Reconcile` (stima ora, aggiusta quando arriva il reale)
- `Schedule Table` (piano rate importabile)

Questo è uno dei principali elementi differenzianti del progetto.

---

## Multi-valuta

Modello corretto “alla GnuCash”:
- **la valuta è proprietà del conto** (es. `Cash EUR`, `Cash USD`)
- le categorie (`Expenses:Coffee`) **non hanno** valuta
- il libro ha una **base currency** per reporting aggregato (es. EUR)

### Trasferimenti tra valute
Un trasferimento EUR→USD richiede un tasso di cambio (manuale o da lista salvata).
Per report coerenti in base currency, si salva il tasso e (se necessario) una differenza di cambio su un conto FX.

---

## Dashboard, KPI e Budget

### Dashboard componibile
Dashboard basata su widget/card:
- saldo totale (in base currency)
- saldi conti principali
- spese/entrate/risparmio mese
- top categorie
- trend 6/12 mesi
- ricorrenti (fisso del mese, inbox pagamenti)
- utilities (gas/luce/acqua): costo e (opzionale) consumo

È sempre disponibile anche la **vista tabellare** dei movimenti con filtri avanzati.

### KPI principali (MVP)
- Income (Entrate)
- Expenses (Spese)
- Net Savings (Entrate − Spese)
- Savings Rate (% risparmio)
- Burn Rate (spesa media giornaliera)
- Forecast fine mese (MVP)

### Budget
- Budget per categoria (se l’utente vuole)
- Budget “automatico” come obiettivo risparmio: “questo mese voglio risparmiare 50€”  
  → l’app avvisa se l’andamento non lo rende più realistico

---

## Import / Export

### Import (priorità alta)
Supporto iniziale:
- GnuCash
- Money Manager
- CSV / Excel (adattatori)

Pipeline robusta:
1) upload → 2) parsing → 3) mapping → 4) normalizzazione → 5) preview → 6) commit atomico → 7) log

### Normalizzazione descrizioni (es. “Gas/gas/metano”)
Per evitare dati sporchi:
- `Payee` canonico + alias
- autocomplete mentre scrivi
- preferiti/like

### Export (step successivo, ma progettato)
- CSV / Excel per transazioni e report
- PDF report mensile/annuale
- JSON backup completo

---

## Architettura (Web + API + Mobile)

### Backend (API)
- API REST (dominio contabile centralizzato)
- validazione partita doppia
- import pipeline
- ricorrenti + inbox
- KPI aggregati
- supporto multi-device

### Web App
- desktop via browser
- analisi comoda su schermo grande
- import/export più pratici
- dashboard completa

### Mobile App
- UX veloce + widget + biometria (sblocco)
- offline-first con DB locale
- sync automatico (seamless)

---

## Offline-first & Sync

Requisito: l’app mobile deve funzionare anche senza internet.

Approccio:
- DB locale (es. SQLite)
- ogni azione genera un evento in **outbox**
- quando online:
  - invio eventi (idempotenti)
  - delta sync dal server
  - gestione conflitti (MVP: last-write-wins + log conflitti)

---

## Modello dati (high level)

Entità principali:
- `User`
- `BookSettings` (base currency)
- `LedgerAccount` (albero conti)
- `Transaction` (testa)
- `Split` (righe, partita doppia)
- `Payee` + `PayeeAlias`
- `ExchangeRate`
- `ScheduledRule` + `ExpectedInstance`
- `DashboardWidget`
- `Budget` + `SavingsTarget`
- `ImportJob` + `ImportRow`
- `Device` + `ChangeEvent` + `SyncCursor` (sync)

---

## Database (schema: overview)

Il database è progettato per:
- garantire coerenza contabile
- supportare multi-valuta e reporting in base currency
- supportare ricorrenti variabili con inbox
- supportare offline-first multi-device con sync

> Il dettaglio completo dello schema SQL vive nei documenti di design; nel repository troverai gli script/migrazioni in `db/` (vedi struttura proposta).

---

## Sicurezza & privacy

- Password hash (es. Argon2/bcrypt) lato server
- HTTPS obbligatorio
- Token di sessione / JWT (a scelta implementativa)
- Dati finanziari = privacy by design:
  - minimizzare log sensibili
  - cifratura backup locale
  - permessi granulari per export/AI future
- Biometria su mobile: sblocco client-side (non sostituisce login)

---

## Monetizzazione

- **Free**: pubblicità + eventuali limiti (storico, export, widget avanzati)
- **Pro (2€/mese)**: no ads + storico illimitato + export completo + analytics avanzate + backup

---

## Roadmap

### Fase 1 (Must-have)
- Core contabile (Accounts/Transactions/Splits)
- Classic/Advanced + preferenze
- Import GnuCash/Money Manager
- Dashboard base + vista tabellare
- KPI base

### Fase 2 (Differenziante)
- Scheduled rules + Inbox con pagamenti variabili
- Budget obiettivo risparmio
- Payee normalization + utilities trend
- Offline-first + sync completo

### Fase 3 (Nice-to-have)
- Export completo (CSV/Excel/PDF/JSON)
- AI/Chat con i tuoi dati (con governance privacy)
- Open banking opzionale (non requisito)

---

## Struttura repository (proposta)

Questa è una struttura consigliata (adattabile):

```
TightWallet/
  README.md
  docs/
    product/
      spec-classic-vs-advanced.md
      kpi.md
      recurring.md
      multicurrency.md
      offline-sync.md
    architecture/
      api-contracts.md
      data-model.md
      database-schema.sql
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

## Contribuire

- Apri issue con:
  - contesto (Classic/Advanced)
  - scenario riproducibile (passi)
  - risultato atteso vs ottenuto
- PR piccole e focalizzate (una feature/bug per PR)
- Prima di aggiungere UI, assicurarsi che il dominio contabile resti coerente:
  - transazioni sempre bilanciate
  - valute per conto rispettate
  - ricorrenti e import non creano duplicati

---

## Glossario

- **Account/Conto**: contenitore contabile (asset, liability, income, expense…)
- **Split**: riga di una transazione (una voce Dare o Avere)
- **Payee**: merchant/fornitore normalizzato (es. “GAS”)
- **Base currency**: valuta del libro per reporting aggregato
- **Scheduled rule**: regola programmata (ricorrenza)
- **Expected instance**: “pagamento atteso” nell’inbox
- **Offline-first**: funziona senza rete; sync quando torna online
- **Ledger**: lista movimenti di un conto

---

Se stai leggendo questo README e ti viene voglia di fare ordine nei soldi (o di costruire un piccolo GnuCash 2.0 senza traumi)… sei nel posto giusto.
