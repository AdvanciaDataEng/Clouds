
# Esecuzione di Query Analitiche in Azure Data Warehouse con T-SQL

Quando si lavora con un Data Warehouse in ambienti come **Azure Synapse Analytics** o **Microsoft Fabric**, **Transact-SQL (T-SQL)** è il linguaggio primario per l'analisi dei dati. La sintassi è coerente con quella usata in SQL Server o Azure SQL Database, rendendo la transizione fluida per chi è già familiare con queste piattaforme.

## 1. Aggregazione di Misure tramite Dimensioni (Star/Snowflake Schema)

L'analisi in un Data Warehouse si concentra tipicamente sull'**aggregazione** di misure (valori numerici) dalle **tabelle dei fatti** in base agli **attributi** definiti nelle **tabelle delle dimensioni**. Questo approccio sfrutta la modellazione dimensionale (schema a stella o a fiocco di neve).

### Joins e Aggregazioni Fondamentali

Le query utilizzano clausole `JOIN` per collegare fatti e dimensioni, e le funzioni di aggregazione (`SUM`, `AVG`, `COUNT`, ecc.) insieme alla clausola `GROUP BY` per definire i livelli di aggregazione desiderati.

#### Esempio 1: Aggregazione per Gerarchia Temporale

La query seguente aggrega gli importi delle vendite per anno e trimestre, collegando la tabella dei fatti (`FactSales`) con la dimensione temporale (`DimDate`).
```SQL
SELECT
    dates.CalendarYear,
    dates.CalendarQuarter,
    SUM(sales.SalesAmount) AS TotalSales -- Aggregazione della misura
FROM dbo.FactSales AS sales
JOIN dbo.DimDate AS dates ON sales.OrderDateKey = dates.DateKey
GROUP BY dates.CalendarYear, dates.CalendarQuarter -- Gerarchia di aggregazione
ORDER BY dates.CalendarYear, dates.CalendarQuarter;
```

#### Esempio 2: Aggregazione su Più Dimensioni

È possibile includere più dimensioni nella query. Questo esempio estende l'aggregazione precedente suddividendo i totali delle vendite per città (`DimCustomer`).

```SQL
SELECT
    dates.CalendarYear,
    dates.CalendarQuarter,
    custs.City,
    SUM(sales.SalesAmount) AS TotalSales
FROM dbo.FactSales AS sales
JOIN dbo.DimDate AS dates ON sales.OrderDateKey = dates.DateKey
JOIN dbo.DimCustomer AS custs ON sales.CustomerKey = custs.CustomerKey
GROUP BY dates.CalendarYear, dates.CalendarQuarter, custs.City
ORDER BY dates.CalendarYear, dates.CalendarQuarter, custs.City;
```

### Join in uno Schema Snowflake

In uno **schema a fiocco di neve (Snowflake)**, le dimensioni sono parzialmente normalizzate, richiedendo **più join** per correlare il fatto alla dimensione più lontana.

**Intuizione:** Ogni join è necessario per navigare nella gerarchia dimensionale e permettere al motore di query di applicare i filtri o le aggregazioni richieste, anche se una colonna intermedia non viene visualizzata nel risultato finale.

#### Esempio: Navigazione tra Dimensioni Normalizzate

Per aggregare gli articoli venduti per categoria, la query deve attraversare la dimensione intermedia `DimProduct` per raggiungere la dimensione `DimCategory`.
```SQL
SELECT
    cat.ProductCategory,
    SUM(sales.OrderQuantity) AS ItemsSold
FROM dbo.FactSales AS sales
JOIN dbo.DimProduct AS prod ON sales.ProductKey = prod.ProductKey -- Join intermedio
JOIN dbo.DimCategory AS cat ON prod.CategoryKey = cat.CategoryKey -- Join alla dimensione finale
GROUP BY cat.ProductCategory
ORDER BY cat.ProductCategory;
```

## 2. Uso delle Funzioni di Classificazione (Ranking Functions)

Le funzioni di classificazione T-SQL sono cruciali per l'analisi che richiede la **prioritizzazione** o la **numerazione** delle righe all'interno di specifici gruppi di dati (**partizioni**).

Tutte queste funzioni utilizzano la clausola `OVER ([PARTITION BY colonna] ORDER BY colonna)`.

|Funzione|Descrizione|Gestione dei Pari (Ties)|Sequenza Tipica|
|---|---|---|---|
|**ROW_NUMBER()**|Assegna un numero sequenziale univoco a ciascuna riga all'interno della partizione.|Non ammette parità (assegna numeri diversi).|1, 2, 3, 4, 5...|
|**RANK()**|Assegna lo stesso rango ai valori pari, ma **salta i numeri** successivi.|I pari hanno lo stesso rango. Il rango successivo salta in base al numero di parità.|1, 2, 2, **4**, 5...|
|**DENSE_RANK()**|Assegna lo stesso rango ai valori pari, ma **non salta i numeri** successivi.|I pari hanno lo stesso rango. Il rango successivo è sempre incrementato di 1.|1, 2, 2, **3**, 4...|
|**NTILE(N)**|Divide la partizione in **N gruppi** (percentili/quartili) e assegna il numero del gruppo a ciascuna riga.|Assegna il gruppo (es. 1, 2, 3 o 4 per NTILE(4)).|1, 1, 2, 2, 3, 4...|

### Esempio Pratico e Confronto

L'esempio seguente confronta il comportamento delle funzioni di classificazione, partizionando i prodotti per categoria e classificandoli per prezzo di listino decrescente.
```SQL
SELECT
    ProductCategory,
    ProductName,
    ListPrice,
    ROW_NUMBER() OVER (PARTITION BY ProductCategory ORDER BY ListPrice DESC) AS RowNumber,
    RANK() OVER (PARTITION BY ProductCategory ORDER BY ListPrice DESC) AS Rank,
    DENSE_RANK() OVER (PARTITION BY ProductCategory ORDER BY ListPrice DESC) AS DenseRank,
    NTILE(4) OVER (PARTITION BY ProductCategory ORDER BY ListPrice DESC) AS Quartile
FROM dbo.DimProduct
ORDER BY ProductCategory, ListPrice DESC;
```

Intuizione sulla Differenza tra RANK e DENSE_RANK:

Immagina una gara:

- Se due atleti arrivano secondi (**RANK**), il prossimo che taglia il traguardo è il quarto.
- Se due atleti arrivano secondi (**DENSE_RANK**), il prossimo che taglia il traguardo è il terzo, perché i ranghi sono compattati, senza "buchi".

## 3. Conteggi Approssimativi per Big Data

Nell'esplorazione e nell'analisi di **set di dati di grandi dimensioni** (Big Data), un conteggio preciso dei valori distinti (`COUNT(DISTINCT)`) può essere molto dispendioso in termini di tempo e risorse (memoria). Spesso, un conteggio stimato è sufficiente per l'analisi preliminare.

### La Funzione APPROX_COUNT_DISTINCT

La funzione aggregata **`APPROX_COUNT_DISTINCT(espressione)`** calcola il numero **approssimativo** di valori univoci non null in un gruppo.

- **Vantaggio Chiave:** Velocità di risposta notevolmente superiore e un footprint di memoria ridotto rispetto a `COUNT(DISTINCT)`, riducendo la probabilità di _spill_ su disco.
- **Precisione:** L'implementazione garantisce un tasso di errore massimo del **2%** con una probabilità del **97%**.

### Algoritmo HyperLogLog

La funzione `APPROX_COUNT_DISTINCT` si basa sull'algoritmo **HyperLogLog (HLL)**, un algoritmo all'avanguardia per la stima della cardinalità (il numero di elementi distinti) nei set di dati molto grandi, utilizzando una quantità di memoria molto piccola (![](data:,)) rispetto alla dimensione del dataset.

#### Esempio di Utilizzo

L'esempio seguente utilizza `APPROX_COUNT_DISTINCT` per stimare il numero di ordini distinti per anno, offrendo un compromesso accettabile tra precisione e velocità di esecuzione.
```SQL
SELECT
    dates.CalendarYear AS CalendarYear,
    APPROX_COUNT_DISTINCT(sales.OrderNumber) AS ApproxOrders
FROM FactSales AS sales
JOIN DimDate AS dates ON sales.OrderDateKey = dates.DateKey
GROUP BY dates.CalendarYear
ORDER BY CalendarYear;
```

La funzione `APPROX_COUNT_DISTINCT` è ottimizzata per scenari Big Data con milioni di righe e colonne con molti valori distinti, dove la velocità è prioritaria rispetto alla precisione assoluta.

Per un approfondimento visivo sulle differenze tra le funzioni di classificazione in SQL, puoi guardare questo video: Difference between rank dense rank and row number in SQL.

---

# Strumenti di Querying e Interazione con Azure/Fabric Data Warehouse

Quando si opera in ambienti di Data Warehousing moderni come **Microsoft Fabric** (che include il servizio Warehouse) o **Azure Synapse Analytics**, l'interazione con i dati può avvenire tramite interfacce integrate e strumenti client esterni.

## 1. Editor di Query SQL (T-SQL) Integrato

L'Editor di Query SQL in Microsoft Fabric è l'interfaccia principale e integrata per l'esecuzione di **Transact-SQL (T-SQL)**. È uno strumento di sviluppo completo, non solo per l'estrazione di dati (`SELECT`), ma anche per la **Data Definition Language (DDL)** e la **Data Manipulation Language (DML)**.

### Funzionalità e Avvio Rapido

|Caratteristica|Dettaglio|Intuizione|
|---|---|---|
|**Sintassi**|Supporta T-SQL completo (creazione tabelle, viste, inserimento/modifica dati, gestione autorizzazioni, `SELECT` avanzate).|Permette di eseguire modifiche strutturali e logiche dirette al Data Warehouse.|
|**Ambiente**|Offre **IntelliSense** per il completamento automatico e supporto per il **debug** degli script.|Velocizza lo sviluppo e riduce gli errori di sintassi.|
|**Accesso**|Si connette automaticamente al Warehouse selezionato dalla tua **Area di Lavoro** (Workspace), senza credenziali aggiuntive.|Semplifica l'accesso: non devi preoccuparti delle stringhe di connessione.|
|**Salvataggio**|Qualsiasi script T-SQL che crei viene salvato automaticamente in una **Nuova Query SQL** all'interno della cartella `Query personali` in **Explorer**.|Mantiene traccia del tuo lavoro, creando un elemento di query riutilizzabile e condivisibile.|

### Esecuzione e Gestione dei Risultati

1. **Esecuzione:** Digita o incolla lo script e seleziona **Esegui**.
2. **Visualizzazione:** La sezione `Risultati` mostra un'anteprima, limitata a **10.000 righe** per non sovraccaricare il browser.
3. **Esportazione:** Puoi esportare i risultati di una query `SELECT` selezionata (**evidenziata**) come **file Excel** tramite il pulsante **Scarica file Excel**.

### Persistenza delle Query

L'editor offre opzioni rapide per salvare il set di risultati di una query `SELECT` complessa, riutilizzando il codice T-SQL evidenziato:

- **Salva come Vista:** Crea una **Vista** logica nel Data Warehouse. Una vista è una query salvata che appare come una tabella, ma non memorizza dati fisici (è sempre aggiornata).
- **Salva come Tabella:** Esegue un'operazione **CTAS** (`CREATE TABLE AS SELECT`), creando una nuova tabella fisica con i dati risultanti dalla query. Utile per snapshot o tabelle aggregate (mart).

## 2. Editor di Query Visivo (No-Code/Low-Code)

L'Editor di Query Visivo fornisce un'alternativa all'uso di T-SQL, basata su un'interfaccia **drag-and-drop** e grafica. È ideale per analisti o membri del team che non hanno familiarità con la sintassi SQL complessa.

### Vantaggi e Funzionamento

- **Interfaccia Grafica Intuitiva:** Permette di trascinare tabelle e definire visivamente le relazioni (`JOIN`) e le condizioni di filtro.
- **Generazione Automatica di T-SQL:** Mentre l'utente progetta la query in modo visivo, lo strumento **genera automaticamente lo script SQL** corrispondente in background.
- **Accessibilità dei Dati:** Democrazia l'accesso ai dati, consentendo agli utenti non tecnici di estrarre e manipolare informazioni preziose per il _decision-making_ rapido.
- **Persistenza:** Anche le query visive possono essere salvate rapidamente come **Tabelle** o **Viste** logiche, analogamente all'editor SQL.

**Intuizione:** Pensa all'Editor Visivo come a un **traduttore istantaneo**. Tu disegni il tuo schema mentale di query, e lui lo converte immediatamente nel linguaggio del Data Warehouse (T-SQL).

## 3. Connessione tramite Strumenti Client Esterni (SSMS e Altri)

Per gli utenti che preferiscono strumenti desktop con funzionalità avanzate di gestione del database, Microsoft Fabric supporta la connessione tramite client esterni, come **SQL Server Management Studio (SSMS)**.

### Procedura di Connessione (SSMS)

Il Data Warehouse in Fabric è esposto tramite un **Endpoint di Analisi SQL**, accessibile tramite una stringa di connessione:

1. **Copia Stringa di Connessione:** In Microsoft Fabric, nell'asset del Warehouse, trova e seleziona **Copia stringa di connessione SQL**.
2. **Connessione in SSMS:** Avvia SSMS. Incolla la stringa copiata nella casella **Nome server**.
3. **Autenticazione:** Specifica le tue credenziali e connettiti.

Una volta connesso, SSMS elencherà il Warehouse, le sue tabelle e le viste, consentendoti di eseguire query e gestire gli oggetti come faresti con un database SQL Server tradizionale.

### Modelli di Autenticazione Supportati

È fondamentale notare che l'autenticazione è strettamente vincolata all'ecosistema Azure/Fabric:

- ✅ **Entità Utente Microsoft Entra ID** (precedentemente Azure Active Directory)
- ✅ **Entità Servizio Microsoft Entra ID**

> ❌ **Autenticazione SQL (Nome utente/Password SQL) NON è supportata.** Devi sempre autenticarti tramite Microsoft Entra ID.

### Strumenti di Terze Parti

Qualsiasi strumento di terze parti o applicazione personalizzata può connettersi al Warehouse di Microsoft Fabric utilizzando la stessa **stringa di connessione SQL**, purché sia in grado di utilizzare:

- **Driver ODBC** o **OLE DB**.
- **Autenticazione Microsoft Entra ID** (obbligatoria).

**Punti Chiave:** L'uso di strumenti esterni offre una maggiore flessibilità e familiarità, ma richiede l'uso esclusivo dell'autenticazione basata su **Microsoft Entra ID** per garantire la sicurezza e la conformità nell'ambiente cloud.