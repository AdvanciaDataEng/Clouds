
Le tabelle in un **Microsoft Fabric Lakehouse** si basano sul formato di archiviazione **Delta Lake** (open source), il che trasforma il data lake in un **Lakehouse** fornendo l'affidabilità e la struttura tipiche di un data warehouse direttamente sull'archiviazione del data lake (OneLake).

## Fondamenti di Delta Lake

**Delta Lake** è un livello di archiviazione open source che opera sopra i file in formato **Parquet** e aggiunge un **log delle transazioni** per portare l'affidabilità e la semantica dei database relazionali all'elaborazione di big data basata su Apache Spark.

### Componenti Strutturali di una Tabella Delta

Una tabella Delta non è un singolo file, ma una **cartella** che contiene due elementi chiave nell'archivio cloud (in Fabric, OneLake):

1. **File di Dati Parquet:** Sono i file sottostanti che contengono i dati effettivi della tabella. Parquet è un formato colonnare ottimizzato per l'analisi e l'efficienza delle query.
2. **Cartella `_delta_log`:** È il cuore transazionale di Delta Lake. Contiene un log ordinato e _append-only_ (solo aggiunta) di tutte le transazioni che hanno modificato la tabella.

### Vantaggi Chiave di Delta Lake

|Caratteristica|Descrizione|
|---|---|
|**Transazioni ACID**|Garantisce l'affidabilità dei dati in ambienti di lettura/scrittura concorrente (batch e streaming).|
|**Schema Enforcement & Evolution**|Impedisce l'inserimento di dati "cattivi" che non corrispondono allo schema della tabella (enforcement) e permette di gestire i cambiamenti di schema in modo controllato (evolution).|
|**Time Travel (Versione dei Dati)**|Consente di accedere e interrogare versioni storiche dei dati basate sulla versione (numero di commit) o sul timestamp.|
|**Unified Batch/Streaming**|Una tabella Delta può fungere contemporaneamente da _sink_ (destinazione) e _source_ (origine) per i dati batch e di streaming, semplificando le architetture dati.|
|**DML Operations**|Abilita operazioni di manipolazione dei dati come `UPDATE`, `DELETE`, e `MERGE` (upsert) in Spark, cosa non possibile con i semplici file Parquet.|

---

## Le Transazioni ACID in Dettaglio

L'acronimo **ACID** è fondamentale per l'affidabilità e la coerenza dei dati. Delta Lake estende questi principi ai data lake:

|Principio|Spiegazione|Implementazione Delta Lake (Tramite `_delta_log`)|
|---|---|---|
|**Atomicità**|Una transazione è considerata un'unica unità di lavoro: o tutte le operazioni riescono, o nessuna viene applicata.|Il log registra l'operazione solo _dopo_ che tutti i file di dati sono stati scritti correttamente. Se la scrittura fallisce, la transazione non viene registrata e i file scritti non corrompono la tabella (sono considerati _uncommitted_).|
|**Coerenza**|Garantisce che una transazione porti il database da uno stato valido a un altro stato valido (rispetto a vincoli, regole di schema, ecc.).|L'**Enforcement dello Schema** e l'applicazione riuscita di tutte le operazioni di commit (Add/Remove file) sul log garantiscono uno stato coerente.|
|**Isolamento**|Le transazioni concorrenti non devono interferire tra loro. Il risultato dell'esecuzione simultanea è lo stesso che si otterrebbe eseguendo le transazioni in serie.|Utilizza il **Controllo di Concorrenza Ottimistico** per gestire scritture multiple e offre un isolamento _Snapshot_ per le letture (i lettori vedono la versione della tabella al momento dell'inizio della query).|
|**Durabilità**|Una volta che una transazione è stata confermata (`committed`), i suoi cambiamenti sono permanenti e sopravvivono a eventuali guasti del sistema.|La transazione è registrata in modo persistente (durabile) nel log (`_delta_log`) su OneLake (archiviazione cloud persistente).|

---

## Time Travel (Versione dei Dati)

La funzionalità di **Time Travel** (Viaggio nel Tempo) è una diretta conseguenza del log delle transazioni di Delta Lake. Il log traccia ogni singola operazione (aggiunta o rimozione di file) con un numero di **versione** incrementale e un **timestamp**.

Questo permette di:

- **Audit:** Avere una cronologia completa e verificabile di ogni cambiamento.
- **Rollback/Correzione Errori:** Ripristinare i dati a una versione precedente dopo un errore umano o di pipeline.
- **Riproducibilità:** Eseguire report o modelli ML su uno _snapshot_ esatto dei dati in un momento preciso.

### Sintassi di Esempio (PySpark)

È possibile interrogare le versioni passate della tabella specificando la versione o il timestamp:

|Tipo di Query|Sintassi (PySpark)|
|---|---|
|**Per Versione**|`spark.read.format("delta").option("versionAsOf", 5).load("path/to/delta/table")`|
|**Per Timestamp**|`spark.read.format("delta").option("timestampAsOf", "2024-01-01").load("path/to/delta/table")`|
|**Cronologia**|`spark.sql("DESCRIBE HISTORY mytable").show()`|

---

## Creazione e Tipologie di Tabelle Delta in Fabric

In Microsoft Fabric Lakehouse, quando si lavora con Spark, si ha il controllo sulla creazione e gestione delle tabelle Delta. Esistono due tipi principali di tabelle che definiscono la relazione tra i metadati nel metastore e i file di dati in OneLake:

### 1. Tabelle Gestite (Managed Tables)

- **Definizione:** Spark (e Fabric) gestisce **sia i metadati** (schema, nome, ecc.) **sia i dati sottostanti** (file Parquet e log).
- **Posizione Dati:** I dati vengono salvati nella sezione `/tables` del Lakehouse (l'area di archiviazione predefinita per le tabelle).
- **Ciclo di Vita:** Se la tabella viene eliminata (`DROP TABLE`), sia la definizione di tabella nel metastore che i file di dati sottostanti vengono **eliminati**.
- **Uso:** Ideali per l'uso generale in Fabric, dove si desidera la massima semplicità e il ciclo di vita del dato è legato alla definizione della tabella.

**Esempio di Creazione (PySpark):**
```Python
# Crea e salva come tabella gestita nel percorso predefinito /tables
df.write.format("delta").saveAsTable("mytable")
```

### 2. Tabelle Esterne (External Tables)

- **Definizione:** Spark gestisce solo i **metadati** della tabella, mentre i file di dati sottostanti sono archiviati in una **posizione specificata dall'utente**.
- **Posizione Dati:** I dati vengono salvati tipicamente nella sezione `/files` del Lakehouse o in un percorso completo di OneLake.
- **Ciclo di Vita:** Se la tabella viene eliminata (`DROP TABLE`), viene eliminata solo la definizione di tabella nel metastore. I file di dati sottostanti **rimangono intatti** nella loro posizione.
- **Uso:** Utili per l'integrazione con dati preesistenti, per data governance, per condividere i dati tra più Lakehouse (tramite Shortcut di OneLake) o per scenari in cui il ciclo di vita dei dati deve essere disaccoppiato dal metastore.

**Esempio di Creazione (PySpark):**
```Python
# Crea la definizione di tabella, ma i dati vengono salvati nel percorso specificato in /files
df.write.format("delta").saveAsTable("myexternaltable", path="Files/myexternaltable_data")
```

---

## Metodi per la Creazione dello Schema di Tabelle Delta

È possibile creare tabelle Delta in Fabric in diversi modi, a seconda che si parta da un DataFrame esistente o che si voglia definire prima lo schema.

### A. Creazione da un DataFrame Esistente (Approccio Data-First)

Questo è il modo più comune, dove lo schema della tabella Delta viene **dedotto** direttamente dal DataFrame di Spark.

**Sintassi PySpark - `saveAsTable()`:**
```Python
# 1. Carica i dati in un DataFrame
df = spark.read.load('Files/mydata.csv', format='csv', header=True)

# 2. Salva il DataFrame come tabella Delta
# Questo crea anche i file di dati Parquet e il _delta_log
df.write.format("delta").saveAsTable("mytable_df")
```

### B. Creazione con Definizione Esplicita dello Schema (Approccio Schema-First)

Questo approccio definisce lo schema _prima_ che i dati vengano popolati, garantendo una forte **Schema Enforcement** fin dall'inizio.

#### 1. Usando Spark SQL (`CREATE TABLE`)

Permette di creare tabelle Delta usando la sintassi SQL standard.

**Esempio: Tabella Gestita**
```SQL
%%sql

CREATE TABLE salesorders
(
    Orderid INT NOT NULL,
    OrderDate TIMESTAMP NOT NULL,
    CustomerName STRING,
    SalesTotal FLOAT NOT NULL
)
USING DELTA
```

**Esempio: Tabella Esterna (basata su percorso)**
```SQL
%%sql

CREATE TABLE MyExternalTable_SQL
USING DELTA
LOCATION 'Files/mydata_location'
-- NOTA: Se il percorso 'Files/mydata_location' contiene già file Delta, lo schema
-- della tabella esterna sarà *dedotto* da quei file.
```

#### 2. Usando l'API `DeltaTableBuilder` (Programmatico)

L'API `DeltaTableBuilder` in Python/Scala offre un controllo programmatico e granulare sulla definizione dello schema, incluse colonne speciali o proprietà di tabella.

**Esempio PySpark:**
```Python
from delta.tables import *

# Inizializza il builder
DeltaTable.create(spark) \
  .tableName("products_api") \
  .addColumn("Productid", "INT", nullable=False) \
  .addColumn("ProductName", "STRING") \
  .addColumn("Price", "FLOAT") \
  .comment("Tabella prodotti creata con DeltaTableBuilder") \
  .execute()
# NOTA: L'uso di .execute() crea la definizione della tabella Delta (schemaless), ma non popola i dati.
```

---

## Manipolazione e Salvataggio dei Dati (Append/Overwrite)

Oltre alla creazione iniziale, Delta Lake supporta la scrittura incrementale e la sostituzione di dati in un percorso Delta. Questo può essere fatto **senza** creare una definizione di tabella nel metastore, ma semplicemente scrivendo i dati in un percorso **delta**.

### Scrittura in un Percorso Delta (File-Based Write)

È possibile salvare un DataFrame in un percorso specifico come formato Delta. 
Questo crea i file Parquet e la cartella `_delta_log` in quel percorso.
```Python
delta_path = "Files/mydatatable"

# Scrittura iniziale
df.write.format("delta").save(delta_path)
```

### Modalità di Scrittura

Delta Lake supporta diverse modalità di scrittura tramite l'opzione `.mode()`:

|Modalità|Descrizione|Sintassi PySpark|
|---|---|---|
|**`append`**|Aggiunge nuove righe di dati al set di dati esistente nel percorso Delta.|`new_rows_df.write.format("delta").mode("append").save(delta_path)`|
|**`overwrite`**|Sostituisce _tutti_ i dati esistenti nel percorso Delta con i nuovi dati dal DataFrame. **Attenzione:** Questa operazione è atomica.|`new_df.write.format("delta").mode("overwrite").save(delta_path)`|
|**`errorifexists`** (o `error`)|Lancia un'eccezione se il percorso Delta esiste già.|`df.write.format("delta").mode("errorifexists").save(delta_path)`|
|**`ignore`**|Non scrive nulla se il percorso Delta esiste già.|`df.write.format("delta").mode("ignore").save(delta_path)`|

> **Intuizione Fabric:** Se si salva un DataFrame in un percorso sotto l'area **`Tables`** del Lakehouse in Fabric, il sistema di **individuazione automatica
> delle tabelle** di Fabric rileverà la struttura Delta e creerà automaticamente i metadati della tabella corrispondente nel metastore (come se fosse stata usata `saveAsTable`). Questo assicura che il dato sia accessibile tramite l'endpoint SQL e altre esperienze Fabric (come Power BI Direct Lake).
---

# Ottimizzare le tabelle delta

Spark è un framework per l'elaborazione parallela, con i dati archiviati in uno o più nodi di lavoro. Inoltre, i file Parquet non sono modificabili e vengono scritti nuovi file per ogni aggiornamento o eliminazione. Questo processo può comportare l'archiviazione dei dati da parte di Spark in un numero elevato di file di piccole dimensioni, ovvero dar luogo a un _problema di file di piccole dimensioni._ Ciò significa che è possibile che le query su grandi quantità di dati potrebbero essere eseguite lentamente o addirittura non essere completate.

## Funzione OptimizeWrite

_OptimizeWrite_ è una funzionalità di Delta Lake che riduce il numero di file durante la scrittura. Invece di scrivere molti file di piccole dimensioni, scrive meno file di grandi dimensioni. In questo modo si evita il problema dei _file di piccole dimensioni_ e si garantisce che le prestazioni non vengano compromesse.

In Microsoft Fabric, `OptimizeWrite` è abilitato per impostazione predefinita. 
È possibile abilitarlo o disabilitarlo a livello di sessione Spark:
```python
# Disable Optimize Write at the Spark session level
spark.conf.set("spark.microsoft.delta.optimizeWrite.enabled", False)

# Enable Optimize Write at the Spark session level
spark.conf.set("spark.microsoft.delta.optimizeWrite.enabled", True)

print(spark.conf.get("spark.microsoft.delta.optimizeWrite.enabled"))
```

---

# Manutenzione e Ottimizzazione Avanzata delle Tabelle Delta

Le tabelle Delta Lake in Microsoft Fabric richiedono operazioni di manutenzione periodica per mantenere l'efficienza delle query e ottimizzare l'utilizzo dello spazio di archiviazione. Le due operazioni principali sono l'**ottimizzazione** (per le prestazioni di lettura) e la **pulizia** (per lo spazio di archiviazione).

## Ottimizzazione (OPTIMIZE)

L'operazione `OPTIMIZE` è una funzione cruciale di manutenzione delle tabelle che migliora le prestazioni di lettura consolidando i file Parquet di piccole dimensioni in un numero inferiore di file più grandi.

### Il Problema dei "File di Piccole Dimensioni" (Small File Problem)

Le scritture incrementali (ad esempio, flussi di dati o frequenti `INSERT`/`UPDATE`) spesso creano molti file Parquet di dimensioni ridotte. L'accesso a molti piccoli file impone un overhead significativo sul motore di calcolo (Spark, SQL, ecc.) perché:

- **Overhead di Metadati:** Il motore deve aprire, leggere i metadati e chiudere un numero elevato di file.
- **Inefficienza di I/O:** Le operazioni di I/O su molti file piccoli sono meno efficienti delle operazioni I/O sequenziali su pochi file grandi.

### Effetti di `OPTIMIZE`

Eseguire `OPTIMIZE` su una tabella di grandi dimensioni risulta in:

- **Consolidamento:** Meno file, ma di grandi dimensioni.
- **Migliore Compressione:** I file più grandi spesso permettono una migliore compressione a livello di blocco.
- **Distribuzione Efficiente:** Migliore allocazione del lavoro tra i nodi del cluster Spark.

### V-Order (Vertical Order)

Quando si esegue `OPTIMIZE`, l'opzione **V-Order** (_Vertical Order_) è fondamentale per le prestazioni in Microsoft Fabric.

|Caratteristica|Dettaglio|
|---|---|
|**Scopo**|Ottimizzare la struttura fisica dei file Parquet per massimizzare la velocità di lettura con i motori Fabric (Power BI, SQL) e Spark.|
|**Funzionamento**|Applica un ordinamento speciale, una distribuzione dei gruppi di righe, la codifica del dizionario e la compressione avanzata all'interno del formato Parquet.|
|**Compatibilità**|È un'ottimizzazione 100% conforme a Parquet; tutti i motori Parquet open source possono leggere i file V-Ordered.|
|**Impatto**|**Letture:** Molto più veloci. I motori Power BI e SQL di Fabric utilizzano la tecnologia **Microsoft Verti-Scan** per sfruttare appieno V-Order, ottenendo prestazioni quasi in memoria. Spark trae comunque un significativo vantaggio in velocità (10-50%). **Scritture:** Causa un piccolo overhead (circa 15%) in fase di scrittura.|

#### Quando Disabilitare V-Order?

V-Order è **abilitato di default** in Fabric. Potrebbe non essere utile per gli **scenari ad alta intensità di scrittura** come gli _staging data stores_ dove i dati vengono letti solo una o due volte prima di essere spostati/trasformati.

### Sintassi e Interfaccia Utente

È possibile eseguire `OPTIMIZE` tramite l'interfaccia utente di **Lakehouse Explorer** (Manutenzione) o tramite codice Spark SQL:

**Esempio Spark SQL:**
```Python
%%sql

-- Esegue l'ottimizzazione e applica V-Order
OPTIMIZE products; 

-- Esegue l'ottimizzazione solo su una specifica partizione e applica V-Order (se abilitato di default)
OPTIMIZE salesorders WHERE year = 2024;
```

---

## Pulizia (VACUUM) e Conservazione dei Dati

Il comando `VACUUM` è un'operazione di pulizia che elimina i file di dati _non più referenziati_ dal log delle transazioni di Delta Lake. È essenziale per gestire i costi di archiviazione.

### Funzionamento e Rischio

Ogni operazione di `UPDATE` o `DELETE` su una tabella Delta crea nuovi file Parquet e contrassegna i vecchi file come obsoleti nel log delle transazioni. Questi vecchi file vengono mantenuti per un certo periodo per consentire il **Time Travel**.

- **VACUUM non elimina i log delle transazioni:** Rimuove solo i file **Parquet obsoleti**.
- **Rischio Time Travel:** Dopo aver eseguito `VACUUM`, non è possibile accedere a versioni della tabella anteriori al periodo di conservazione specificato. L'eliminazione dei file è **permanente**.

### Periodo di Conservazione (Retention Period)

Il parametro `RETAIN <N> HOURS` determina per quanto tempo i vecchi file devono essere mantenuti.

- **Valore Predefinito:** **7 giorni** (168 ore).
- **Minimo di Sicurezza:** Il sistema di solito impedisce di usare un periodo di conservazione inferiore a 7 giorni per evitare la corruzione dei dati in caso di esecuzioni di query lunghe o _stream_ che potrebbero fare riferimento a file che `VACUUM` ha rimosso.

### Quando Eseguire `VACUUM`?

- Quando si hanno grandi volumi di dati e frequenti operazioni di `UPDATE`/`DELETE` che generano molti file obsoleti.
- Quando i costi di archiviazione stanno diventando elevati a causa dell'accumulo di file obsoleti.

### Sintassi

È possibile eseguire `VACUUM` tramite l'interfaccia utente di **Lakehouse Explorer** (Manutenzione) o tramite codice:

**Esempio Spark SQL:**
```Python
%%sql

-- Rimuove i file obsoleti più vecchi di 168 ore (7 giorni) dalla tabella 'products'
VACUUM products RETAIN 168 HOURS;

-- Per le tabelle esterne o i percorsi delta:
VACUUM 'Files/mytable' RETAIN 24 HOURS; -- Attenzione: usare RETAIN inferiore a 168 ore è sconsigliato e richiede una configurazione aggiuntiva di Spark.
```

---

## Partizionamento delle Tabelle Delta

Il partizionamento è una tecnica di organizzazione fisica dei dati che divide logicamente i dati in sottocartelle basate sui valori di una o più colonne.

### Funzionamento e Benefici

- **Layout Fisico:** La tabella viene suddivisa in cartelle sull'archiviazione: ad esempio, partizionando per `Category` si avranno cartelle `Category=Electronics`, `Category=Books`, ecc.
- **Data Skipping (Salto Dati):** Il beneficio principale. Quando si interroga con un filtro sulla colonna di partizione (es. `WHERE Category = 'Books'`), il motore di calcolo ignora completamente la lettura delle cartelle che non corrispondono al filtro.

### Guida all'Uso del Partizionamento

|Criterio|Descrizione|
|---|---|
|**Grandi Volumi**|Utilizzare il partizionamento solo con grandi volumi di dati (milioni/miliardi di righe).|
|**Bassa Cardinalità**|Scegliere colonne con **bassa cardinalità** (un numero limitato di valori univoci, es. `year`, `country`, `month`).|
|**Evitare il "Small File Problem"**|**NON** partizionare su colonne con alta cardinalità (es. `CustomerID`, `timestamp` completo) perché si creerebbero troppe partizioni, ognuna contenente file minuscoli, annullando i benefici di `OPTIMIZE`.|
|**Layout Fisso**|Le partizioni sono un layout fisico **fisso**. Non si adattano a modelli di query dinamici.|

> **Intuizione:** Delta Lake include il **Z-Ordering** come alternativa più flessibile al partizionamento tradizionale per i filtri ad alta cardinalità. Z-Ordering co-localizza i valori correlati all'interno dello stesso file, riducendo anche in quel caso il _data skipping_.

### Sintassi di Creazione

**Esempio PySpark (Scrittura con Partizionamento):**
```Python
# Partiziona la tabella "partitioned_products" per la colonna "Category"
df.write.format("delta").partitionBy("Category").saveAsTable("partitioned_products", mode="overwrite")
```

**Esempio Spark SQL (Definizione di Tabella Partizionata):**
```SQL
%%sql
CREATE TABLE partitioned_products_sql (
    ProductID INTEGER,
    ProductName STRING,
    ListPrice DOUBLE,
    Category STRING
)
USING DELTA
PARTITIONED BY (Category) 
LOCATION 'Files/partitioned_products_sql'; -- Si noti che le tabelle partizionate sono spesso esterne
```

---

## Uso Avanzato delle Tabelle Delta in Spark

Le tabelle Delta offrono due modi principali per interagire con i dati in Apache Spark: il familiare **Spark SQL** e la potente **API Delta Lake**.

### 1. Manipolazione Dati Tramite Spark SQL

È il modo più immediato per eseguire operazioni di DML (Data Manipulation Language) su una tabella Delta.

|Operazione|Descrizione|Esempio PySpark + SQL|
|---|---|---|
|**`INSERT`**|Aggiunge nuove righe alla tabella.|`spark.sql("INSERT INTO products VALUES (1, 'Widget', 'Accessories', 2.99)")`|
|**`UPDATE`**|Modifica i valori delle righe esistenti.|`spark.sql("UPDATE products SET ListPrice = 2.49 WHERE ProductId = 1")`|
|**`DELETE`**|Rimuove le righe che soddisfano una condizione.|`spark.sql("DELETE FROM products WHERE ListPrice > 100")`|

### 2. Uso Diretto dell'API Delta Lake

L'API Delta Lake è preferibile quando si lavora direttamente con i **percorsi dei file Delta** (ad esempio, tabelle esterne o percorsi non registrati nel metastore) o quando si vogliono eseguire operazioni complesse come il **`MERGE`** (Upsert).

#### Creazione dell'Oggetto `DeltaTable`

Per usare l'API, si crea un oggetto `DeltaTable` che rappresenta la tabella o il percorso Delta:
```Python
from delta.tables import *
from pyspark.sql.functions import *

# Associa l'oggetto DeltaTable a un percorso file (piuttosto che a un nome di catalogo)
delta_path = "Files/mytable"
deltaTable = DeltaTable.forPath(spark, delta_path) 
```

#### Esempio di `UPDATE` Tramite API

L'API consente di eseguire DML in modo programmatico e robusto:
```Python
# Aggiorna la tabella (riduce il prezzo degli accessori del 10%)
deltaTable.update(
    condition = "Category == 'Accessories'",
    set = { "Price": "Price * 0.9" }
)
```

> **Intuizione:** L'operazione `MERGE INTO`  è la caratteristica più potente dell'API Delta. 
> Permette di combinare `INSERT`, `UPDATE` e `DELETE` in un'unica operazione atomica, rendendo facile l'implementazione di pattern ETL come **SCD Type 2** (Slowly Changing Dimensions) o **Upsert** (Insert-or-Update).

---

## Query Storiche (Time Travel)

Le operazioni di DML/Manutenzione (come `UPDATE`, `OPTIMIZE`, `VACUUM`) sono registrate come nuove versioni nel log delle transazioni.

### Visualizzare la Cronologia

Il comando `DESCRIBE HISTORY` rivela il log delle transazioni, mostrando ogni **versione** e l'**operazione** che l'ha generata.
```Python
%%sql

DESCRIBE HISTORY products;
-- Oppure per un percorso:
DESCRIBE HISTORY 'Files/mytable'
```

|Colonna|Ruolo|
|---|---|
|**`version`**|L'identificativo unico e incrementale della versione (commit).|
|**`timestamp`**|L'ora UTC in cui la transazione è stata completata.|
|**`operation`**|Il tipo di operazione (es. `UPDATE`, `WRITE`, `CREATE TABLE`, `OPTIMIZE`, `DELETE`).|
|**`operationParameters`**|Dettagli aggiuntivi sull'operazione (es. il predicato WHERE dell'UPDATE).|

### Recuperare Dati Storici

Il Time Travel si ottiene leggendo i dati, specificando la versione o il timestamp desiderato, **senza cambiare la versione corrente** della tabella:

|Metodo|Opzione di Lettura|Esempio PySpark|
|---|---|---|
|**Per Versione**|`versionAsOf`|`df = spark.read.format("delta").option("versionAsOf", 2).load(delta_path)`|
|**Per Timestamp**|`timestampAsOf`|`df = spark.read.format("delta").option("timestampAsOf", '2023-04-04 21:00:00').load(delta_path)`|

> **Nota:** Mentre Time Travel permette di _leggere_ dati storici, per _ripristinare_ una versione precedente della tabella in modo permanente, è necessario usare il comando **`RESTORE`**.

# Usare le Tabelle Delta con Dati di Streaming

La capacità di unificare l'elaborazione di dati **batch** (statici) e **streaming** (in tempo reale) è uno dei principali punti di forza di Delta Lake in Microsoft Fabric. Questa unificazione è resa possibile da **Spark Structured Streaming**.

## Spark Structured Streaming: Un'Introduzione

**Spark Structured Streaming** è l'API nativa di Apache Spark per l'elaborazione dei flussi di dati. Tratta un flusso di dati illimitato (infinito) come un **DataFrame illimitato**.

### Architettura del Flusso

Una soluzione di elaborazione dei flussi tipica in Structured Streaming segue questo modello:

1. **Origine (Source):** Lettura costante di un flusso di dati. Esempi includono porte di rete, servizi di messaggistica in tempo reale (come **Hub eventi di Azure** o **Kafka**) o percorsi del file system.
2. **Elaborazione/Trasformazione:** Applicazione di operazioni standard sui dati in arrivo (filtri, aggregazioni, raggruppamenti, aggiunta di colonne).
3. **Sink (Destinazione):** Scrittura continua dei risultati del flusso.

> **Intuizione:** Structured Streaming utilizza lo stesso motore di esecuzione ottimizzato e le stesse API dei DataFrame di Spark, rendendo l'elaborazione in tempo reale accessibile agli sviluppatori Spark.

---

## Streaming con Tabelle Delta Lake

Le tabelle Delta Lake sono perfette per gli scenari di streaming perché possono fungere **sia da Origine che da Sink** per Spark Structured Streaming, con la garanzia delle transazioni ACID.

### 1. Tabella Delta come Sink di Streaming (Ingestione)

L'uso più comune è acquisire un flusso di dati in tempo reale e scriverlo direttamente in una tabella Delta.

#### Meccanismo di Scrittura

Quando si scrive in streaming in una tabella Delta, si usa il metodo `writeStream` e si specifica `format("delta")`.
```Python
# 1. Definizione del percorso di output e del checkpoint
output_table_path = 'Tables/orders_processed'
checkpointpath = 'Files/delta/checkpoint'

# 2. Avvia la query di streaming
deltastream = transformed_df.writeStream.format("delta") \
    .option("checkpointLocation", checkpointpath) \
    .start(output_table_path)

print("Streaming avviato in orders_processed...")
```

#### Punti Chiave

- **Punto di Controllo (`checkpointLocation`):** Questo è cruciale per la durabilità e la tolleranza ai guasti. Il file di checkpoint memorizza lo stato esatto del flusso (quali dati sono stati elaborati e dove) su OneLake. Se il processo fallisce, può riavviarsi dal punto esatto in cui si è interrotto, garantendo la semantica **exactly-once** (ogni record viene processato una sola volta).
- **Accessibilità:** Una volta che i dati sono in streaming nella tabella Delta, sono immediatamente disponibili per la lettura da parte dei motori batch/query (come l'Endpoint SQL o Power BI Direct Lake) poiché **Delta Lake unifica batch e streaming**.

### 2. Tabella Delta come Origine di Streaming (Consumo)

È possibile leggere una tabella Delta come se fosse un flusso di dati, permettendo di creare pipeline _multi-hop_ (a più stadi) dove ogni tabella funge da origine per il processo successivo.

#### Configurazione e Opzioni

Per leggere una tabella come flusso, si usa `spark.readStream`.
```Python
# Carica un DataFrame di streaming dalla tabella Delta "orders_in"
stream_df = spark.readStream.format("delta") \
    .option("ignoreChanges", "true") \
    .table("orders_in") 
    
# Verifica che l'oggetto sia in streaming
stream_df.isStreaming # Restituisce True
```

#### Gestione dei Cambiamenti (Change Data Capture)

Per impostazione predefinita, un flusso che legge da una tabella Delta si aspetta solo operazioni di **aggiunta** (`APPEND`). Se una transazione successiva alla prima versione del flusso include operazioni di **`UPDATE`** o **`DELETE`**, il flusso si interrompe per mantenere la coerenza dei dati.

Per gestire questo, si usano due opzioni:

1. **`ignoreChanges` = `true`:** Ignora operazioni di `UPDATE` e `DELETE`. Il flusso continua a elaborare solo le nuove righe aggiunte.
2. **`ignoreDeletes` = `true`:** Ignora solo le operazioni di `DELETE`.

---

## Trasformazione e Query sul Flusso

Le trasformazioni sui dati in streaming utilizzano esattamente la stessa sintassi delle API dei DataFrame di Spark (selezioni, filtri, aggregazioni, ecc.).

### Esempio di Trasformazione (Aggiunta di Logica)

Nell'esempio seguente, il DataFrame di streaming (`stream_df`) viene trasformato prima di essere scritto.
```Python
from pyspark.sql.functions import col, expr, lit

transformed_df = stream_df.filter(col("Price").isNotNull()) \
    .withColumn('IsBike', expr("INSTR(Product, 'Bike') > 0").cast('int')) \
    .withColumn('Total', expr("Quantity * Price").cast('decimal'))
```

Questo codice:

1. **Filtra** le righe con un prezzo nullo (`Price IS NOT NULL`).
2. **Aggiunge** una colonna booleana (`IsBike`) basata sul nome del prodotto.
3. **Calcola** il totale dell'ordine.

Il risultato è un **nuovo DataFrame di streaming** (`transformed_df`) che può essere inviato a qualsiasi sink (come una seconda tabella Delta, `orders_processed`).

### Verifica e Stop del Flusso

Dopo l'avvio del processo di streaming, è possibile interrogare la tabella di output (`orders_processed`) come una normale tabella Delta per visualizzare i risultati del flusso in tempo quasi reale.

Quando il lavoro di streaming non è più necessario, è cruciale arrestarlo per liberare le risorse di calcolo ed evitare costi:
```Python
# Arresta il processo di streaming
deltastream.stop()
```

