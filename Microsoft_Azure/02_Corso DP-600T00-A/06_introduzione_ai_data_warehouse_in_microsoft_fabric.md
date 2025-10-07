---
autore: Kevin Milli
---

## Introduzione e Concetti Fondamentali del Data Warehouse

Il **Data Warehouse (DW)** è storicamente la **spina dorsale delle soluzioni di Business Intelligence (BI)** aziendali. La sua funzione principale è centralizzare e organizzare i dati provenienti da fonti eterogenee (sistemi transazionali, database, file, ecc.) in una **vista unificata e ottimizzata per l'analisi** e la creazione di report.

### Lo Schema Standard del Data Warehouse Relazionale

Tradizionalmente, per massimizzare le prestazioni delle query analitiche, un DW relazionale adotta uno **schema denormalizzato e multidimensionale**. Questo approccio, in contrasto con la normalizzazione dei database transazionali (OLTP), è progettato per:

1. **Ridurre i `JOIN`**: Mantenere i dati correlati (fatti e dimensioni) vicini per minimizzare i costosi `JOIN` in fase di query.
2. **Semplificare la Comprensione**: Fornire agli analisti un modello dati intuitivo (stellare o a fiocco di neve) che riflette il business.

## Il Data Warehouse in Microsoft Fabric: La Rivoluzione Basata su Lakehouse

Il **Data Warehouse di Microsoft Fabric** rappresenta l'evoluzione moderna del DW tradizionale, integrando il meglio dei sistemi relazionali con l'architettura dei Data Lake.

### Caratteristiche Distintive

|Caratteristica|Descrizione|Vantaggio|
|---|---|---|
|**Semantica SQL Completa**|Supporta **T-SQL transazionale** standard (`INSERT`, `UPDATE`, `DELETE`, `MERGE`), come un DW aziendale tradizionale.|Familiarità per gli sviluppatori SQL; gestione completa del ciclo di vita dei dati.|
|**Architettura Lakehouse**|È costruito nativamente sopra il **Lakehouse** di Fabric. I dati sono archiviati nel formato open-source **Delta Lake**.|Massima interoperabilità (leggibile da Spark, Python, R, ecc.) e separazione storage/compute per scalabilità.|
|**Query SQL su Lake**|Il motore SQL integrato esegue direttamente le query sui file Delta del Lakehouse.|Nessuna necessità di spostare o copiare i dati per l'analisi SQL.|
|**Piattaforma Unificata (OneLake)**|È parte di Fabric, che unifica data engineering, data science e BI.|Collaborazione semplificata tra team; esperienza **low-code** e tradizionale in un unico strumento.|

### Il Processo Moderno di Data Warehousing

La creazione e gestione di un DW moderno su Fabric segue quattro fasi chiave, gestibili all'interno della stessa piattaforma:

1. **Ingestione Dati**: Spostamento dei dati dai sistemi sorgente al Data Lakehouse (e al DW).
2. **Archiviazione Dati**: I dati vengono conservati in un formato ottimizzato per l'analisi (Delta Lake).
3. **Elaborazione/Trasformazione Dati**: Conversione dei dati grezzi in un formato strutturato e pronto per l'analisi (spesso utilizzando T-SQL o Spark).
4. **Analisi e Recapito**: Esecuzione di query (T-SQL) e creazione di report (Power BI) per estrarre _insight_ e distribuirli all'azienda.

---

## Progettazione del Data Warehouse: Modellazione Dimensionale

La **modellazione dimensionale** è la tecnica di design più diffusa per ottimizzare le prestazioni e l'usabilità del DW. Struttura i dati in due tipi principali di tabelle: **Fatti** (cosa è successo) e **Dimensioni** (chi, cosa, dove, quando, come).

### 1. Tabelle dei Fatti (Fact Tables)

Contengono i **dati numerici misurabili** (_misure_) relativi a un evento o una transazione aziendale (es. ordini di vendita, click su un sito, misurazioni di sensori).

- **Contenuto**: Misure quantitative (es. `ImportoTotale`, `QuantitaVenduta`, `Ricavi`).
- **Caratteristiche**:
    - Generalmente **molto grandi** (milioni o miliardi di righe).
    - Contengono **chiavi esterne** che collegano ogni riga alle rispettive **tabelle delle dimensioni**.
    - Sono la **fonte primaria** per le aggregazioni e l'analisi.

### 2. Tabelle delle Dimensioni (Dimension Tables)

Forniscono il **contesto descrittivo** ai dati contenuti nelle tabelle dei fatti, rispondendo alle domande _dove, quando, chi, cosa_.

- **Contenuto**: Attributi qualitativi e descrittivi (es. `NomeCliente`, `Indirizzo`, `CategoriaProdotto`).
- **Caratteristiche**:
    - Generalmente **più piccole** delle tabelle dei fatti.
    - Sono la base per il **filtraggio**, il **raggruppamento** e l'**analisi _slice-and-dice_**.

#### Le Chiavi nelle Dimensioni: Surrogata vs. Alternativa

Ogni riga in una tabella delle dimensioni è identificata univocamente da una coppia di chiavi che servono a scopi diversi:

|Tipo di Chiave|Scopo|Caratteristiche|
|---|---|---|
|**Chiave Surrogata** (Surrogate Key)|Identificatore **interno** del Data Warehouse.|Intero (`INT`) generato automaticamente (es. identità) non legato al sistema sorgente. **Garantisce l'univocità** e l'immutabilità nel DW, essenziale per gestire le **Dimensioni a Modifica Lenta (SCD)**.|
|**Chiave Alternativa** (Alternate Key)|Identificatore **naturale o aziendale** dal sistema sorgente.|Codice prodotto, ID cliente, numero d'ordine. **Mantiene la tracciabilità** e la relazione con il sistema OLTP.|

### Tipi Speciali di Tabelle delle Dimensioni

#### A. Dimensioni Temporali (Date/Time Dimensions)

Forniscono informazioni dettagliate sul **momento** in cui un evento è accaduto.

- **Utilità**: Consentono l'aggregazione flessibile dei fatti in base a gerarchie temporali ( Anno $\longrightarrow$  Trimestre $\longrightarrow$  Mese $\longrightarrow$  Giorno $\longrightarrow$ Ora ).
- **Esempio**: Una riga per ogni giorno contiene colonne come `Anno`, `GiornoDellaSettimana`, `Festivita`, ecc.

#### B. Dimensioni a Modifica Lenta (Slowly Changing Dimensions - SCD)

Sono dimensioni i cui attributi cambiano nel tempo. Il DW deve tracciare queste modifiche per garantire che le analisi storiche siano accurate. Ad esempio, se l'indirizzo di un cliente cambia, l'analisi deve poter correlare le vendite _prima_ e _dopo_ il cambio di indirizzo.

- **Importanza**: Assicurano l'integrità storica e l'accuratezza delle decisioni basate sui dati.

---

## Opzioni di Schema del Data Warehouse

Il modello dimensionale si concretizza in schemi specifici, i più comuni sono lo **Schema Star** e lo **Schema Snowflake**.

### 1. Schema Star (Schema a Stella)

È lo schema **più comune e preferito** per le prestazioni analitiche.

- **Struttura**: Una **tabella dei fatti centrale** è direttamente correlata a **più tabelle delle dimensioni** (_denormalizzate_), formando una struttura che assomiglia a una stella.
- **Vantaggi**:
    - **Semplicità di Query**: Richiede un solo `JOIN` tra il fatto e ogni dimensione, risultando in query più veloci.
    - **Intuitività**: Facile da comprendere per gli analisti.
- **Svantaggi**: Potenziale **ridondanza** dei dati (se gli attributi descrittivi sono ripetuti nella dimensione).

### 2. Schema Snowflake (Schema a Fiocco di Neve)

Si verifica quando le tabelle delle dimensioni sono **normalizzate** (suddivise) in più tabelle correlate tra loro.

- **Struttura**: Le tabelle delle dimensioni sono collegate ad **altre sottodimensioni**. Ad esempio, `DimProdotto` si collega a `DimCategoria` e `DimFornitore`.
- **Vantaggi**:
    - **Riduzione della Ridondanza**: Ottimo per dimensioni con molti attributi a diversi livelli di gerarchia (come nel tuo esempio: Geografia normalizzata da Cliente/Store).
    - **Manutenzione**: Più facile da gestire per gli aggiornamenti delle dimensioni.
- **Svantaggi**:
    - **Complessità di Query**: Richiede **più `JOIN`** per accedere agli attributi, il che può **rallentare le query** rispetto allo schema Star.

---

## Architettura e Funzionalità del Data Warehouse in Fabric

Il Data Warehouse di Fabric è un componente chiave dell'architettura **Lakehouse** di Microsoft Fabric, progettato per combinare la potenza transazionale di un database SQL con la flessibilità e la scalabilità di un Data Lake.

### Lakehouse vs. Data Warehouse in Fabric

L'architettura Fabric offre un'esperienza unificata e bidirezionale tra i dati nel **Lakehouse** e la struttura relazionale nel **Data Warehouse**.

|Componente|Ruolo Principale|Interfacce/Motori|Capacità di Modifica Dati|
|---|---|---|---|
|**Lakehouse**|Storage e processamento Big Data (livello dati).|Motore **Spark** (per Data Engineering) e **SQL Analytics Endpoint** (per Query SQL).|**Lettura** (query) e **Modifica** (tramite Spark o SQL Analytics Endpoint).|
|**Data Warehouse**|Modellazione relazionale (tabelle, viste) e funzionalità transazionali complete.|Motore **T-SQL** completo.|**Modifica DML completa** (`INSERT`, `UPDATE`, `DELETE`) sui dati.|

#### Endpoint di Analisi SQL (SQL Analytics Endpoint)

L'Endpoint di Analisi SQL è un meccanismo che **espone il Lakehouse come un DW relazionale**. Permette di eseguire query SQL sui file e le tabelle del Lakehouse. L'Endpoint è principalmente focalizzato sulla lettura, mentre l'esperienza del Data Warehouse offre la capacità di **modellare** e **modificare** i dati in modo transazionale e completo tramite T-SQL.

### Creazione e Accesso al Data Warehouse

Creare un DW in Fabric è un processo semplice e integrato:

1. **Creazione**: Può essere creato direttamente dall'**Hub di Creazione** di Fabric o all'interno di un'**Area di Lavoro** esistente.
2. **Modellazione e Query**: Una volta creato, si possono aggiungere oggetti come tabelle e viste. L'interfaccia di Fabric o client SQL esterni (come **SSMS**) possono essere usati per connettersi ed eseguire comandi **T-SQL** per querying (query) e Data Manipulation Language (DML).

---

## Inserimento Dati (Data Ingestion)

Fabric offre una vasta gamma di strumenti per caricare i dati nel Data Warehouse, sfruttando l'integrazione nativa con la piattaforma.

### Metodi di Ingestione Principali

1. **Pipeline di Dati (Data Pipelines)**: Strumento ETL/ELT basato sul cloud per l'orchestrazione e lo spostamento di dati da diverse origini. Ideale per carichi batch complessi.
2. **Dataflows Gen2**: Strumento di preparazione dati _low-code_ che utilizza Power Query, ottimo per trasformazioni di dati interattive e a basso volume prima dell'inserimento.
3. **Query Tra Database (Cross-Database Query)**: Tecnica che consente di leggere dati da altre posizioni in Fabric (come Lakehouse o altri DW) e inserirli nel DW di destinazione.
4. **`COPY INTO`**: Comando T-SQL altamente ottimizzato per l'inserimento di grandi quantità di dati da file esterni (es. Archiviazione BLOB di Azure, S3) direttamente nelle tabelle del DW.

#### Esempio Pratico: Comando `COPY INTO`

Il comando `COPY INTO` è il metodo preferito per l'ingestione di massa da storage esterni.
```SQL
COPY INTO dbo.Region 
FROM 'https://mystorageaccountxxx.blob.core.windows.net/private/Region.csv' 
WITH ( 
    FILE_TYPE = 'CSV' /* Specifica il formato del file sorgente */
    ,CREDENTIAL = ( 
        IDENTITY = 'Shared Access Signature' /* Metodo di autenticazione */
        , SECRET = 'xxx' /* Token SAS per l'accesso */
        ) 
    ,FIRSTROW = 2 /* Salta la prima riga (header) */
    )
GO
```

**Intuizione**: A differenza del tradizionale `BULK INSERT`, il `COPY INTO` in Fabric (basato sul motore Synapse SQL) è ottimizzato per il caricamento massivo e parallelo direttamente dallo storage cloud, bypassando il bisogno di un server intermedio.

### Creazione di Tabelle (CREATE TABLE)

Le tabelle possono essere create tramite:

1. **Interfaccia Utente di Fabric**: Strumenti grafici per la creazione rapida.
2. **Client SQL Esterni (SSMS)**: Connessione al DW tramite l'endpoint SQL e esecuzione di istruzioni **`CREATE TABLE`** standard T-SQL.

---

## Gestione Avanzata delle Tabelle

### Clonazione delle Tabelle (Zero-Copy Cloning)

Una funzionalità fondamentale di Fabric è la possibilità di creare **cloni di tabelle a copia zero (zero-copy clones)**.

- **Principio**: Invece di duplicare i file Parquet sottostanti in OneLake, il clone crea una nuova voce di metadati che punta agli **stessi file di dati** della tabella originale.
- **Vantaggi**:
    - **Costo Zero o Minimo**: Non duplica lo storage dei dati.
    - **Velocità Istantanea**: La clonazione è quasi immediata, poiché vengono copiati solo i metadati.

|Scenario d'Uso|Descrizione|
|---|---|
|**Sviluppo e Test**|Creazione rapida di ambienti di _sandbox_ per testare nuove trasformazioni o patch senza alterare i dati di produzione.|
|**Ripristino Dati (Point-in-Time)**|Mantenere cloni che rappresentano lo stato dei dati in un momento critico (_snapshot_) per un potenziale ripristino in caso di corruzione.|
|**Reportistica Storica**|Catturare lo stato dei dati in momenti significativi (es. fine trimestre) per report che richiedono una visione immutabile.|

**Sintassi (T-SQL)**:
```SQL
CREATE TABLE dbo.TabellaTest AS CLONE OF dbo.TabellaProduzione;
```

### La Strategia delle Tabelle di Staging (Staging Tables)

Il processo di caricamento dei dati in un DW, soprattutto per i carichi batch periodici, dovrebbe sempre utilizzare un'area temporanea: le **Tabelle di Staging**.

- **Definizione**: Tabelle temporanee all'interno del DW utilizzate per ospitare i dati grezzi o parzialmente trasformati appena estratti dalla sorgente.
- **Funzione**: Servono da ponte tra l'ingestione e le tabelle finali (Fatti e Dimensioni), consentendo:
    1. **Pulizia e Convalida**: Identificazione e correzione di errori di formato o valori mancanti.
    2. **Trasformazione Dati**: Applicazione di logiche complesse (es. calcoli, normalizzazione).
    3. **Gestione Delle Chiavi**: Ricerca delle **Chiavi Surrogate** per le righe delle dimensioni prima di caricare la Tabella dei Fatti.

#### Ciclo Ideale di Caricamento (ETL/ELT)

Un processo di caricamento robusto segue una sequenza logica per garantire l'integrità e la coerenza del DW:

1. **Inserimento in Lake/Staging**: I dati grezzi vengono caricati nei file del Data Lake (o direttamente nelle tabelle di staging del DW).
2. **Caricamento Dimensioni**:
    - I dati delle dimensioni vengono estratti dalle tabelle di staging.
    - Vengono aggiornate le righe esistenti e inserite le nuove righe.
    - Vengono generate le **Chiavi Surrogate**.
3. **Caricamento Fatti**:
    - I dati dei fatti vengono estratti dallo staging.
    - Per ogni riga del fatto, vengono ricercate (tramite `JOIN`) le **Chiavi Surrogate** dalle Dimensioni (usando le Chiavi Alternative come lookup).
    - La riga completa (con le chiavi surrogate) viene inserita nella Tabella dei Fatti.
4. **Ottimizzazione Post-Caricamento**: Aggiornamento delle statistiche di distribuzione e gestione degli indici (se applicabile) per garantire prestazioni ottimali nelle query.

### Query Inter-Database (Cross-Database Querying)

Un'abilità potente di Fabric è la possibilità di eseguire query sui dati in un Lakehouse **direttamente** da un Data Warehouse, senza copiare i dati.

- **Meccanismo**: Sfrutta la connessione nativa all'architettura OneLake.
- **Vantaggio**: Se il Lakehouse contiene dati che devono solo essere analizzati ma non modificati all'interno del contesto relazionale del DW, è possibile accedervi direttamente, **evitando ridondanza** e riducendo i tempi di caricamento.
- **Esempio (Concept)**: Eseguire un `SELECT` nel DW che unisce una tabella dei fatti del DW con una tabella di staging o una tabella Delta nel Lakehouse.

---

## Eseguire Query e Trasformare i Dati nel Data Warehouse

Il Data Warehouse di Fabric fornisce un ambiente di query flessibile che si adatta a diversi livelli di competenza degli utenti, dal **low-code** T-SQL avanzato.

### Strumenti di Query Principali

Fabric offre due interfacce principali integrate per interrogare il Data Warehouse (e il Lakehouse tramite _cross-database querying_):

|Strumento|Descrizione|Target Utente|Vantaggi|
|---|---|---|---|
|**Editor di Query SQL**|Ambiente di sviluppo T-SQL completo (simile a SSMS o Azure Data Studio).|Sviluppatori SQL, Data Engineer.|Supporta **IntelliSense**, completamento del codice, evidenziazione della sintassi e l'esecuzione di comandi T-SQL per creare **tabelle, viste e _stored procedure_**.|
|**Editor di Query Visuale**|Interfaccia grafica **drag-and-drop** e **low-code**.|Analisti di dati, utenti con scarsa esperienza SQL.|Permette di costruire query complesse tramite la **trasformazione** visiva dei dati (aggiunta di colonne, filtri, _merge_, ecc.), in modo analogo alla _Diagram View_ di Power Query.|

**Intuizione**: L'obiettivo di Fabric è fornire l'accesso SQL relazionale al Lakehouse. L'**Endpoint di Analisi SQL** è il punto di connessione _universale_ che consente a qualsiasi strumento esterno (es. Power BI Desktop, SSMS, ADS) di connettersi al DW e al Lakehouse.

### Esecuzione di Query Avanzate

L'Editor di Query SQL è lo strumento ideale per la preparazione avanzata dei dati analitici, come la creazione di oggetti di persistenza:

- **Viste (Views)**: Tabelle virtuali che incapsulano query complesse. Sono essenziali per fornire agli analisti una vista semplificata e controllata dei dati, ad esempio una vista denormalizzata pronta per il reporting.
- **Stored Procedure**: Sequenze di comandi T-SQL riutilizzabili per automatizzare operazioni DML o logiche di trasformazione complesse.

---

## Preparare i Dati per l'Analisi e il Reporting: Il Modello Semantico

La fase successiva all'interrogazione è la preparazione dei dati per gli strumenti di Business Intelligence. In Fabric, il contenitore di questa logica è il **Modello Semantico**.

### Cos'è un Modello Semantico?

Un **Modello Semantico** (precedentemente noto come _Dataset_ in Power BI) è uno strato logico che definisce la **Business Logic** sui dati grezzi del Data Warehouse.

Definisce:

1. **Relazioni**: I collegamenti tra le tabelle (Fatti e Dimensioni).
2. **Misure (Measures)**: Calcoli complessi (metriche) per l'analisi.
3. **Gerarchie**: Percorsi predefiniti per l'analisi drill-down (es. Anno ![](data:,) Mese ![](data:,) Giorno).

Il Modello Semantico è l'unica interfaccia a cui si connettono gli strumenti di reporting come Power BI.

### Viste del Data Warehouse in Fabric

L'esperienza del DW in Fabric offre tre viste per gestire il ciclo di vita dei dati:

- **Vista Dati**: Mostra le tabelle e il contenuto del Data Warehouse.
- **Vista Query**: Utilizzata per scrivere ed eseguire query SQL o visuali.
- **Vista Modello**: L'interfaccia principale per la definizione del Modello Semantico.

### Costruire la Business Logic nella Vista Modello

#### 1. Creare Relazioni
Le **relazioni** (tramite chiavi surrogate) sono l'elemento fondamentale del modello dimensionale. Vengono create nella **Vista Modello** tramite un'interfaccia **drag-and-drop** che collega le colonne chiave tra le tabelle (es. `FactSales` e `DimCustomer`).

#### 2. Creare Misure (Measures)
Le **Misure** sono i calcoli dinamici che rispondono alle domande aziendali (es. "Fatturato Anno Precedente", "Margine Lordo").

- Vengono create tramite il pulsante **Nuova Misura** nella Vista Modello.
- Utilizzano il linguaggio **DAX (Data Analysis Expressions)**, il linguaggio standard per i modelli semantici Microsoft (Power BI, SSAS).

#### 3. Nascondere i Campi
È una pratica di modellazione fondamentale per l'usabilità del report.

- Si **nascondono** le colonne che non devono essere utilizzate direttamente dai creatori di report (es. le Chiavi Surrogate o le colonne di staging).
- La colonna rimane disponibile per l'uso interno nelle Misure e nelle relazioni del modello, ma viene rimossa dalla vista del report, semplificando la navigazione per l'utente finale.

---

## Il Modello Semantico Predefinito (Default Semantic Model)

Fabric automatizza una parte significativa della preparazione dei dati creando un **Modello Semantico Predefinito** per ogni Data Warehouse o Lakehouse.

### Caratteristiche del Modello Predefinito

|Caratteristica|Implicazione|
|---|---|
|**Generazione Automatica**|Viene creato automaticamente con il DW/Lakehouse.|
|**Ereditarietà Logica**|Eredita la struttura (tabelle e viste) dal DW padre.|
|**Sincronizzazione Automatica**|Qualsiasi modifica (nuova tabella, nuova colonna) nel DW viene **automaticamente sincronizzata** con il modello semantico predefinito. Elimina la gestione manuale del _dataset_.|
|**Ottimizzazione**|Il modello è gestito e ottimizzato automaticamente da Fabric.|
|**Flessibilità**|Gli utenti possono **selezionare manualmente** quali tabelle o viste dal DW includere nel modello predefinito. Gli oggetti inclusi vengono mappati come _layout_ nella Vista Modello.|

**Intuizione**: Questo modello predefinito funge da **punto di partenza immediato** per la BI. Consente agli analisti di iniziare a creare report _istantaneamente_ dopo che il Data Engineer ha terminato il caricamento dei dati, riducendo drasticamente il tempo di _time-to-insight_.

## Visualizzazione Immediata dei Dati

Fabric non richiede di uscire dall'ambiente per validare i dati.

- **Visualizzazione in-situ**: È possibile visualizzare i risultati di una **singola query** o i dati di un'**intera tabella** direttamente nell'interfaccia di query.
- **Generazione di Report Power BI**: Il pulsante **Nuovo Report** crea e lancia automaticamente un nuovo report di **Power BI Service** collegato al modello semantico del Data Warehouse. Questo permette la creazione e la condivisione rapida di report aziendali.

## Protezione e Monitoraggio del Data Warehouse

La gestione efficace di un Data Warehouse moderno richiede un'attenzione costante a due pilastri fondamentali: **Sicurezza** (per proteggere i dati) e **Monitoraggio** (per garantire prestazioni e utilizzo efficiente).

## Sicurezza (Security)
Fabric offre un approccio di sicurezza a più livelli, sfruttando l'integrazione nativa con l'ecosistema Microsoft.

### Misure di Sicurezza di Fabric

|Funzionalità di Sicurezza|Descrizione|
|---|---|
|**Microsoft Entra ID (già Azure AD)**|Gestione centralizzata delle identità e dell'accesso utente (IAM).|
|**Autenticazione a più Fattori (MFA)**|Aggiunge un livello di sicurezza critico agli accessi.|
|**Crittografia TLS (Transport Layer Security)**|Protegge la comunicazione (_dati in transito_) tra il Data Warehouse e le applicazioni client.|
|**Crittografia del Servizio di Archiviazione di Azure**|Protegge i _dati inattivi_ (a riposo) nel sottostante OneLake.|
|**Controllo degli Accessi in Base al Ruolo (RBAC)**|Definisce chi può accedere, modificare o gestire il Data Warehouse e i relativi dati.|
|**Monitoraggio e Audit**|Utilizzo di strumenti come **Monitoraggio di Azure** e **Azure Log Analytics** per tracciare l'attività del warehouse e registrare gli accessi ai dati.|

### Gestione delle Autorizzazioni

L'accesso ai dati in Fabric è gestito gerarchicamente:

#### 1. Ruoli dell'Area di Lavoro (Workspace Roles)
Sono la prima linea di difesa e si applicano a **tutti gli elementi** (Data Warehouse, Lakehouse, Notebook, ecc.) all'interno di un'Area di Lavoro. Definiscono i diritti di alto livello (es. Amministratore, Membro, Collaboratore, Visualizzatore).

#### 2. Autorizzazioni per gli Elementi (Item Permissions)
Consentono un controllo dell'accesso più granulare per **singoli Data Warehouse**. Sono cruciali per scenari di _condivisione downstream_, dove si vuole concedere l'accesso a un singolo DW senza esporre l'intera Area di Lavoro.

Le autorizzazioni chiave concesse agli utenti che accedono al DW tramite l'Endpoint di Analisi SQL sono:

|Autorizzazione|Funzionalità|Pre-Requisito|
|---|---|---|
|**`Read`**|Consente all'utente di **connettersi** tramite la stringa di connessione SQL.|_Minimo indispensabile_ per la connessione.|
|**`ReadData`**|Consente all'utente di **leggere i dati** da _qualsiasi tabella o vista_ all'interno del warehouse.|Necessario per l'analisi SQL standard.|
|**`ReadAll`**|Consente all'utente di leggere i dati dei **file Parquet non elaborati in OneLake**.|Utile per gli strumenti di _Data Engineering_ basati su Spark.|

**Nota**: Senza l'autorizzazione minima di **`Read`**, la connessione all'Endpoint di Analisi SQL fallisce.

## Monitoraggio (Monitoring)

Il monitoraggio è vitale per la salute del Data Warehouse, poiché consente l'identificazione precoce di colli di bottiglia e la gestione efficiente delle risorse.

### Viste a Gestione Dinamica (DMV - Dynamic Management Views)

Le DMV sono strumenti potenti basati su T-SQL che espongono informazioni dinamiche sullo stato e le prestazioni del motore SQL. Sono usate per monitorare:

- **Stato della Connessione**: Chi è connesso.
- **Stato della Sessione**: Dettagli su ogni sessione autenticata.
- **Stato della Richiesta**: Dettagli sulle query SQL attive.

#### DMV Disponibili in Fabric:

1. **`sys.dm_exec_connections`**: Informazioni su ogni **connessione** stabilita (chi e come si è connesso).
2. **`sys.dm_exec_sessions`**: Informazioni su ogni **sessione** autenticata (contesto utente e ambiente).
3. **`sys.dm_exec_requests`**: Informazioni su ogni **richiesta (query)** attualmente in esecuzione.

### Monitoraggio e Gestione delle Query

Le DMV sono usate per l'identificazione e la risoluzione dei problemi, in particolare per le query che consumano troppe risorse.

#### 1. Identificare le Query a Lunga Esecuzione

Si utilizza la DMV `sys.dm_exec_requests` per trovare le query attive che hanno il tempo trascorso maggiore:
```SQL
SELECT request_id, session_id, start_time, total_elapsed_time
FROM sys.dm_exec_requests
WHERE status = 'running'
ORDER BY total_elapsed_time DESC;
```

#### 2. Identificare l'Utente Responsabile

Una volta individuato l'ID della sessione (`session_id`) della query problematica, si usa la DMV `sys.dm_exec_sessions` per risalire all'utente che l'ha eseguita:
```SQL
SELECT login_name
FROM sys.dm_exec_sessions
WHERE session_id = 'ID_SESSIONE_CON_QUERY_LENTA'; -- Sostituire con l'ID reale
```

#### 3. Terminare la Sessione (KILL)

Se una query a lunga esecuzione compromette le prestazioni del sistema, può essere terminata forzatamente usando il comando `KILL`:
```SQL
KILL 'ID_SESSIONE_CON_QUERY_LENTA'; -- Sostituire con l'ID reale
```

**Permessi per il Monitoraggio DMV**:

- **Amministratori dell'Area di Lavoro**: Possono eseguire tutte e tre le DMV e visualizzare i risultati di tutti gli utenti. Sono gli unici autorizzati a eseguire il comando **`KILL`**.
- **Membri/Collaboratori/Visualizzatori**: Possono eseguire le DMV, ma visualizzano solo i risultati relativi alle **proprie** connessioni, sessioni e richieste all'interno del Data Warehouse.

