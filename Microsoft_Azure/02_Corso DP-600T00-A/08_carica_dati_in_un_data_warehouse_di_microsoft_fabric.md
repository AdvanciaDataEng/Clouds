---
autore: Kevin Milli
---

## Introduzione a Microsoft Fabric Data Warehouse

**Microsoft Fabric** è una piattaforma unificata completa per dati, analisi e intelligenza artificiale (AI), che centralizza l'archiviazione e integra funzionalità avanzate.

Il **Data Warehouse in Microsoft Fabric** è una delle esperienze chiave, potenziata da **Synapse Analytics**. Offre prestazioni e scalabilità di livello enterprise, supportando funzionalità **T-SQL transazionali complete** per un'architettura moderna di data warehousing.

### Punti Chiave dell'Architettura

- **OneLake**: È il cuore del data warehousing e di tutte le altre esperienze di Fabric (come Data Engineering, Data Factory, Power BI). OneLake è il **Single Store** unificato per tutti i dati analitici dell'organizzazione, agendo come il "OneDrive per i dati".
    
    - **Architettura**: Costruito su **Azure Data Lake Storage (ADLS) Gen2**.
    - **Formato Dati**: I dati, sia che provengano da un Lakehouse che da un Warehouse, vengono archiviati nativamente nel formato aperto **Delta Parquet** in OneLake. Questo assicura che ci sia una **singola copia dei dati** accessibile da più motori analitici (T-SQL, Spark, Power BI) senza necessità di duplicazione o spostamento.
- **Separazione di Calcolo e Archiviazione**: Il Data Warehouse di Fabric separa il compute (motore SQL di Synapse) dallo storage (OneLake), consentendo una scalabilità indipendente e nativa.

## Processo di Integrazione Dati: ETL vs. ELT

Il processo fondamentale per popolare un Data Warehouse è l'integrazione dei dati. Storicamente dominato da **ETL**, le moderne piattaforme cloud come Fabric tendono a favorire l'approccio **ELT**.

|Processo|Sequenza|Descrizione|Contesto Moderno|
|---|---|---|---|
|**ETL**|**E**xtract, **T**ransform, **L**oad (Estrazione, Trasformazione, Caricamento)|La trasformazione avviene su un server di staging o un motore di elaborazione dedicato **esterno** al Data Warehouse. Solo i dati puliti e conformi vengono caricati.|Migliore per la conformità (pulisce i dati sensibili prima del caricamento) o per carichi di lavoro legacy/più piccoli.|
|**ELT**|**E**xtract, **L**oad, **T**ransform (Estrazione, Caricamento, Trasformazione)|I dati grezzi vengono **caricati direttamente** nello storage di destinazione (OneLake in Fabric), e la trasformazione avviene **all'interno** del Data Warehouse (sfruttando la potenza del motore SQL o Spark).|Ideale per Big Data e Cloud Data Warehouse. **Più veloce** (carica dati grezzi in parallelo) e **più flessibile** (gestisce dati strutturati e non strutturati e permette trasformazioni _on-demand_).|

**Intuizione per Fabric**: L'architettura unificata di Fabric, basata su OneLake (Data Lake) e sul potente motore Synapse (compute), è **intrinsecamente orientata a ELT**. Si caricano i dati grezzi in OneLake e si usa la potenza del Warehouse o del Lakehouse per le trasformazioni (T-SQL, Spark, Dataflow Gen2, Pipeline).

## Strategie di Caricamento Dati in Fabric

Il caricamento dei dati è la fase in cui i dati elaborati o grezzi vengono spostati nello storage finale (tabelle delle dimensioni e dei fatti) per l'analisi.

### 1. Inserimento Dati vs. Caricamento Dati

- **Inserimento/Estrazione (Ingestione)**: Spostamento dei dati _grezzi_ dalle sorgenti in un repository centrale, in genere l'area di staging o direttamente in OneLake (nel Lakehouse).
- **Caricamento**: Spostamento dei dati _trasformati o elaborati_ nella destinazione finale per l'analisi (**tabelle dei fatti e delle dimensioni** nel Data Warehouse).

### 2. Metodi di Caricamento Dati in Fabric

Il Data Warehouse di Fabric supporta diverse tecniche di caricamento ad alte prestazioni:

|Metodo|Descrizione|Scenario d'Uso|
|---|---|---|
|**Pipeline (Copy Activity)**|Uso di **Data Factory** in Fabric per un'operazione di copia di dati gestita e orchestrata.|Ideale per l'automazione dei carichi di lavoro (sia completi che incrementali) da varie fonti di dati a OneLake/Warehouse.|
|**`COPY INTO`**|Istruzione T-SQL ottimizzata per il caricamento in blocco di dati direttamente da file in OneLake.|Caricamento ad alte prestazioni di file di grandi dimensioni (ad esempio, Parquet o CSV) che risiedono in OneLake.|
|**Operazioni T-SQL (CTAS/INSERT..SELECT)**|Utilizzo di `CREATE TABLE AS SELECT` (CTAS) o `INSERT INTO ... SELECT` per operazioni di caricamento e trasformazione inter-database o inter-tabelle.|Trasformazioni basate su SQL (approccio ELT) direttamente all'interno del Warehouse. **Raccomandato** per l'inserimento di grandi quantità di dati rispetto agli `INSERT` singoli per motivi di prestazioni.|
|**Spark**|Uso di notebook Spark (Data Engineering) per scrivere dati in blocco direttamente nelle tabelle Delta/Parquet in OneLake.|Trasformazioni complesse o su Big Data in linguaggio come PySpark, con scrittura diretta nel formato aperto.|
|**Dataflow Gen2**|Strumento ETL/di trasformazione a basso codice in Fabric, che può caricare direttamente nel Warehouse.|Ideale per sviluppatori non SQL per modellare e caricare dati.|

### 3. Tipi di Caricamento

|Tipo di Caricamento|Strategia|Vantaggi|Svantaggi/Note|
|---|---|---|---|
|**Caricamento Completo (Iniziale)**|Le tabelle di destinazione vengono troncate e completamente ricaricate con tutti i dati più recenti.|Facile da implementare, garantisce l'accuratezza totale dei dati (ripartenza pulita).|Non mantiene la cronologia, richiede più tempo per grandi volumi di dati.|
|**Caricamento Incrementale**|Vengono caricate solo le modifiche (nuovi record, aggiornamenti, eliminazioni) avvenute dall'ultima esecuzione.|Più veloce, riduce l'impatto sul Warehouse, essenziale per la gestione della cronologia.|Più complesso da implementare, richiede meccanismi di **Change Data Capture (CDC)** o **Change Tracking** sulla fonte.|

## Staging dei Dati (Staging)

Lo **staging** si riferisce a un'area di archiviazione temporanea dove i dati estratti vengono tenuti e manipolati prima di essere caricati nelle tabelle finali del Data Warehouse.

- **Funzione**: Serve come buffer per la **pulizia, la convalida e la trasformazione** preliminare dei dati estratti.
- **Vantaggi**:
    - **Isolamento**: Riduce al minimo l'impatto delle operazioni ETL/ELT sulle prestazioni di query del Data Warehouse finale.
    - **Semplicità**: Semplifica il processo di caricamento nelle tabelle finali (Fatti e Dimensioni).
    - **Gestione degli Errori**: Permette di isolare ed esaminare i dati non validi o che causano errori prima che inquinino il Data Warehouse.
- **Contesto Fabric**: In un'architettura Lakehouse, il concetto di "staging" può essere rappresentato dai **dati grezzi (raw data)** archiviati come file in OneLake prima di essere promossi a tabelle Delta o caricati nel Warehouse.

## Chiavi: Business vs. Sostitutive

|Chiave|Definizione|Ruolo nel Data Warehouse|Esempi|
|---|---|---|---|
|**Chiave Business (Naturale)**|Identificatore che proviene dal sistema di origine (OLTP) e ha un **significato aziendale**.|Utilizzata per il **tracciamento** e l'integrazione con i sistemi sorgente.|Codice Prodotto, ID Cliente, Codice Fiscale.|
|**Chiave Sostitutiva (Surrogate)**|Identificatore **generato dal sistema** (spesso un intero incrementale) che non ha significato aziendale.|**Chiave primaria** delle tabelle Dimensioni e chiave esterna nelle tabelle Fatti. Assicura **coerenza** e permette di gestire le **Dimensioni a Modifica Lenta (SCD)**.|`ProductKey`, `CustomerKey`.|

**Intuizione**: La Chiave Sostitutiva isola il Data Warehouse dai problemi del sistema sorgente (es. chiavi riutilizzate o modificate) ed è cruciale per la gestione della cronologia nelle SCD.

## Caricamento delle Tabelle Dimensioni

Le **Tabelle Dimensioni** forniscono il **contesto descrittivo** (_Chi, Cosa, Dove, Quando, Perché_) ai dati numerici nelle tabelle dei Fatti. Sono centrali nel modello dimensionale (Star/Snowflake Schema).

### Dimensioni a Modifica Lenta (Slowly Changing Dimensions - SCD)

Le SCD sono record descrittivi (come l'indirizzo di un cliente) che cambiano nel tempo in modo **lento e imprevedibile**. L'implementazione di una SCD definisce come viene gestita la cronologia di queste modifiche.

|Tipo SCD|Definizione|Meccanismo|Implementazione|
|---|---|---|---|
|**Tipo 0**|**Fissa**. L'attributo non cambia mai.|Nessuna gestione.|Utile per attributi statici (es. Data di Nascita).|
|**Tipo 1**|**Sovrascrittura**. Il valore precedente viene aggiornato con il nuovo.|**Aggiorna** il record esistente (perdita della cronologia).|Facile da implementare. Usato quando la cronologia non è necessaria, ma solo l'attributo corrente.|
|**Tipo 2**|**Aggiunta di Riga**. Viene creato un nuovo record per la modifica.|**Mantiene la cronologia completa** tramite l'uso di chiavi surrogate, indicatori di attività (`IsActive`) e intervalli di validità (`ValidFrom`, `ValidTo`).|Essenziale per l'analisi storica, per associare i fatti allo **stato della dimensione al momento del fatto**.|
|**Tipo 3**|**Nuova Colonna**. Aggiunge una colonna per memorizzare il valore precedente.|Mantiene solo una **cronologia limitata** (il valore corrente e uno o più valori precedenti).|Meno comune, usato per un facile accesso all'ultimo stato precedente.|
|**Tipo 6 (Ibrido)**|Combinazione di Tipo 1, 2 e 3.|Utilizza la riga aggiunta (Tipo 2) con una colonna di attributo corrente sovrascritta (Tipo 1) e talvolta una colonna "precedente" (Tipo 3).|Fornisce la massima flessibilità per l'analisi corrente e storica.|

#### Esempio di Implementazione SCD Tipo 2 (T-SQL)

L'SCD Tipo 2 è il più comune per l'analisi storica. Quando un record (identificato da `SourceKey`) cambia, il record precedente viene **scaduto** (`IsActive = 'False'`, `ValidTo = DataCorrente`) e viene **inserito un nuovo record** (`IsActive = 'True'`, `ValidTo = '9999-12-31'`).

```SQL
-- Pseudocodice T-SQL per la gestione SCD Tipo 2 (Tabella: Dim_Products)
IF EXISTS (SELECT 1 FROM Dim_Products WHERE SourceKey = @ProductID AND IsActive = 'True')
BEGIN
    -- 1. Se il prodotto esiste e la sua versione corrente è attiva:
    --    - Scadenza della versione precedente (Type 2 logic)
    UPDATE Dim_Products
    SET ValidTo = GETDATE(), IsActive = 'False'
    WHERE SourceKey = @ProductID
        AND IsActive = 'True'
        AND (
            -- Condizione aggiuntiva: verifica se l'attributo è effettivamente cambiato.
            -- Per semplicità, ipotizziamo una modifica, altrimenti si aggiungerebbe la logica di confronto.
            1 = 1 -- Sostituire con una verifica di cambio attributo
        );
END

-- 2. Inserimento della nuova versione (nuovo record)
INSERT INTO Dim_Products (SourceKey, ProductName, ValidFrom, ValidTo, IsActive)
VALUES (@ProductID, @ProductName, GETDATE(), '9999-12-31', 'True');
```

**Meccanismi di Rilevamento Modifiche**: Per implementare un carico incrementale (incluso SCD), la logica deve sapere cosa è cambiato. Strumenti come **Change Data Capture (CDC)**, **Change Tracking** (in SQL Server/Azure SQL) o semplicemente l'uso di colonne di timestamp (`LastModifiedDate`) nel sistema di origine sono cruciali.

## Caricamento delle Tabelle Fatti

Le **Tabelle Fatti** contengono le **misure** (eventi o transazioni come vendite, accessi) e le chiavi esterne (le chiavi surrogate) per le tabelle delle dimensioni. Vengono caricate in genere **dopo** le tabelle delle dimensioni.

### Logica di Ricerca (Lookup) delle Chiavi Sostitutive

Il passaggio cruciale è la **ricerca** (lookup) delle chiavi surrogate corrispondenti a partire dalle chiavi business presenti nei dati di staging.

- **Processo Standard**: Eseguire un `JOIN` tra la tabella di staging (che contiene le chiavi business) e le tabelle delle dimensioni (che contengono sia le chiavi business che le chiavi surrogate).
- **Gestione SCD Tipo 2**: Quando si carica un fatto, è vitale associare l'evento al **corretto stato storico della dimensione**. Per un SCD Tipo 2, non basta trovare la chiave surrogate, ma bisogna trovare quella la cui validità copre la data dell'evento.

#### Esempio di Lookup nel Caricamento Fatti (T-SQL)

L'esempio seguente è stato migliorato per illustrare la **corretta gestione dell'SCD Tipo 2** per una dimensione storica, in contrasto con l'esempio originale che usava `MAX(Key)`:

```SQL
-- Utilizzo di CTE (Common Table Expression) o JOIN per lookup più efficiente e SCD Type 2
INSERT INTO dbo.FactSales
(OrderDateKey, CustomerKey, ProductKey, StoreKey, OrderNumber, OrderLineItem, OrderQuantity, UnitPrice, Discount, Tax, SalesAmount)
SELECT
    -- 1. Lookup per la Dimensione Data (tipicamente SCD Tipo 0/1, ma può essere ricercata)
    DD.DateKey AS OrderDateKey,

    -- 2. Lookup SCD Tipo 2 per Dimensione Cliente
    DC.CustomerKey,

    -- 3. Lookup SCD Tipo 2 per Dimensione Prodotto
    DP.ProductKey,

    -- 4. Lookup per Dimensione Store (assumendo SCD Tipo 1 o 2)
    DS.StoreKey,

    -- 5. Misure e altri attributi del Fatto
    stg.OrderNumber,
    stg.OrderLineItem,
    stg.OrderQuantity,
    stg.UnitPrice,
    stg.Discount,
    stg.Tax,
    stg.SalesAmount
FROM dbo.StageSales AS stg
-- Lookup Dimensione Data (ricerca su chiave naturale e data, se necessario)
INNER JOIN dbo.DimDate AS DD ON stg.OrderDate = DD.FullDateAlternateKey

-- Lookup Dimensione Cliente (LOGICA SCD Tipo 2: ricerca su chiave business E intervallo di validità)
INNER JOIN dbo.DimCustomer AS DC ON stg.CustNo = DC.CustomerAlternateKey
    AND stg.OrderDate >= DC.ValidFrom   -- La data dell'ordine deve essere DOPO o UGUALE alla data di inizio validità della dimensione
    AND stg.OrderDate < DC.ValidTo      -- E PRIMA della data di fine validità della dimensione

-- Lookup Dimensione Prodotto (LOGICA SCD Tipo 2)
INNER JOIN dbo.DimProduct AS DP ON stg.ProductID = DP.ProductAlternateKey
    AND stg.OrderDate >= DP.ValidFrom
    AND stg.OrderDate < DP.ValidTo

-- Lookup Dimensione Store (assumendo SCD Tipo 1 o 2 - stessa logica)
INNER JOIN dbo.DimStore AS DS ON stg.StoreID = DS.StoreAlternateKey
    AND stg.OrderDate >= DS.ValidFrom
    AND stg.OrderDate < DS.ValidTo;
```

**Nota sulla Performance**: Per carichi di lavoro su Big Data in Fabric Data Warehouse, è preferibile utilizzare operazioni in blocco come `COPY INTO` o `CTAS` e sfruttare la potenza del motore SQL per le `JOIN` e le ricerche di chiavi, anziché eseguire molti `INSERT` singoli o subquery come nell'esempio di staging.

---

# Usare le Pipeline di Dati (Data Factory) per Caricare un Warehouse

Il Data Warehouse di Microsoft Fabric fornisce strumenti integrati, sia **con codice** (es. T-SQL, Spark) che **senza codice/low-code** (es. Pipeline di Dati, Dataflow Gen2), per l'inserimento e il caricamento dei dati su larga scala.

## Data Pipeline: Orchestrazione e Integrazione

La **Pipeline di Dati** (un'esperienza basata su **Azure Data Factory**) è un servizio di integrazione dati basato sul cloud che permette di creare e orchestrare flussi di lavoro complessi per lo spostamento e la trasformazione dei dati.

- **Funzione**: Creare processi complessi **ETL** (Extract, Transform, Load) o **ELT** (Extract, Load, Transform), spesso eseguiti in base a una pianificazione.
- **Vantaggi**: Offre un'esperienza **senza codice/low-code** tramite un'interfaccia utente grafica (GUI), ideale per orchestratori di dati e flussi di lavoro che si connettono a sorgenti dati eterogenee.

### Creazione di una Pipeline

L'editor della Pipeline si può avviare:

1. **Dall'Area di Lavoro**: Selezionando `+ Nuovo` -> `Pipeline di dati`.
2. **Dall'Asset Warehouse**: Selezionando `Recupera dati` -> `Nuova pipeline di dati`.

|Opzione Iniziale|Descrizione|Intuizione|
|---|---|---|
|**1. Aggiungere attività della pipeline**|Avvia l'editor vuoto per creare una pipeline personalizzata.|Controllo completo e flessibilità per flussi di lavoro complessi (es. logica condizionale, cicli, esecuzione di script).|
|**2. Copiare dati**|Avvia un assistente passo-passo per configurare un'attività **Copia dati**.|Caricamento e spostamento di dati **veloce** e guidato, senza bisogno di scrivere codice.|
|**3. Scegliere un'attività da avviare**|Utilizzo di **modelli predefiniti** per scenari comuni.|Avvio facilitato per flussi di lavoro standardizzati.|

## L'Attività Chiave: Copia Dati (Copy Data)

L'assistente **Copia dati** (generato dall'opzione 2) guida l'utente attraverso la configurazione dell'attività di copia, gestendo:

1. **Origine e Connessione**: Selezione del connettore (es. database SQL, ADLS Gen2, API) e configurazione delle credenziali.
2. **Selezione Dati**: Scelta dei dati da copiare (tabelle, viste o query SQL personalizzate).
3. **Destinazione e Mapping**: Selezione dell'archivio di destinazione (il Warehouse di Fabric), e mapping delle colonne dall'origine alla destinazione (creando una nuova tabella o caricando in una esistente).
4. **Impostazioni**: Configurazione di dettagli aggiuntivi (es. impostazioni di **staging**, gestione dei valori predefiniti).

Dopo la copia, è possibile integrare altre attività per la trasformazione e l'analisi dei dati, o per pubblicare i risultati in un modello semantico di Power BI.

### Pianificazione

Una pipeline può essere eseguita automaticamente a intervalli regolari. La pianificazione si configura facilmente selezionando l'opzione **Pianifica** o **Impostazioni** (nel menu **Home**) all'interno dell'editor della pipeline.

# Caricare i Dati con T-SQL

Per gli sviluppatori SQL che preferiscono l'interfaccia T-SQL, il Warehouse di Fabric (basato sul motore SQL di Synapse) consente l'esecuzione di complesse query e manipolazioni dei dati direttamente a livello di database.

## 1. L'Istruzione `COPY INTO` (Bulk Loading)

L'istruzione **`COPY INTO`** è il metodo primario in T-SQL per l'importazione efficiente di dati in blocco (bulk) da un account di archiviazione **esterno** (come Azure Blob Storage o ADLS Gen2).

### Caratteristiche Principali

- **Efficienza**: Progettata per l'inserimento ad alta velocità di grandi volumi di dati.
- **Flessibilità di Formato**: Supporta diversi formati di file come **Parquet**, **CSV** e **JSON**.
- **Gestione degli Errori (`REJECTED_ROW_LOCATION`)**: Consente di specificare un percorso in cui salvare le righe non importate correttamente. Questo è cruciale per la pulizia dei dati e l'analisi dei problemi di qualità. _(Nota: `ERRORFILE`/`REJECTED_ROW_LOCATION` si applica principalmente a CSV)._
- **Caricamento Multiplo**: Permette di specificare **caratteri jolly** (`*`) o un elenco delimitato da virgole di percorsi per caricare più file contemporaneamente, purché abbiano la stessa struttura.

### Esempio di Sintassi per Caricamento

```SQL
COPY INTO my_table  -- Tabella di destinazione nel Warehouse
FROM 'https://myaccount.blob.core.windows.net/myblobcontainer/folder/*.parquet' -- Sorgente (percorso ADLS/Blob)
WITH (
    FILE_TYPE = 'PARQUET', -- Tipo di file (opzionale per Parquet/Delta)
    CREDENTIAL=(IDENTITY= 'Shared Access Signature', SECRET='<LaTuaSAS_Token>') -- Metodo di autenticazione alla sorgente esterna
    -- Altre opzioni: FIELDTERMINATOR, ROWTERMINATOR, FIRSTROW (per ignorare intestazioni), ecc.
);
```

**Importante**: `COPY INTO` è tipicamente usato per caricare file esterni che non sono già registrati in OneLake, sebbene l'approccio ELT nativo in Fabric spesso favorisca l'uso di `CTAS` e `INSERT...SELECT` per i dati già presenti in un Lakehouse.

---

## 2. Caricamento Dati tra Asset (Lakehouse e Warehouse)

Uno dei maggiori vantaggi di Microsoft Fabric è l'integrazione fluida tra Data Warehouse, Lakehouse e altri asset, tutti all'interno della stessa area di lavoro e che condividono lo stesso server SQL logico. Questo permette query e caricamenti **tra database**.

Per caricare dati tra asset, si usa la **denominazione in tre parti** di T-SQL: `[NomeWarehouse/Lakehouse].[Schema].[NomeTabella]`.

|Istruzione SQL|Uso Principale|Sintassi e Esempio|
|---|---|---|
|**`CREATE TABLE AS SELECT (CTAS)`**|**Creazione e Caricamento** di una _nuova_ tabella, spesso dopo un processo di trasformazione. Estremamente performante.|`CREATE TABLE [analysis_warehouse].[dbo].[nuova_tabella] AS SELECT * FROM [sales_warehouse].[dbo].[dati_vendite];`|
|**`INSERT...SELECT`**|**Aggiornamento o Inserimento** di dati in una tabella _esistente_. Utile per carichi incrementali.|`INSERT INTO [MyWarehouse].[dbo].[fatti] SELECT * FROM [SourceLakehouse].[dbo].[staging_table];`|

### Esempio di Integrazione tra Asset

È possibile combinare dati provenienti da un **Lakehouse** e un **Warehouse** per creare un nuovo set di dati analitico:
```SQL
-- Crea la tabella 'combined_data' nel 'analysis_warehouse'
-- unendo i dati da 'sales_warehouse' e 'social_lakehouse'.
CREATE TABLE [analysis_warehouse].[dbo].[combined_data]
AS
SELECT 
    sales.*, 
    social.num_reviews
FROM [sales_warehouse].[dbo].[sales_data] sales -- Dati strutturati
INNER JOIN [social_lakehouse].[dbo].[social_data] social -- Dati potenzialmente meno strutturati (esposti dal Lakehouse)
ON sales.[product_id] = social.[product_id];
```

**Intuizione**: Sfruttando la denominazione in tre parti, si elimina la necessità di spostare o copiare i dati tra i diversi asset all'interno di Fabric, operando direttamente sul singolo store logico (OneLake) tramite il motore SQL.

---

# Caricare e Trasformare i Dati con un Flusso di Dati Gen2

**Dataflow Gen2** è l'esperienza di **Power Query** di nuova generazione all'interno di Microsoft Fabric. Offre una soluzione di trasformazione dati low-code/senza codice, essenziale per gli scenari **ETL (Extract, Transform, Load)** o **ELT (Extract, Load, Transform)**.

## Concetti e Creazione

- **Piattaforma**: Sfrutta la potenza e l'interfaccia utente grafica di **Power Query**, semplificando l'intero processo di preparazione dei dati.
- **Utilizzo**: Può essere utilizzato per inserire dati in:
    - **Lakehouse** (come dati raw o trasformati).
    - **Warehouse** (tabelle finali).
    - Definire un **Set di Dati** per i report di Power BI.

Per creare un nuovo flusso di dati:

1. Vai all'Area di Lavoro.
2. Seleziona `+ Nuovo`.
3. Trova e seleziona **Dataflow Gen2** nella sezione `Data Factory`.

---

## Processo di Trasformazione

### 1. Importare Dati

Dopo l'avvio, Dataflow Gen2 offre un'ampia gamma di connettori per importare dati (file locali, database, servizi cloud, ecc.). Il processo di importazione è guidato e richiede pochi passaggi, indipendentemente dal tipo di file (es. CSV, testo).

### 2. Trasformazione con Power Query

Una volta importati, i dati risiedono come _query_ all'interno del flusso di dati. L'interfaccia di Power Query consente di eseguire trasformazioni complesse senza codice:

- Pulizia dei dati (es. gestione valori nulli).
- Rimodellazione (es. pivot, unpivot).
- Rimozione/Creazione di colonne.
- Unione (Merge) e Accodamento (Append) di query.

Tutti i passaggi eseguiti vengono registrati e sono modificabili, consentendo la riproducibilità del processo ETL.

### Intuizione: Trasformare i Dati con Copilot (AI)

**Copilot** in Dataflow Gen2 è un assistente basato su IA che può aiutare a eseguire trasformazioni complesse tramite comandi in linguaggio naturale.

- **Funzionamento**: Attivi Copilot, digiti un comando descrittivo (es. _"Trasformare la colonna Gender. Se maschio 0, se femmina 1. Quindi convertirlo in integer."_).
- **Vantaggio**: Copilot traduce la tua istruzione in passaggi di Power Query, aggiungendoli automaticamente alla trasformazione. Questo democratizza l'accesso a logiche complesse.

---

## 3. Aggiungere una Destinazione Dati (Data Destination)

La funzionalità **Aggiungi destinazione dati** è fondamentale e permette di separare la **logica ETL (la trasformazione in Power Query)** dallo **storage di destinazione**.

- **Destinazioni Supportate**: Warehouse (Fabric), Lakehouse (Fabric), Azure Synapse Analytics, Azure Data Explorer, Azure SQL Database.
- **Passaggio**: Nella scheda `Impostazioni query`, si aggiunge un passaggio di destinazione che specifica dove i dati finali devono essere scritti.

Quando si seleziona il **Warehouse** come destinazione, si deve scegliere il **Metodo di Aggiornamento**:

|Metodo di Aggiornamento|Descrizione|Scenario d'Uso|
|---|---|---|
|**Aggiungere (Append)**|Aggiunge nuove righe alla tabella esistente.|Carico Incrementale.|
|**Sostituire (Replace)**|Troncamento della tabella esistente e caricamento di tutti i dati del flusso di dati.|Carico Completo (Full Load) o quando non è richiesta la cronologia.|

### 4. Pubblicare e Eseguire

L'ultimo passaggio è **Pubblicare** il flusso di dati. La pubblicazione:

- Rende la logica di trasformazione (i passaggi di Power Query) e la configurazione di destinazione **effettive**.
- Incapsula l'intera operazione ETL in un'unica unità riutilizzabile.
- Abilita l'esecuzione manuale o automatica (tramite **Pipeline di Dati**).

---

# Riassunto: Considerazioni sul Caricamento Dati

Scegliere la strategia di caricamento corretta (Pipeline, T-SQL `COPY`, Dataflow Gen2) è cruciale e dipende da: la quantità dei dati, la frequenza di aggiornamento e le competenze del team.

| Categoria                      | Considerazione                                                                                        | Best Practice                                                                                                                         |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Volume e Frequenza**         | Determina se usare un Carico Completo o Incrementale.                                                 | Per carichi massivi e frequenti, preferire metodi bulk (`COPY INTO`, `CTAS`) e logiche incrementali (CDC).                            |
| **Competenze**                 | Chi sta scrivendo la logica di caricamento?                                                           | Sviluppatori SQL $\rightarrow$ T-SQL (`COPY`, `CTAS`). Data Engineer/Analyst low-code $\rightarrow$ Dataflow Gen2 o Pipeline di Dati. |
| **Governance & Storage**       | Tutti i dati sono gestiti in modo coerente.                                                           | Tutti i dati convergono in **OneLake** e sono governati per impostazione predefinita, garantendo sicurezza e accessibilità unificata. |
| **Mapping dei Dati**           | Garantire la coerenza nel passaggio dalla <br>sorgente $\rightarrow$ staging $\rightarrow$ warehouse. | Utilizzare Dataflow Gen2 per un mapping visuale o T-SQL `SELECT` per trasformazioni esplicite.                                        |
| **Dipendenze**                 | Le tabelle dei Fatti dipendono dalle Dimensioni.                                                      | **Caricare sempre le Dimensioni prima delle Tabelle dei Fatti** per garantire la presenza delle Chiavi Sostitutive (Surrogate Keys).  |
| **Progettazione degli Script** | Ottimizzazione della logica di inserimento.                                                           | Evitare `INSERT` singoli per grandi volumi; preferire `CTAS` o `INSERT...SELECT` per un caricamento in blocco.                        |

