# Marubozu

Marubozu è una piattaforma web per il journaling e la gestione operativa del trading, composta da:

- **project-marubozu-client**: frontend web in Next.js/TypeScript
- **project-marubozu-server**: backend API in Java/Quarkus con autenticazione, subscription, storage file e logica di dominio

Dalla struttura del codice, i due repository implementano insieme un trading hub con gestione account, diario di trading, wallet, documenti, note, analytics e piani/subscription.

---

## Visione d'insieme del prodotto

Il client si presenta esplicitamente come "The next generation trading hub!" e reindirizza la root verso `/home`, con un layout applicativo completo fatto di sidebar, transizioni pagina, banner di subscription e pulsante rapido per aggiungere trade.

Il backend espone API REST Quarkus con persistenza ORM, migrazioni Flyway, OpenAPI, JWT, mailer, scheduler, cache, PostgreSQL e integrazione S3/R2.

---

## 1. Cosa fa project-marubozu-client

### Ruolo del repository

È il frontend dell'applicazione, realizzato con **Next.js 15 + React 19 + TypeScript**. Gestisce:

- UI applicativa
- autenticazione lato client
- routing pubblico/protetto
- gestione stato di subscription
- pagine operative per trading journal, wallet, documenti, note, dashboard, ecc.
- interazione con le API backend tramite layer `services/*`

### Stack e librerie principali

Dal `package.json` emerge che il frontend usa:

- **Next.js**
- **next-auth** per autenticazione/sessione
- **React Query** per fetching/caching dati
- **TanStack Table / Virtual** per tabelle grandi e virtualizzazione
- **TipTap** per editor rich text
- **Radix UI / shadcn/ui** per componenti UI
- **Recharts** per grafici/analytics
- **axios** per chiamate HTTP

### Funzionalità core del client

#### 1. Autenticazione e accesso utenti

Il client include pagine e contesto per:

- login
- registrazione
- forgot password
- reset password
- verifica email
- login con OAuth provider (Google / Microsoft / Apple)

Inoltre:

- usa `SessionProvider` di next-auth
- mostra flusso specifico se l'utente non ha verificato l'email
- consente il reinvio della mail di verifica
- distingue pagine pubbliche e protette

#### 2. Layout applicativo e navigazione dashboard

Il layout client include:

- sidebar
- transizioni di pagina
- floating button per aggiungere trade
- notifiche di subscription: `ExpiryNotification`, `ExpiredBanner`

Questo suggerisce una UX da dashboard SaaS strutturata.

#### 3. Gestione tema e UX personalizzata

C'è un `ThemeContext` che supporta:

- tema light/dark
- persistenza su localStorage
- fallback sul tema di sistema

C'è anche un `SidebarContext` per stato collapsible della sidebar.

#### 4. Gestione subscription e feature gating

Il client ha una `SubscriptionContext` piuttosto centrale. Supporta:

- lettura dello stato abbonamento
- elenco feature attive
- trial automatico di 7 giorni
- stato expired
- controllo accesso alle feature

Nel codice risultano come feature core visibili anche in sola lettura quando la subscription è scaduta:
`TRADES`, `JOURNAL`, `ANALYTICS`, `WALLET`, `DOCS`, `NOTES`

Questa è un'indicazione molto importante del dominio funzionale complessivo.

#### 5. Wallet e patrimonio

Esiste una pagina protetta `wallet` con UI per:

- snapshot del wallet
- total balance
- allocated balance
- liquidity
- PnL
- gestione exchange collegati
- deposit / withdraw
- tabelle asset / exchange / transazioni

Quindi il prodotto non è solo diario trading, ma anche monitoraggio patrimoniale / portafoglio.

#### 6. Servizi frontend per i moduli di business

Dalla cartella `services` si vede chiaramente il perimetro delle feature applicative. Il client ha servizi dedicati a:

`assetService`, `dashboardService`, `docFileService`, `docFolderService`, `journalService`, `legalService`, `macroService`, `marketPairService`, `masterExchangeService`, `moodService`, `noteService`, `planService`, `subscriptionService`, `takeProfitService`, `timeframeService`, `tipService`, `tradeService`, `tradeTypeService`, `uploadService`, `userConfigService`, `userExchangeService`, `userService`, `walletBalanceService`, `walletService`, `walletTransactionService`, `backendAuthService`

Questo permette di dedurre in modo affidabile le principali capability di prodotto.

### Core features lato client, in elenco

#### Area account e accesso

- Registrazione account
- Login email/password
- Login OAuth
- Verifica email
- Reinvio email di verifica
- Recupero password
- Reset password
- Gestione sessione utente

#### Area trading journal

- Gestione trade
- Gestione journal
- Gestione mood
- Gestione take profit
- Gestione trade type
- Gestione timeframe
- Gestione market pair
- Probabile gestione di piani/regole (`planService`)
- Tip/insights (`tipService`)

#### Area analytics e dashboard

- Dashboard dati
- Grafici / analytics
- Probabile sintesi prestazioni trading
- Tabelle avanzate e visualizzazioni grandi dataset

#### Area wallet / portfolio

- Snapshot wallet
- Bilancio totale / liquidità / PnL
- Exchange collegati
- Asset, transazioni, depositi e prelievi

#### Area documentale e conoscenza personale

- Cartelle documenti
- File documentali
- Upload
- Note
- Editor rich text TipTap

#### Area configurazione utente

- Profilo utente
- Configurazioni utente
- Preferenze tema
- Preferenze valuta/default settings

#### Area commerciale / subscription

- Stato abbonamento
- Trial automatico 7 giorni
- Feature gating
- Stato expired
- Piani / permessi / accesso funzionalità

#### Area legale/pubblica

- Privacy / Terms
- Flussi pubblici separati dall'area autenticata

---

## 2. Cosa fa project-marubozu-server

### Ruolo del repository

È il backend REST dell'ecosistema Marubozu, sviluppato in **Java 21 + Quarkus**. Gestisce:

- autenticazione e autorizzazione JWT/ruoli
- persistenza dati
- subscription e trial
- profilo utente
- journal e altri moduli di dominio
- email transazionali
- upload file/avatar
- integrazioni cloud storage
- health/openapi

### Stack backend principale

Dal `pom.xml` risultano:

- **Quarkus**
- **Hibernate ORM / Panache**
- **Flyway**
- REST + Jackson
- PostgreSQL JDBC + reactive client
- JWT verify/build
- Mailer / Scheduler / Cache
- Health checks
- MapStruct / S3 SDK / BCrypt / Jsoup / Bean Validation

Quindi è un backend abbastanza completo da SaaS production-style.

### Funzionalità core del server

#### 1. Autenticazione completa

`AuthResource` espone endpoint per:

| Metodo | Endpoint |
|--------|----------|
| POST | `/api/auth/register` |
| GET | `/api/auth/verify-email` |
| POST | `/api/auth/resend-verification` |
| POST | `/api/auth/login` |
| POST | `/api/auth/oauth` |
| POST | `/api/auth/refresh` |
| POST | `/api/auth/logout` |
| POST | `/api/auth/forgot-password` |
| POST | `/api/auth/reset-password` |

#### 2. Sicurezza JWT e regole di accesso

Il backend usa `@Authenticated`, `@RolesAllowed`, SmallRye JWT e un `MustChangePasswordFilter` che blocca tutte le chiamate autenticate se l'utente ha il claim `mustChangePassword = true`.

È tipico di: utenti invitati, password temporanee, onboarding amministrato.

#### 3. Waitlist e inviti

`WaitlistResource` espone:

- registrazione pubblica in waitlist
- count iscritti waitlist
- invito da parte admin di un iscritto, con creazione utente e password temporanea

Quindi Marubozu sembra avere/aver avuto anche una fase di accesso controllato tramite waitlist.

#### 4. Profilo utente e avatar

`UserResource` gestisce:

- recupero profilo utente loggato
- aggiornamento profilo
- upload avatar su storage S3/R2 con validazione MIME type, limite dimensione file e gestione URL pubblico

#### 5. Subscription, trial, upgrade e promo access

`SubscriptionResource` mostra un modulo subscription abbastanza ricco:

- recupero permessi dell'utente
- attivazione trial di 7 giorni
- upgrade/downgrade piano
- creazione sessione checkout
- webhook provider di pagamento
- gestione promo access lato admin
- storico promo per utente

#### 6. Journal di trading

`JournalResource` conferma il modulo journaling:

- get journal per id / per utente / per trade
- create journal
- accesso protetto e gating via subscription tier `CORE`

#### 7. Gating per tier di abbonamento

Nel backend compare l'annotazione custom `@RequiresTier(SubscriptionTier.CORE)`. La logica dei piani non è solo frontend, ma è **enforced lato server**.

#### 8. Email transazionali

Il server invia email per: verifica account, forgot password, waitlist/inviti.

#### 9. Storage file cloud

L'uso di AWS SDK S3 con endpoint custom e chiavi configurabili suggerisce storage tipo Cloudflare R2 o S3-compatible object storage, utile per avatar e documenti/file upload.

#### 10. Documentazione API e salute del servizio

Il backend include OpenAPI / Swagger e Health checks.

### Core features lato server, in elenco

#### Autenticazione e sicurezza

- Registrazione utente / Login email/password / Login OAuth
- JWT access token / Refresh token rotation / Logout/revoca token
- Verifica email / Reinvio verifica con rate limiting
- Forgot password / Reset password
- Enforcement cambio password obbligatorio
- Autorizzazione per ruoli user / admin

#### Gestione utenti

- Profilo utente corrente / Aggiornamento profilo / Upload avatar
- Storage cloud per immagini/file

#### Waitlist / onboarding

- Iscrizione waitlist pubblica / Conteggio iscritti
- Invito admin da waitlist / Creazione utenti con password temporanea

#### Subscription / billing

- Stato permessi utente / Trial 7 giorni
- Upgrade/downgrade piano / Checkout session
- Webhook provider pagamenti / Promo access admin / Storico promo utente
- Feature gating per tier

#### Trading journal e moduli di dominio

- Journal per trade/utente/id
- Supporto API anche per trade, wallet, notes, docs, dashboard, ecc.

#### Infrastruttura applicativa

- Persistenza PostgreSQL / ORM Hibernate/Panache / Migrazioni Flyway
- Cache / Scheduler / Health checks / OpenAPI / Mapping DTO / Validazione input

---

## 3. Cosa fanno insieme i due repository

Insieme, **project-marubozu-client** e **project-marubozu-server** implementano una piattaforma SaaS per trader che unisce:

- gestione account
- journaling operativo
- tracking trade
- analytics
- wallet/portfolio management
- documentazione e note personali
- subscription management
- onboarding/waitlist
- supporto multi-feature con gating per piano

In pratica, Marubozu è un hub personale per trader dove l'utente può:

- registrarsi o accedere via provider OAuth
- verificare l'email
- gestire profilo e avatar
- tracciare trade e diario operativo
- monitorare wallet, asset, exchange e transazioni
- salvare note e documenti
- consultare dashboard e analytics
- usare un trial e poi passare a piani superiori
- continuare a vedere in read-only alcune feature core anche se l'abbonamento è scaduto

---

## 4. Elenco finale delle funzionalità core

- Autenticazione completa / OAuth login / Verifica email e recupero password
- Gestione profilo utente / Cambio password obbligatorio in casi speciali
- Trading journal / Gestione trade / Dashboard e analytics
- Wallet tracking / Asset / Exchange / Transazioni
- Note personali / Documenti e cartelle / Upload file
- Mood / tagging / classificazione operativa
- Piani / regole / configurazioni
- Subscription con trial / Upgrade/downgrade piano / Promo access
- Waitlist e onboarding amministrato
- UI moderna con tema dark/light
- Feature gating per tier / Persistenza e API documentate

> **Nota:** Per la parte server e client il quadro è stato ricostruito dal codice e dalla struttura dei file disponibili; i risultati da code search possono essere incompleti. Un secondo passaggio può produrre: mappa completa feature-by-feature, tabella Client vs Server con responsabilità, analisi architetturale dei moduli, lista endpoint backend + relativa pagina frontend, documento stile README/prodotto pronto da condividere.

---

## Analisi funzionale dettagliata per sezione

### 1. Home

#### Cosa vede l'utente

La Home è la dashboard iniziale personale. Serve a dare in pochi secondi un quadro dell'attività di trading recente e della performance complessiva.

**KPI principali:**
- Net Profit
- Win Rate
- Total Trades
- Profit Factor

**KPI secondari:**
- Average Risk/Reward
- Average Win / Average Loss
- Max Drawdown

**Contenuto della pagina:**
- grafico Equity Curve
- grafico Drawdown
- tips operativi
- lista dei trade chiusi più recenti
- banner di stato subscription (trial, promo, expired)

#### Significato funzionale

Questa sezione non è solo "statistica": è una dashboard decisionale. L'utente può capire subito se sta performando bene o male, se il drawdown è sotto controllo, come si sta muovendo la curva del capitale.

#### Come è strutturata

Sul client usa un hook per i dati dashboard, uno per i trade recenti e uno per il profilo utente. Sul server è supportata da `DashboardResource` (`/api/dashboard/stats`) e `TradeResource` (`/api/trades/recent`).

> La Home rientra nelle feature **Core**, disponibile agli utenti con il piano base.

---

### 2. Trade Tracker

#### Cosa vede l'utente

Questa è la sezione dove l'utente vede tutti i propri trade in modo ordinato.

#### Funzionalità principali

- lista completa dei trade con filtri (status, direction) e ordinamento
- UI desktop a tabella / UI mobile a card
- creazione, aggiornamento, cancellazione trade
- accesso diretto al journal del singolo trade

#### Flusso creazione trade

Quando l'utente clicca su **Add Trade**:
1. se la subscription è scaduta → portato alla sezione profilo/subscription
2. altrimenti → creato un draft trade
3. aperta automaticamente la pagina del trade

Il sistema non obbliga a compilare tutto subito: prima crea la struttura, poi l'utente la completa nel journal.

#### Backend collegato

`TradeResource` espone: elenco completo/paginato, singolo trade, creazione/draft/aggiornamento/eliminazione, recent trades, aggiunta executions.

---

### 3. Journal del Trade

#### Cosa rappresenta

Il Journal è il cuore del prodotto. Qui il trade smette di essere solo una riga in una tabella e diventa una scheda completa di analisi, esecuzione e disciplina operativa.

#### Dati principali del trade

- entry, stop loss, take profit
- margin, asset, wallet
- direzione e stato del trade

#### Strato di analisi

- note di analisi sul trade
- time frame, mood
- contestualizzazione dell'operazione

#### Executions

L'endpoint dedicato `/api/trades/{tradeId}/executions` permette di gestire più esecuzioni reali per trade, supportando:

- ingressi multipli / scaling in
- prese di profitto parziali
- esecuzione non lineare del trade

#### Take profit multipli

Il trade supporta una lista di take profit creabile, modificabile ed eliminabile — coerente con operazioni che hanno target multipli.

#### Checklist

Il journal include una checklist operativa tramite `ChecklistResource`. L'utente può usarla per:

- verificare se il trade rispetta i criteri del piano
- mantenere disciplina operativa
- valutare il processo, non solo il risultato

#### Moduli collegati

trade / take profits / mood / timeframe / trade type / market pair / checklist / journal

---

### 4. Docs

#### Cosa vede l'utente

La sezione Docs funziona come una piccola area di documentazione personale strutturata, organizzata in 3 livelli visivi:

1. sidebar cartelle
2. lista file della cartella selezionata
3. editor del file

Su desktop le tre aree convivono affiancate. Su mobile c'è una navigazione a step (cartelle → file → editor).

#### Funzionalità

- creare, rinominare, eliminare cartelle
- aprire, modificare, spostare, eliminare file
- lavorare in una UI tipo knowledge base personale

Docs è pensato per: playbook operativi, strategie, template di analisi, regole di rischio, studio personale, documentazione di setup.

#### Backend collegato

`DocFolderResource` + `DocFileResource` con supporto a create/read/update/delete sia per cartelle che per file.

> Nel server compare anche un vecchio riferimento a una conversione file → trade, oggi deprecata, che suggerisce che in passato i documenti potevano essere un punto di partenza per costruire contenuti operativi legati al trading.

---

### 5. Notes

#### Cosa vede l'utente

La sezione Notes è pensata come un sistema rapido di annotazione, simile a **Google Keep**.

#### Funzionalità principali

- creazione nota rapida dal solo titolo
- apertura in modale con editor ricco
- pin nota / cambio colore / eliminazione
- caricamento progressivo con lista suddivisa in: **Pinned** / **Other**

#### Flusso rapido

1. scrivi il titolo → premi invio
2. la nota viene creata
3. si apre subito la modale di editing

#### Differenza rispetto a Docs

| | Docs | Notes |
|---|---|---|
| Struttura | gerarchica | libera |
| Orientamento | contenuti stabili | appunti veloci |
| Velocità | media | alta |

#### Backend collegato

`NoteResource` con lista completa/paginata/pinned, singola nota, create/update/delete.

> Se la subscription è **expired**, le note restano visibili ma alcune azioni di modifica vengono bloccate (modalità read-only).

---

### 6. Dashboard Macro

#### Cosa rappresenta

La Dashboard Macro è la sezione più "strategica" del prodotto: non guarda il singolo trade, ma il contesto generale dei mercati.

#### Blocchi funzionali

| Blocco | Descrizione |
|--------|-------------|
| **Economic Cycle** | Fase del ciclo economico secondo i segnali raccolti |
| **Macro Signals** | Segnali interpretativi che guidano la lettura del ciclo |
| **Money Flow** | Dove si spostano i capitali tra settori e classi di asset |
| **Intermarket Correlations** | Come si muovono tra loro gli asset principali |
| **Key Indicators** | Griglia di indicatori macro con valore recente e mini-storico |

Questa sezione copre: situazione generale dei mercati, rapporto oro/argento, forza del dollaro (DXY), dove si stanno muovendo i fondi.

#### Accesso

Questa sezione è protetta come feature **PRO** (non Core). Se l'utente non può accedere, il client mostra una pagina di feature bloccata.

#### Backend collegato

`MacroResource` espone: ciclo economico, indicatori, money flow, correlazioni, refresh manuale dei dati macro. La Dashboard Macro non è statica: è alimentata da una pipeline dati vera e propria.

---

### 7. Wallet

#### Cosa vede l'utente

La sezione Wallet è una **dashboard patrimoniale**.

#### Funzionalità principali

- wallet snapshot
- total balance / allocated balance / available balance / liquidity / PnL
- gestione exchange collegati
- depositi e prelievi
- asset e transazioni

#### Significato funzionale

Questa sezione amplia il concetto di journaling oltre il singolo trade, aiutando a leggere il proprio capitale di trading nel complesso: liquidità disponibile, capitale impegnato, patrimonio totale, risultato non realizzato.

#### Gating

Quando la subscription è expired, sono disabilitate: aggiunta exchange, deposit, withdraw. Il wallet resta visibile ma con limitazioni di modifica.

#### Backend collegato

`WalletResource`, `WalletBalanceResource`, `WalletTransactionResource`, `UserExchangeResource`, `AssetResource`, `MasterExchangeResource`

---

### 8. Autenticazione, profilo e onboarding

#### Accesso e registrazione

- Registrazione / Login email/password / Login OAuth
- Verifica email / Reinvio email di verifica
- Recupero e reset password / Logout / Refresh token

#### Profilo

- visualizzazione e aggiornamento profilo
- aggiornamento avatar

#### Onboarding amministrato

- waitlist pubblica
- inviti da parte admin
- creazione utenti da waitlist con password temporanee
- obbligo di cambio password al primo accesso

**Flusso chiave:** quando un utente viene invitato o ha una password temporanea, il backend blocca tutta l'operatività finché non completa il cambio password (logica `MustChangePasswordFilter` via JWT).

#### Backend collegato

`AuthResource`, `UserResource`, `WaitlistResource`

---

### 9. Subscription, trial e permessi

#### Funzionalità visibili all'utente

L'utente può trovarsi in questi stati:

- piano attivo / trial / subscription scaduta / promo access
- possibilità di upgrade/downgrade piano

#### Trial

Per i nuovi utenti il trial si attiva automaticamente per **7 giorni**.

#### Accesso per feature

| Tier | Accesso |
|------|---------|
| **Core** | Feature principali (Trades, Journal, Analytics, Wallet, Docs, Notes) |
| **Pro** | Feature avanzate (es. Dashboard Macro) |
| **Expired** | Feature core in modalità read-only, Pro bloccate |

#### Promo e pagamenti

Il backend supporta: upgrade/downgrade, sessioni checkout, webhook di pagamento, promo access gestite da admin, storico promo per utente.

---

### 10. Visione d'insieme del prodotto

> Marubozu è un **trading operating system personale**: unisce tracking, journaling, organizzazione della conoscenza e lettura macro in un unico ambiente.

#### I quattro pilastri del prodotto

| Pilastro | Sezioni |
|----------|---------|
| **Operatività** | Trade Tracker, Journal, Executions, Checklist, Wallet |
| **Performance** | Home Dashboard, KPI, Equity Curve, Drawdown, Recent Trades |
| **Conoscenza personale** | Docs, Notes, Analisi, Archiviazione |
| **Contesto di mercato** | Dashboard Macro, Ciclo Economico, Money Flow, Correlazioni |

#### Sintesi finale

Marubozu permette al trader di:

- registrare e gestire tutti i propri trade
- costruire un journal dettagliato per ogni operazione
- monitorare performance e drawdown
- vedere subito i KPI principali dalla Home
- seguire il proprio piano tramite checklist
- annotare mood, timeframe e analisi
- gestire documenti strutturati in cartelle e file
- tenere note veloci stile Google Keep
- monitorare wallet, exchange, asset e transazioni
- leggere il contesto macro e i flussi di mercato
- usare subscription, trial e feature gating in modo integrato
