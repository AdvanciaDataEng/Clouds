---
autore: Kevin Milli
---

## Introduzione ad Apache Spark

**Apache Spark** è un framework di elaborazione parallela open source essenziale per l'analisi e l'elaborazione di dati su larga scala (**Big Data**). La sua popolarità deriva dalla capacità di elaborare rapidamente grandi volumi di dati, distribuendo il lavoro su un **cluster** di nodi di calcolo.

In Microsoft Fabric, Spark è integrato per l'uso in contesti come **Lakehouse**, **Azure HDInsight**, **Azure Synapse Analytics** e **Microsoft Fabric** stesso, semplificando l'incorporazione dell'elaborazione dati di Spark nelle soluzioni di analisi complessive.

### Architettura e Funzionamento

Spark utilizza un approccio "divide et impera" per l'elaborazione distribuita:

- **Pool di Spark (Cluster):** In Microsoft Fabric, è l'insieme di nodi di calcolo che distribuiscono le attività di elaborazione.
- **Nodo Head (Driver):** Contiene il programma _driver_ che coordina i processi distribuiti.
- **Nodi di Lavoro (Worker):** Eseguono i processi _executor_, ovvero le attività di elaborazione dei dati effettive.

Il cluster Spark coordina l'accesso e l'elaborazione dei dati in archivi compatibili, come un **Lakehouse** basato su **OneLake**.

### Linguaggi Supportati

Spark può eseguire codice in diversi linguaggi, con una forte enfasi su:

- **PySpark:** Variante di Python ottimizzata per Spark. È il linguaggio predominante per l'ingegneria e la data science in Spark.
- **Spark SQL:** Utilizzato per interrogare i dati tramite comandi SQL.
- Altri linguaggi includono Java, Scala (scripting basato su Java) e Spark R.

---

## Configurazione Avanzata di Spark in Microsoft Fabric

Microsoft Fabric offre diverse opzioni per ottimizzare l'ambiente Spark per soddisfare esigenze specifiche di carico di lavoro, costi e prestazioni.

### 1. Pool di Spark e Configurazione dei Nodi

Un **Pool di Spark** è il set di risorse di calcolo. Fabric fornisce un **Pool di Avvio** predefinito per l'avvio rapido e consente la creazione di **Pool di Spark personalizzati**.

- **Configurazione:** Gestibile tramite il **Portale di amministrazione** (Impostazioni capacità > Data Engineering/Science Settings).
- **Famiglia di Nodi:** Definisce il tipo di macchine virtuali per i nodi. I nodi **ottimizzati per la memoria** sono spesso preferiti per le prestazioni in carichi di lavoro analitici.
- **Scalabilità Automatica:** Abilita il provisioning automatico dei nodi in base alle esigenze, definendo un numero minimo e massimo di nodi.
- **Allocazione Dinamica:** Permette di allocare dinamicamente i processi _executor_ sui nodi di lavoro in base ai volumi di dati, migliorando l'efficienza.

### 2. Runtime e Ambienti

Un **Runtime** di Spark definisce le versioni di Apache Spark, Delta Lake, Python e altri componenti software installati.

- **Ambienti:** Consentono di definire configurazioni specifiche che includono una versione di runtime, librerie personalizzate e impostazioni di configurazione di Spark per diverse operazioni di elaborazione dati.
- **Librerie:** È possibile installare librerie pubbliche da **PyPI** o librerie personalizzate tramite file di pacchetto. L'ampia disponibilità di librerie Python (tramite PySpark) copre la maggior parte delle esigenze di elaborazione.
- **Tip:** È possibile impostare un ambiente personalizzato come ambiente predefinito a livello di area di lavoro.

### 3. Ottimizzazioni delle Prestazioni

#### Motore di Esecuzione Nativo (Native Execution Engine)

È un motore di elaborazione **vettorializzato** che esegue le operazioni Spark direttamente nell'infrastruttura Lakehouse, offrendo un significativo miglioramento delle prestazioni, specialmente su set di dati di grandi dimensioni in formato **Parquet** o **Delta**. Si basa su componenti OSS chiave come **Velox** (libreria di accelerazione database C++) e **Apache Gluten (incubating)**.

- **Abilitazione:**
**A livello di Ambiente:** Tramite l'interruttore nella sezione _Accelerazione_ delle impostazioni dell'ambiente o impostando le proprietà Spark:
```
spark.native.enabled: true
spark.shuffle.manager: org.apache.spark.shuffle.sort.ColumnarShuffleManager
```

**A livello di Notebook/Script:** All'inizio del codice:
```JSON
%%configure 
{ 
 "conf": {
	 "spark.native.enabled": "true", 
	 "spark.shuffle.manager": "org.apache.spark.shuffle.sort.ColumnarShuffleManager" 
 } 
}
```

- **Intuito:** Se il motore nativo incontra funzionalità non supportate (es. UDF, `array_contains`), si verifica un **fallback automatico** al motore Spark tradizionale per garantire la continuità del flusso di lavoro.

#### Modalità di Concorrenza Elevata (High Concurrency Mode)

Ottimizza l'utilizzo delle risorse Spark consentendo la **condivisione delle sessioni Spark** tra più utenti o processi simultanei, purché siano nello stesso ambiente, dello stesso utente, con configurazioni Spark e pacchetti di libreria corrispondenti.

- **Funzionamento:** Se un notebook ha una sessione di concorrenza elevata attiva, altri notebook possono collegarsi ad essa, riducendo il tempo di avvio della sessione. La condivisione è sempre limitata al singolo utente per garantire l'isolamento del codice.
- **Tip:** Abilitata per impostazione predefinita in tutte le aree di lavoro Fabric.

---

## Esecuzione del Codice Spark

In Microsoft Fabric, è possibile eseguire codice Spark tramite due meccanismi principali:

### 1. Notebook

Ideali per l'esplorazione, l'analisi interattiva dei dati e la collaborazione.

- **Struttura:** Composti da _celle_ che possono contenere testo formattato (Markdown) o codice eseguibile (PySpark, Spark SQL, ecc.).
- **Esecuzione:** Il codice viene eseguito in modo interattivo, mostrando immediatamente i risultati.

### 2. Definizione del Processo Spark (Spark Job Definition)

Utilizzata per l'inserimento e la trasformazione dei dati come parte di un processo **automatizzato** (batch o streaming) eseguibile su richiesta o in base a una pianificazione.

- **Configurazione:** Si crea una definizione di processo Spark specificando lo script principale da eseguire (es. file binari come `.jar` o script Python), file di riferimento e il riferimento a un **Lakehouse** specifico (che funge da filesystem predefinito).

### Registrazione Automatica MLFlow (MLFlow Autologging)

**MLFlow** è una libreria open source per la gestione del ciclo di vita del Machine Learning (training e distribuzione).

- **Funzionalità:** Per impostazione predefinita, Microsoft Fabric utilizza **MLFlow Autologging** per registrare implicitamente i parametri, le metriche e gli artefatti del modello durante l'attività di esperimento di Machine Learning, riducendo la necessità di codice esplicito di tracciamento.

---

# Utilizzare i Dati in un DataFrame Spark

Apache Spark lavora nativamente con una struttura dati chiamata **RDD (Resilient Distributed Dataset)**. Sebbene gli RDD siano la base per l'elaborazione distribuita, nella pratica, per i dati strutturati si utilizza quasi esclusivamente il **DataFrame**, fornito dalla libreria **Spark SQL**.

- **DataFrame vs. RDD:** Il DataFrame è un'astrazione di livello superiore rispetto all'RDD. Offre ottimizzazioni automatiche (tramite l'**Optimizer Catalyst**) e un'interfaccia API più intuitiva e tipizzata, molto simile ai famosi DataFrame della libreria **Pandas** di Python, ma progettata per l'ambiente di calcolo distribuito di Spark.

---

## 1. Caricamento dei Dati in un DataFrame

Il caricamento dei dati è il primo passo per l'elaborazione. L'approccio di Spark permette di leggere direttamente da **OneLake** di Microsoft Fabric (Lakehouse), semplificando l'accesso ai dati.

### Inferenza dello Schema (Schema Inference)

Quando i dati includono una riga di intestazione, Spark può tentare di **dedurre (inferire)** automaticamente sia i nomi delle colonne che i tipi di dati.

**Esempio (PySpark):** Caricamento da `products.csv` con intestazione:
```Python
%%pyspark
# Legge il file CSV, assume che ci sia un header e inferisce lo schema dai dati.
# spark.read è il punto d'ingresso per la lettura dei dati.
df = spark.read.load('Files/data/products.csv',
    format='csv',
    header=True # Indica a Spark che la prima riga è l'intestazione
)
# display() è una funzione specifica di Fabric/Databricks/Synapse per mostrare i risultati in modo ottimizzato.
display(df.limit(10))
```

- **Magic Command (`%%pyspark` / `%%spark`):** Queste righe _magic_ all'inizio della cella definiscono il linguaggio di esecuzione (PySpark o Scala).

**Esempio (Scala):**
```scala
%%spark
val df = spark.read.format("csv").option("header", "true").load("Files/data/products.csv")
display(df.limit(10))
```

### Specifica di uno Schema Esplicito (Explicit Schema)

È cruciale specificare uno schema esplicito quando:

1. Il file di dati **non contiene intestazioni**.
2. Si vuole **garantire la correttezza dei tipi di dati**, prevenendo errori dovuti a inferenze errate.
3. Si ha un **volume di dati molto grande** (l'inferenza dello schema richiede una passata completa sui dati).

**Esempio (PySpark):** Specificare lo schema per un file senza intestazione (`product-data.csv`).

```Python
from pyspark.sql.types import *
from pyspark.sql.functions import * # Import per funzioni comuni

# Definizione dello Schema esplicito usando StructType e StructField
productSchema = StructType([
    StructField("ProductID", IntegerType()),
    StructField("ProductName", StringType()),
    StructField("Category", StringType()),
    StructField("ListPrice", FloatType()) # Usare i tipi più specifici possibili
])

df = spark.read.load('Files/data/product-data.csv',
    format='csv',
    schema=productSchema, # Applica lo schema definito
    header=False) # Indica che non c'è intestazione
display(df.limit(10))
```

|Tipo di Dati Spark|Tipi Equivalenti|Descrizione|
|---|---|---|
|`IntegerType()`|`int`|Numeri interi a 4 byte.|
|`FloatType()`|`float`|Numeri in virgola mobile a singola precisione.|
|`DoubleType()`|`double`|Numeri in virgola mobile a doppia precisione (consigliato per precisione).|
|`StringType()`|`str`|Sequenza di caratteri.|
|`TimestampType()`|`datetime`|Data e ora, con fuso orario.|

---

## 2. Manipolazione e Trasformazione dei DataFrame

I DataFrame in Spark supportano una vasta gamma di metodi che consentono la **trasformazione immutabile** dei dati (ogni operazione restituisce un **nuovo DataFrame**). Questo approccio è fondamentale per l'ottimizzazione del piano di esecuzione di Spark (Lazy Evaluation).

### Filtraggio e Selezione (select e where)

Le operazioni vengono spesso **concatenate** (chained) per costruire pipeline di trasformazione.

- **`select()`:** Seleziona colonne specifiche.
 
```Python
pricelist_df = df.select("ProductID", "ListPrice")
```

- **`where()`** o **`filter()`:** Filtra le righe in base a una condizione booleana.

```Python
# Uso di `where` con operatori logici (| per OR, & per AND)
bikes_df = df.select("ProductName", "Category", "ListPrice").where((df["Category"]=="Mountain Bikes") | (df["Category"]=="Road Bikes"))
display(bikes_df)
```
    
- **Intuito (Lazy Evaluation):** Queste operazioni sono _trasformazioni_ e non vengono eseguite immediatamente. Spark le registra e le esegue solo quando viene richiesta un'_azione_ (ad esempio `display()`, `count()`, o `write()`), ottimizzando l'intera sequenza.

### Raggruppamento e Aggregazione (groupBy e funzioni di aggregazione)

Utilizzati per calcolare metriche aggregate sui dati, come conteggi, medie e somme.

- **`groupBy()`:** Raggruppa le righe in base ai valori di una o più colonne.
- **`count()`:** Funzione di aggregazione per contare le righe in ogni gruppo.

```Python
# Conta il numero di prodotti per ogni categoria
counts_df = df.select("ProductID", "Category").groupBy("Category").count()
display(counts_df)
```

|Categoria|count|
|---|---|
|Cuffie|3|
|Biciclette da montagna|32|
|...|...|

---

## 3. Salvataggio Ottimizzato dei DataFrame

Salvare i dati trasformati in un formato efficiente è essenziale per l'analisi successiva.

### Salvataggio Standard (Parquet)

Il formato **Parquet** è il formato a colonne preferito nell'ecosistema Big Data.

- **Vantaggi di Parquet:**
    - **Orientato alle Colonne:** Ideale per carichi di lavoro analitici che interrogano solo un sottoinsieme di colonne.
    - **Compressione Efficiente:** Riduce lo spazio su disco e l'I/O.
    - **Schema Evolution:** Gestisce i cambiamenti nello schema dei dati.

```Python
# Modalità "overwrite" (sovrascrive), "append" (aggiunge) o "errorifexists"
bikes_df.write.mode("overwrite").parquet('Files/product_data/bikes.parquet')
```

### Partizionamento (partitionBy) per Ottimizzazione I/O

Il **Partizionamento** è una tecnica di ottimizzazione fisica che organizza i dati nel data lake in sottocartelle basate sui valori di una o più colonne.

- **Come Funziona:** Riduce l'I/O del disco. Quando una query filtra sulla colonna di partizionamento (es. `WHERE Category = 'Road Bikes'`), Spark può **eliminare le cartelle** che non contengono i dati richiesti (Partition Pruning).

```Python
# Partiziona i dati in sottocartelle in base alla colonna "Category"
bikes_df.write.partitionBy("Category").mode("overwrite").parquet("Files/bike_data")
```

**Struttura Risultante nel Data Lake:**
```
Files/bike_data/
├── Category=Mountain Bikes/
│   └── part-00000-....parquet
└── Category=Bici da strada/
    └── part-00001-....parquet
```

---

## 4. Caricamento di Dati Partizionati

Quando si leggono dati partizionati, è possibile sfruttare la struttura della cartella per caricare solo i dati necessari, migliorando drasticamente la velocità di lettura.

- **Lettura con Partizionamento Esplicito:** Specificando la sottocartella, si forzano i filtri in fase di lettura.
```Python
# Carica ESCLUSIVAMENTE i dati contenuti nella sottocartella Road Bikes
# Questa operazione è molto più veloce rispetto a caricare tutto e poi filtrare
road_bikes_df = spark.read.parquet('Files/bike_data/Category=Road Bikes')
display(road_bikes_df.limit(5))
```

- **Intuito:** I valori della colonna di partizionamento (`Category` in questo caso) non sono contenuti nei file Parquet all'interno della sottocartella, ma vengono _letti dal nome della cartella_ da Spark. Questo è il motivo per cui la colonna `Category` sarà comunque presente nel DataFrame `road_bikes_df`.
---

# Usare i Dati con Spark SQL

**Spark SQL** è un modulo di Spark che facilita l'uso di espressioni SQL per interrogare e manipolare i dati. L'API DataFrame è in realtà parte della libreria Spark SQL, il che sottolinea l'importanza dell'approccio relazionale in Spark.

## 1. Il Catalogo Spark: Metastore dei Dati

Il **Catalogo Spark** è un metastore integrato che funge da _riferimento centrale_ per gli oggetti dati relazionali (tabelle, viste, database) all'interno dell'ambiente Spark. Permette l'integrazione fluida tra il codice nativo di Spark (PySpark, Scala) e le query SQL.

### Viste Temporanee (Temp Views)

Il modo più rapido per rendere accessibile un DataFrame tramite SQL è creare una **Vista Temporanea**.

- **Natura:** Una vista temporanea è un alias _valido solo per la sessione Spark corrente_. Viene eliminata automaticamente al termine della sessione.

```Python
%%pyspark
# Crea una vista temporanea chiamata 'products_view'
df.createOrReplaceTempView("products_view")
```

### Tabelle Persistenti (Managed e External)

Per mantenere la struttura dei dati al di là della sessione corrente, è necessario creare delle **Tabelle Persistenti** nel catalogo. In Microsoft Fabric, i dati delle tabelle risiedono nel Lakehouse (OneLake).

- **Tabelle Gestite (Managed Tables):**
    - Dati e metadati sono gestiti da Spark. I dati sottostanti vengono archiviati nella sezione **Tabelle** del Lakehouse.
    - **Intuizione:** Se si elimina la tabella gestita, Spark **elimina anche i dati sottostanti** nel Lakehouse.

```Python
# Salva il DataFrame come nuova tabella 'products'
# Utilizzando il formato Delta (il preferito in Fabric)
df.write.format("delta").saveAsTable("products")
```
    
- **Tabelle Esterne (External Tables):**
    
    - Vengono definiti i metadati nel catalogo Spark, ma i dati sottostanti rimangono in una posizione esterna specificata (tipicamente nella sezione **File** del Lakehouse).
    - **Intuizione:** Se si elimina la tabella esterna, Spark **NON elimina i dati sottostanti**. Questo è ideale quando si vogliono gestire separatamente i metadati e i dati grezzi.

### Delta Lake (Il Formato Preferito)

Il formato di file **Delta** (tecnologia **Delta Lake**) è il formato di tabella consigliato in Microsoft Fabric per Spark SQL.

- **Funzionalità Chiave:**
    - **ACID Transactions:** Garantisce l'affidabilità dei dati in ambienti concorrenti.
    - **Schema Enforcement/Evolution:** Previene l'inserimento di dati errati e gestisce i cambiamenti di schema.
    - **Time Travel (Controllo delle versioni):** Consente di risalire a versioni precedenti dei dati.

## 2. Esecuzione di Query Spark SQL

È possibile eseguire query SQL in due modi principali: tramite API nel codice o direttamente tramite il magic command SQL.

### Tramite API Spark SQL (Nei linguaggi come PySpark)

Si utilizza il metodo `spark.sql()` per eseguire una query SQL e il risultato viene restituito come un **nuovo DataFrame Spark**.

```Python
%%pyspark
# Esegue una query SQL sulla tabella 'products'
bikes_df = spark.sql("SELECT ProductID, ProductName, ListPrice \
                     FROM products \
                     WHERE Category IN ('Mountain Bikes', 'Road Bikes')")
display(bikes_df)
```

### Tramite Magic Command (`%%sql`) nel Notebook

Questo approccio è più naturale per gli analisti SQL. Si utilizza il magic command `%%sql` all'inizio della cella, e il risultato viene visualizzato immediatamente nel notebook.

```SQL
%%sql

-- Query SQL nativa
SELECT Category, COUNT(ProductID) AS ProductCount
FROM products
GROUP BY Category
ORDER BY ProductCount DESC
```

---

# Visualizzare i Dati in un Notebook Spark

La visualizzazione è un passaggio cruciale nell'analisi interattiva dei dati. I notebook di Fabric offrono strumenti integrati e la flessibilità di potenti librerie Python.

## 1. Grafici di Notebook Predefiniti

Quando si esegue una query Spark SQL o si visualizza un DataFrame Spark (`display(df)`), l'interfaccia utente del notebook consente di passare dalla visualizzazione tabellare a una visualizzazione grafica (grafico a barre, a torta, a linee, ecc.) tramite il pannello di output.

- **Vantaggio:** Utile per un **riepilogo visivo rapido** senza scrivere codice di visualizzazione.
- **Limitazione:** Controllo limitato sulla personalizzazione e sulla formattazione del grafico.

## 2. Visualizzazioni Personalizzate con Librerie Python

Per un controllo maggiore sulla formattazione e sulla tipologia di grafico, si ricorre alle librerie di visualizzazione Python.

- **Requisito Chiave:** La maggior parte delle librerie di visualizzazione Python (come Matplotlib, Seaborn, Plotly) richiede che i dati siano in formato **Pandas DataFrame** e non Spark DataFrame.

### Esempio con Matplotlib

La libreria **Matplotlib** è la base di molte librerie grafiche Python.

1. **Conversione dei Dati:** Si esegue prima una query Spark SQL, e poi si utilizza il metodo `.toPandas()` per convertire il risultato da un DataFrame Spark a un DataFrame Pandas.
2. **Visualizzazione:** Si usano le API della libreria per disegnare il grafico.

```Python
%%pyspark
from matplotlib import pyplot as plt

# 1. Recupera i dati aggregati tramite Spark SQL e converti in Pandas
data = spark.sql("SELECT Category, COUNT(ProductID) AS ProductCount \
                  FROM products \
                  GROUP BY Category \
                  ORDER BY ProductCount DESC").toPandas()

# 2. Inizia la configurazione del grafico
plt.clf()
fig = plt.figure(figsize=(12,8))

# 3. Crea il grafico a barre
plt.bar(x=data['Category'], height=data['ProductCount'], color='orange')

# 4. Personalizza e mostra il grafico
plt.title('Conteggio Prodotti per Categoria')
plt.xlabel('Categoria')
plt.ylabel('Prodotti')
plt.xticks(rotation=70) # Ruota le etichette per leggibilità
plt.grid(color='#95a5a6', linestyle='--', linewidth=1, axis='y', alpha=0.7)

plt.show()
```

- **Alternativa:** È possibile utilizzare altre librerie di alto livello come **Seaborn** (costruito su Matplotlib per grafici statistici eleganti) o **Plotly/Altair** (per visualizzazioni interattive) per creare grafici altamente personalizzati e avanzati.

