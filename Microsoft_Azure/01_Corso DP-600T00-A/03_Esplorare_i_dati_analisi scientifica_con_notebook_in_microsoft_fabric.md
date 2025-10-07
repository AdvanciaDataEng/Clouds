
L'analisi scientifica dei dati è una disciplina **multidiscplinare** che impiega metodi scientifici, processi, algoritmi e sistemi per estrarre **conoscenza** e **intuizioni** da dati strutturati e non strutturati. Microsoft Fabric offre una piattaforma unificata per semplificare questo processo, integrando Data Engineering, Data Science, Data Warehouse e Business Intelligence.

## Il Processo di Data Science e l'Exploratory Data Analysis (EDA)

In un progetto di data science, la fase di **Exploratory Data Analysis (EDA)** è cruciale. L'EDA è l'analisi preliminare che getta le basi per tutti i passaggi successivi, concentrandosi sulla comprensione dei modelli, l'individuazione di anomalie, la verifica delle ipotesi e il controllo dei presupposti relativi ai dati sottostanti.

### Fasi del Processo di Data Science

Il processo si articola in passaggi ben definiti:

1. **Definire il Problema (Business Understanding)**:
    - Determinare, con gli stakeholder, l'obiettivo del modello (cosa deve prevedere) e i criteri di successo.
    
2. **Ottenere e Ingerire i Dati (Data Acquisition)**:
    - Identificare le origini dati e renderli accessibili, tipicamente archiviandoli in una **Lakehouse** di Fabric (che utilizza **OneLake** per l'archiviazione unificata dei dati).
    
3. **Preparare i Dati (EDA e Preprocessing)**:
    - Caricare e leggere i dati dalla Lakehouse in un **notebook** (ad es., come DataFrame Spark o Pandas).
    - **EDA**: Comprendere le distribuzioni, identificare valori anomali, studiare le correlazioni.
    - Pulire e trasformare i dati (gestione dei dati mancanti, normalizzazione, feature engineering) in base ai requisiti del modello.
    
4. **Eseguire il Training del Modello (Modeling)**:
    - Scegliere l'algoritmo e ottimizzare gli **iperparametri** tramite tentativi ed errori.
    - Utilizzare **MLflow** integrato in Fabric per tracciare e monitorare gli esperimenti.
    
5. **Generare Informazioni Dettagliate (Insights Generation)**:
    - Utilizzare l'assegnazione dei punteggi (batch scoring) per generare le stime richieste.
    - Integrare i risultati per alimentare soluzioni di reporting a valle, come un report **Power BI**.

## Esplorare i Dati con i Notebook di Microsoft Fabric

I notebook di Fabric sono l'ambiente principale per l'EDA e lo sviluppo di codice in data science e data engineering. Essi offrono una superficie interattiva, basata su **Apache Spark**, dove si combinano codice, testo esplicativo in **Markdown** e risorse multimediali.

### Intuizione sui Notebook

I notebook creano una **narrazione** del progetto di data science. Questo facilita la **riproducibilità**, la **condivisione del codice** e la **creazione rapida di prototipi** (rapid prototyping), accelerando il ciclo di vita del progetto.

### Linguaggi Supportati

I notebook di Microsoft Fabric supportano quattro linguaggi Apache Spark:

- **PySpark** (Python) 
- **Spark** (Scala)
- **Spark SQL**
- **SparkR** (R)

### Caratteristiche Chiave dei Notebook di Fabric

|Caratteristica|Descrizione|
|---|---|
|**Integrazione Lakehouse**|Facile accesso a Lakehouse Explorer per aggiungere Lakehouse nuove o esistenti. Il **trascinamento della selezione** genera automaticamente frammenti di codice per la lettura dei dati (tabelle, file, immagini).|
|**Risorse (Resources)**|Un'area di archiviazione simile a Unix per file di piccole dimensioni (codice, set di dati, immagini) accessibile direttamente con percorsi relativi (ad es. `builtin/MyFile.py`).|
|**IntelliSense e Code Completion**|Funzionalità avanzate che includono l'evidenziazione della sintassi, il contrassegno degli errori e il completamento automatico del codice, migliorando la velocità e l'accuratezza di scrittura.|
|**Esploratore Variabili (Variable Explorer)**|Strumento integrato (solo per kernel **PySpark/Python**) che tiene traccia automaticamente dello stato delle variabili dopo l'esecuzione delle celle. Mostra **Nome**, **Tipo**, **Lunghezza** e **Valore** per una rapida panoramica.|
|**Data Wrangler**|Strumento low-code integrato per la preparazione e pulizia dei dati, che genera codice Python riutilizzabile e ripetibile.|
|**Copilot per Data Science (Anteprima)**|Un assistente AI che genera risposte e frammenti di codice per l'analisi e la visualizzazione dei dati direttamente tramite chat o _magic commands_ nel notebook.|

## Gestione delle Librerie e Dipendenze

La gestione delle librerie in Fabric è flessibile e può essere eseguita a due livelli per garantire che gli ambienti siano configurati correttamente.

### 1. Gestione a Livello di Area di Lavoro (Ambienti)

Le librerie installate a livello di **Area di Lavoro** (tramite la funzionalità **Ambienti** in _Impostazioni area di lavoro > Data Engineering/Science > Gestione librerie_) sono persistenti e disponibili per **tutti** i notebook e i job Spark all'interno di quell'area di lavoro, anche in diverse sessioni.

- **Vantaggio**: Crea un ambiente comune e standardizzato per la collaborazione.
- Supporta l'installazione di librerie da **feed Python** (ad es. PyPI) e **librerie personalizzate** (`.whl`, `.py`, `.jar`, `.tar.gz`).

### 2. Installazione in Linea (Sessione Corrente)

È possibile installare librerie direttamente all'interno di un notebook utilizzando i comandi **magic** di `pip`:

- **Comando**:
```Python
%pip install nome-libreria
```
- **Applicabilità**: Le librerie installate con `%pip` sono disponibili **SOLO per la sessione Spark corrente** del notebook.
- **Attenzione**: L'esecuzione di un comando `%pip` riavvierà l'interprete Python, causando la **perdita** di tutte le variabili definite precedentemente nella sessione.

### Esempio Pratico: Installazione e Importazione

```Python
# Installazione della libreria Seaborn per la sessione corrente
%pip install seaborn

# Installazione della libreria scikit-learn per la sessione corrente
%pip install sklearn

# Importazione delle librerie
import seaborn as sns
from sklearn import datasets
```

## Collaborazione in Tempo Reale nei Notebook

La collaborazione è una caratteristica distintiva dei notebook di Microsoft Fabric, essenziale per l'efficienza e la risoluzione collettiva dei problemi.

### Programmazione in Coppia e Debug Remoto

Più utenti possono **modificare contemporaneamente** lo stesso notebook.

- **Visibilità in Tempo Reale**: I collaboratori possono vedere in tempo reale i movimenti del cursore, le selezioni e le modifiche del codice degli altri, proprio come in un editor di testo collaborativo (es. Google Docs).
- **Vantaggio**: Ideale per la **programmazione in coppia**, il **debug remoto** e per la condivisione istantanea di soluzioni o l'apprendimento da colleghi più esperti.
- **Condivisione**: I notebook possono essere facilmente condivisi con autorizzazioni specifiche (**Visualizza**, **Modifica**, **Esegui**) per controllare l'accesso e favorire la democratizzazione dei dati e delle informazioni.

## EDA: Distribuzioni e Dati Mancanti

L'EDA si concentra sull'indagine delle caratteristiche chiave dei dati.

### Tipi di Distribuzione dei Dati

Comprendere la distribuzione di una variabile è fondamentale per la scelta del modello statistico o di ML appropriato. 
Si usano tecniche di visualizzazione come **istogrammi** e **box plot** (diagrammi a scatola).

- **Distribuzione Normale (Gaussiana)**: A forma di campana, simmetrica attorno alla media. Molti algoritmi ML (es. Regressione Lineare) assumono la normalità.
- **Distribuzione Asimmetrica (Skewed)**: Non simmetrica. Può essere _asimmetrica a destra_ (coda lunga a destra) o _asimmetrica a sinistra_. Spesso richiede trasformazioni (es. logaritmo) per la modellazione.
- **Distribuzione Uniforme**: Tutti i valori hanno circa la stessa probabilità di occorrenza.
- **Distribuzione di Poisson/Binomiale**: Comuni per dati di conteggio o dati discreti.

### Gestione dei Dati Mancanti (Missing Data)

I dati mancanti (`NaN`, `None`, o valori speciali) sono comuni e devono essere gestiti efficacemente, poiché possono distorcere i risultati dell'analisi e del modello.

|Strategia|Descrizione|
|---|---|
|**Cancellazione (Listwise Deletion)**|Rimuovere l'intera riga (osservazione) che contiene almeno un valore mancante. **Attenzione**: Rischio di perdere dati preziosi e introdurre _bias_ se i dati non mancano in modo casuale (**MCAR** - Missing Completely at Random).|
|**Imputazione con Statistiche (Mean/Median Imputation)**|Sostituire il valore mancante con la media o la mediana della colonna. La mediana è preferibile per le distribuzioni asimmetriche (skewed).|
|**Imputazione con Valore Costante/Modalità**|Sostituire con una costante (es. 0) o la moda (valore più frequente) per le variabili categoriche.|
|**Imputazione Avanzata (Model-based Imputation)**|Utilizzare un modello di Machine Learning (es. k-NN, Regressione) per stimare il valore mancante basandosi sugli altri dati disponibili. È più accurata ma anche più complessa.|

### Librerie di Visualizzazione

Librerie Python come **Pandas**, **Matplotlib**, **Seaborn** e **Plotly** sono essenziali per l'EDA e sono facilmente installabili e utilizzabili nei notebook di Fabric.

```Python
# Esempio di codice EDA tipico in un notebook Fabric PySpark

import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Assumendo che 'df_spark' sia un DataFrame Spark letto da Lakehouse
# Conversione in Pandas per l'EDA e la visualizzazione su piccola scala
df_pandas = df_spark.toPandas()

# 1. Visualizzazione della distribuzione con un Istogramma
plt.figure(figsize=(8, 6))
sns.histplot(df_pandas['colonna_numerica'], kde=True)
plt.title('Distribuzione della Colonna Numerica')
plt.show()

# 2. Controllo dei dati mancanti
print(df_pandas.isnull().sum())

# 3. Imputazione con la mediana
mediana = df_pandas['colonna_con_mancanti'].median()
df_pandas['colonna_con_mancanti'].fillna(mediana, inplace=True)
```

# Caricare i dati per l'esplorazione

Il caricamento e l'esplorazione dei dati sono i primi passaggi di qualsiasi progetto di data science. Implicano la comprensione della struttura, del contenuto e dell'origine dei dati, fondamentali per l'analisi successiva.

Dopo la connessione a un'origine dati, è possibile salvare i dati in un [**lakehouse**](https://learn.microsoft.com/it-it/fabric/data-engineering/lakehouse-overview/) di Microsoft Fabric. È possibile usare il lakehouse come posizione centrale per archiviare tutti i file strutturati, semistrutturati e non strutturati. È quindi possibile connettersi facilmente al lakehouse ogni volta che si desidera accedere ai dati per l'esplorazione o la trasformazione.

## Caricare i dati usando i notebook

I notebook in Microsoft Fabric facilitano il processo di gestione degli asset di dati. Una volta archiviati gli asset di dati nel lakehouse, è possibile generare facilmente codice all'interno del notebook per inserire tali asset.

Si consideri uno scenario in cui un data engineer ha già trasformato i dati dei clienti e lo ha archiviato nella lakehouse. Un data scientist può caricare facilmente i dati usando notebook per un'ulteriore esplorazione per creare un modello di Machine Learning. Ciò consente di iniziare immediatamente, indipendentemente dal fatto che si tratti di manipolazioni aggiuntive dei dati, analisi esplorativa dei dati o sviluppo di modelli.

Per illustrare l'operazione di caricamento si creerà un file Parquet di esempio. 
Il codice PySpark seguente crea un dataframe di dati dei clienti e lo scrive in un file Parquet nel lakehouse.

[Apache Parquet](https://parquet.apache.org/) è un formato di archiviazione dati open source orientato alle colonne. È progettato per un'efficiente archiviazione e recupero dei dati ed è noto per le prestazioni elevate e la compatibilità con molti framework di elaborazione dati.

```python
from pyspark.sql import Row

Customer = Row("firstName", "lastName", "email", "loyaltyPoints")

customer_1 = Customer('John', 'Smith', 'john.smith@contoso.com', 15)
customer_2 = Customer('Anna', 'Miller', 'anna.miller@contoso.com', 65)
customer_3 = Customer('Sam', 'Walters', 'sam@contoso.com', 6)
customer_4 = Customer('Mark', 'Duffy', 'mark@contoso.com', 78)

customers = [customer_1, customer_2, customer_3, customer_4]
df = spark.createDataFrame(customers)

df.write.parquet("<path>/customers")
```

Per generare il percorso del file Parquet, selezionare i puntini di sospensione in Lakehouse explorer e quindi scegliere **Copia percorso ABFS** o **Copia percorso relativo per Spark**. Se si scrive codice Python, è possibile usare l'opzione **Copia api file** o **Copia percorso ABFS**.

Il codice seguente carica il file parquet in un dataframe.
```python
df = spark.read.parquet("<path>/customers")

display(df)
```

In alternativa, è anche possibile generare il codice per caricare automaticamente i dati nel notebook. Scegliere il file di dati e quindi selezionare **Carica dati**. Successivamente, è necessario scegliere l'API che si vuole usare.

Sebbene il file Parquet nell'esempio precedente sia archiviato nel lakehouse, è possibile caricare i dati anche da origini esterne come Archiviazione BLOB di Azure.

```python
blob_account_name = ""
blob_container_name = ""
blob_relative_path = ""
blob_sas_token = ""

wasbs = f'wasbs://{blob_container_name}@{blob_account_name}.blob.core.windows.net/{blob_relative_path}?{blob_sas_token}'

df = spark.read.parquet(wasbs)
df.show()
```

È possibile seguire passaggi simili per caricare altri tipi di file, come `.csv`, `.json` e `.txt`. Sostituire semplicemente il `.parquet` metodo con il metodo appropriato per il tipo di file, ad esempio:

```python
# For CSV files
df_csv = spark.read.csv('<path>')

# For JSON files
df_json = spark.read.json('<path>')

# For text files
df_text = spark.read.text('<path>')
```

---

# Informazioni sulla distribuzione dei dati

Comprendere la distribuzione dei dati è essenziale per un'efficace analisi dei dati, visualizzazione e compilazione di modelli.

Se un set di dati ha una distribuzione asimmetrica, significa che i punti dati non vengono distribuiti uniformemente e tendono a allinearsi più verso destra o sinistra. Ciò può causare una stima imprecisa di punti dati da gruppi sottorappresentati o ottimizzati in base a una metrica inappropriata.

## Importanza della distribuzione dei dati

Di seguito sono riportate le aree chiave in cui la comprensione della distribuzione dei dati può migliorare l'accuratezza dei modelli di Machine Learning.

|Passo|Descrizione|
|---|---|
|**Analisi esplorativa dei dati (EDA)**|Comprendere la distribuzione dei dati semplifica l'esplorazione di un nuovo set di dati e la ricerca di modelli.|
|**Pre-elaborazione dei dati**|Alcune tecniche di pre-elaborazione, come la normalizzazione o la standardizzazione, vengono usate per rendere i dati più normalmente distribuiti, che è un presupposto comune in molti modelli.|
|**Selezione del modello**|I diversi modelli fanno presupposti diversi sulla distribuzione dei dati. Ad esempio, alcuni modelli presuppongono che i dati vengano normalmente distribuiti e che non funzionino correttamente se questo presupposto viene violato.|
|**Migliorare le prestazioni del modello**|La trasformazione della variabile di destinazione per ridurre l'asimmetria può linearizzare la destinazione, utile per molti modelli. Ciò può ridurre l'intervallo di errori e potenzialmente migliorare le prestazioni del modello.|
|**Pertinenza del modello**|Una volta distribuito un modello nell'ambiente di produzione, è importante che rimanga rilevante nel contesto dei dati più recenti. Se si verifica un'asimmetria dei dati, ovvero la distribuzione dei dati cambia nell'ambiente di produzione rispetto a quello usato durante il training, il modello potrebbe uscire dal contesto.|

Comprendere la distribuzione dei dati può migliorare il processo di compilazione del modello. Consente di stabilire presupposti più accurati identificando la media, la diffusione e l'intervallo di una variabile casuale nelle funzionalità e nella destinazione.

Verranno ora esaminati alcuni dei tipi di distribuzione dei dati più comuni, ad esempio le distribuzioni normali, binomiali e uniformi.

## Distribuzione normale

Una [distribuzione normale](https://en.wikipedia.org/wiki/Normal_distribution) è rappresentata da due parametri: la media e la deviazione standard. La media indica dove la curva a campana è centrata e la deviazione standard indica la distribuzione della distribuzione.

Di seguito è riportato un esempio di una normale funzionalità distribuita. Il codice seguente genera i dati per la `var` funzionalità a scopo dimostrativo.

```python
import seaborn as sns
import numpy as np
​import matplotlib.pyplot as plt

# Set the mean and standard deviation
mu, sigma = 0, 0.1 

# Generate a normally distributed variable
var = np.random.normal(mu, sigma, 1000)

# Create a histogram of the variable using seaborn's histplot
sns.histplot(var, bins=30, kde=True)

# Add title and labels
plt.title('Histogram of Normally Distributed Variable')
plt.xlabel('Value')
plt.ylabel('Frequency')

# Show the plot
plt.show()
```

Si noti che la `var` caratteristica è distribuita normalmente, in cui la media e la mediana (50% percentile) devono essere più o meno uguali. Per le distribuzioni asimmetriche, la media tende a sporgersi verso la coda più pesante.

Tuttavia, questi sono controlli euristici e la determinazione effettiva vengono eseguiti usando test statistici specifici come [**il test Shapiro-Wilk**](https://en.wikipedia.org/wiki/Shapiro%E2%80%93Wilk_test) o [**kolmogorov-Smirnov**](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) test per la normalità.

## Distribuzione binomiale

Supponiamo di voler comprendere quanto bene venga osservata una particolare caratteristica in un gruppo di pinguini.

Si decide di esaminare un set di dati di 200 pinguini per verificare se sono della specie _Adelie_ . Si tratta di un problema di [distribuzione binomiale](https://en.wikipedia.org/wiki/Binomial_distribution) perché esistono due possibili risultati (_Adelie_ o _non Adelie_), un numero fisso di prove (200 pinguini) e ogni prova è indipendente da altri.

Dopo aver analizzato il set di dati, si scopre che 150 pinguini sono della specie _Adelie_ .

Sapendo che i dati seguono una distribuzione binomiale, è possibile eseguire stime su set di dati o gruppi futuri di pinguini. Ad esempio, se si studia un altro gruppo di 200 pinguini, ci si può aspettare circa 150 essere della specie Adelie.

Il codice Python seguente traccia un istogramma della `is_adelie` variabile binomiale. L'argomento `discrete=True` in `sns.histplot` garantisce che i bin vengano trattati come intervalli discreti. Ciò significa che ogni barra nell'istogramma corrisponde esattamente a una categoria o valore booleano, rendendo il tracciato più facile da interpretare.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# Load the Penguins dataset from seaborn
penguins = sns.load_dataset('penguins')

# Create a binomial variable for 'species'
penguins['is_adelie'] = np.where(penguins['species'] == 'Adelie', 1, 0)

# Plot the distribution of 'is_adelie'
sns.histplot(data=penguins, x='is_adelie', bins=2, discrete=True)
plt.title('Binomial Distribution of Species')
plt.xticks([0, 1], ['Not Adelie', 'Adelie'])
plt.show()
```

## Distribuzione uniforme

Una distribuzione uniforme, nota anche come distribuzione rettangolare, è un tipo di distribuzione di probabilità in cui tutti i risultati sono ugualmente probabili. Ogni intervallo della stessa lunghezza sul supporto della distribuzione ha la stessa probabilità.

```python
import numpy as np
import matplotlib.pyplot as plt

# Generate a uniform distribution
uniform_data = np.random.uniform(-1, 1, 1000)

# Plot the distribution
plt.hist(uniform_data, bins=20, density=True)
plt.title('Uniform Distribution')
plt.xlabel('Value')
plt.ylabel('Frequency')
plt.show()
```

In questo codice, la `np.random.uniform` funzione genera 1000 numeri casuali distribuiti in modo uniforme tra -1 e 1. L'argomento `bins=30` specifica che i dati devono essere divisi in 30 bin e `density=True` garantisce che l'istogramma sia normalizzato per formare una densità di probabilità. Ciò significa che l'area sotto l'istogramma si integra con 1, utile quando si confrontano le distribuzioni.

>[!Nota]
>È probabile che si ottengano risultati diversi se si esegue il codice più volte. L'idea di base della casualità è che è imprevedibile e ogni volta che si esegue l'esempio, è possibile ottenere risultati diversi.
>È possibile controllare questo processo impostando un valore _seed_ con `np.random.seed`. Ciò è molto utile
>per il test e il debug nella fase di compilazione del modello, in quanto consente di riprodurre gli stessi
>risultati.

---

# Verificare la presenza di dati mancanti nei notebook

Per dati mancanti si intende l'assenza di valori in determinate variabili all'interno di un set di dati.

L'identificazione e la gestione dei dati mancanti è un aspetto fondamentale della fase di esplorazione e pre-elaborazione dei dati in un progetto di Machine Learning e il modo in cui è possibile gestirli può influire significativamente sulle prestazioni del modello.

I passaggi principali per gestire i dati mancanti includono la valutazione della quantità di dati mancanti, l'identificazione della natura dei dati mancanti e la scelta del metodo migliore per gestire i valori di dati mancanti.

## Identificare i dati mancanti

Per identificare se sono presenti dati mancanti nel set di dati, è possibile usare le funzioni `isnull()` o `isna()` da Pandas.

```python
import pandas as pd
import numpy as np

# Create a sample DataFrame with some missing values
data = {
    'A': [1, 2, np.nan],
    'B': [4, np.nan, np.nan],
    'C': [7, 8, 9]
}
df = pd.DataFrame(data)

# Check for missing data
print(df.isnull())
```

In questo modo viene restituito un dataframe con le stesse dimensioni di df, ma con **True** nelle posizioni in cui mancano valori (NaN) e **False** altrove.

Per ottenere il numero totale di valori mancanti nel dataframe, è possibile usare `df.isnull().sum()`. Verrà restituito il numero di valori mancanti per ogni colonna.

```python
df.isnull().sum()
```

## Valutare la natura dei valori mancanti

In un progetto di data science i valori mancanti possono verificarsi per diversi motivi e comprendere la loro natura è fondamentale per gestirli in modo appropriato.

Ecco alcuni tipi di valori mancanti:

- **Mancante completamente a caso (MCAR):** la mancanza di dati non è correlata ai valori di qualsiasi altra variabile ed è casuale. Questo è lo scenario ideale, ma spesso non è il caso nei dati reali.
- **Mancante in modo casuale (MAR):** la mancanza di dati è correlata ad altri valori delle variabili, ma non ai dati mancanti. Ad esempio, se le donne sono più propense a divulgare il loro numero di passaggi giornalieri rispetto agli uomini, i dati dei passaggi giornalieri sono MAR.
- **Mancanza non casuale (MNAR):** la mancanza di dati è correlata ai valori stessi dei dati mancanti. Ad esempio, le persone con salari più elevati potrebbero essere meno propensi a divulgare il loro reddito. La rimozione di questi record potrebbe introdurre distorsioni nel modello, impedendogli di riflettere in modo accurato le informazioni complete contenute nei dati.

Comprendere la natura dei valori mancanti nel set di dati può guidarti su come gestirli. Per MCAR e MAR, è possibile scegliere metodi di eliminazione o imputazione. Per MNAR, questi metodi potrebbero introdurre distorsioni, quindi potrebbe essere preferibile raccogliere più dati o usare metodi basati su modello in grado di gestire i valori mancanti.

## Decidere come gestire i dati mancanti

L'approccio alla gestione dei dati mancanti può influire significativamente sui risultati dell'analisi e sulle prestazioni del modello. Ecco alcune strategie che potresti prendere in considerazione.

- **Ignorare:** Se manca solo una piccola quantità di dati, potrebbe non avere un impatto significativo sulle prestazioni del modello.
- **Togliere:** Se una determinata riga o colonna contiene molti valori mancanti, potrebbe essere preferibile rimuoverla completamente.
- **Imputare**: completare i valori mancanti con un valore specificato o una stima come media, mediana, moda, o usando un algoritmo di apprendimento automatico come [K-Nearest Neighbors (KNN)](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm).
- **Usarli come nuova caratteristica**: a volte, il fatto che un valore non sia presente può essere usato come informazione. Ad esempio, in un sondaggio su un prodotto, le domande senza risposta relative alla volontà di raccomandare il prodotto ad altri potrebbero indicare l'insoddisfazione dei clienti. In questo caso, la mancata risposta può essere una nuova caratteristica che indica una probabile insoddisfazione dei clienti.

Non esiste una soluzione universale per la gestione dei dati mancanti. L'approccio migliore dipende dalle specifiche del set di dati e dalla domanda a cui si sta tentando di rispondere.

---

# Applicare tecniche avanzate di esplorazione dei dati

Le tecniche avanzate di esplorazione dei dati, ad esempio l'analisi della correlazione e la riduzione della dimensionalità, consentono di individuare modelli e relazioni nascosti nei dati, fornendo informazioni preziose che possono guidare il processo decisionale.

## Correlazione

La correlazione è un metodo statistico usato per valutare la forza e la direzione della relazione lineare tra due variabili quantitative. Il coefficiente di correlazione varia da -1 a 1.

|Coefficiente di correlazione|Descrizione|
|---|---|
|**1**|Indica una **correlazione lineare** positiva perfetta. Con l'aumentare di una variabile, aumenta anche l'altra variabile.|
|**-1**|Indica una **correlazione lineare negativa** perfetta. Man mano che aumenta una variabile, l'altra variabile diminuisce.|
|**0**|Indica **nessuna correlazione lineare**. Le due variabili non hanno una relazione tra loro.|

Si userà ora il set di dati penguins per spiegare il funzionamento della correlazione.

>[!Nota]
>Il set di dati sui pinguini usato è un sottinsieme di dati raccolti e resi disponibili dal [dottor Kristen Gorman](https://www.uaf.edu/cfos/people/faculty/detail/kristen-gorman.php)
>e dalla Stazione Palmer, Antartide LTER, membro del [Network di Ricerca Ecologica a Lungo Termine](https://lternet.edu/).

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

# Load the penguins dataset
penguins = pd.read_csv('https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/penguins.csv')

# Calculate the correlation matrix
corr = penguins.corr()

# Create a heatmap
sns.heatmap(corr, annot=True)
plt.show()
```

La correlazione più forte nel set di dati è tra `FlipperLength` le variabili e `BodyMass` , con un coefficiente di correlazione pari a **0,87**. Questo suggerisce che pinguini con pinne più grandi tendono ad avere una massa corporea più grande.

L'identificazione e l'analisi delle correlazioni sono importanti per i motivi seguenti.

- **Analisi predittiva:** Se due variabili sono altamente correlate, è possibile stimare una variabile dall'altra.
- **Selezione funzionalità:** Se due funzionalità sono altamente correlate, è possibile eliminarle perché non forniscono informazioni univoche.
- **Informazioni sulle relazioni:** La correlazione consente di comprendere la relazione tra variabili diverse nei dati.

## Analisi dei componenti principale (PCA)

[L'analisi dei componenti principale può](https://en.wikipedia.org/wiki/Principal_component_analysis) essere usata sia per l'esplorazione che per la pre-elaborazione dei dati.

In molti scenari di dati reali vengono usati dati di dimensioni elevate che possono risultare difficili da usare. Se usato per l'esplorazione, PCA consente di ridurre il numero di variabili mantenendo al tempo stesso la maggior parte delle informazioni originali. In questo modo i dati sono più facili da usare e meno a elevato utilizzo di risorse per gli algoritmi di Machine Learning.

Per semplificare l'esempio, si sta lavorando con il set di dati penguins che contiene solo cinque variabili. Tuttavia, si seguirà un approccio simile quando si usa un set di dati più grande.

```python
import pandas as pd
import seaborn as sns
from sklearn.decomposition import PCA
import matplotlib.pyplot as plt

# Load the penguins dataset
penguins = pd.read_csv('https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/penguins.csv')

# Remove missing values
penguins = penguins.dropna()

# Prepare the data and target
X = penguins.drop('Species', axis=1)
y = penguins['Species']

# Initialize and apply PCA
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)

# Plot the data
plt.figure(figsize=(8, 6))
for color, target in zip(['navy', 'turquoise', 'darkorange'], penguins['Species'].unique()):
    plt.scatter(X_pca[y == target, 0], X_pca[y == target, 1], color=color, alpha=.8, lw=2,
                label=target)
plt.legend(loc='best', shadow=False, scatterpoints=1)
plt.title('PCA of Penguins dataset')
plt.show()
```

Applicando PCA nel set di dati penguins, è possibile ridurre queste cinque variabili a due componenti principali che acquisiscono la maggior varianza dei dati. Questa trasformazione riduce la dimensionalità dei dati da cinque dimensioni a due. È quindi possibile creare un grafico a dispersione 2D per visualizzare i dati e identificare i cluster di pinguini con caratteristiche simili.

Ogni punto del tracciato rappresenta un pinguino dal set di dati. I valori per i primi e i secondi componenti principali (_x_ e _y_) determinano la posizione di un punto.

Si tratta di nuove variabili create da PCA da combinazioni lineari delle `CulmenLength`variabili , `CulmenDepth``FlipperLength`, `BodyMass`, e `Species` . Il primo componente principale acquisisce la maggior varianza dei dati e ogni componente successivo acquisisce meno varianza.

I risultati mostrano una separazione tra pinguini di diverse specie. Ovvero, i punti dello stesso colore (specie) sono più vicini e punti di colori diversi sono più distanti. Le differenze nelle distribuzioni delle caratteristiche per classi diverse comportano questa separazione, suggerendo che è possibile distinguere queste specie in base ai relativi attributi.

# Visualizzare i grafici nei notebook

La visualizzazione dei dati è un aspetto chiave dell'esplorazione dei dati. Implica la presentazione dei dati in un formato grafico, rendendo più comprensibili e utilizzabili i dati complessi.

Con i notebook di Microsoft Fabric e i dataframe Apache Spark, i risultati tabulari vengono presentati automaticamente come grafici senza la necessità di scrivere codice aggiuntivo.

>[!Nota]
>È anche possibile usare librerie open source come [matplotlib](https://matplotlib.org/) e [tracciate](https://plotly.com/) tra le altre per migliorare
>l'esperienza di esplorazione dei dati.

## Informazioni sulle variabili categoriche e numeriche

Per le variabili categoriche, è importante comprendere le diverse categorie o livelli nella variabile. Ciò implica l'identificazione del numero di osservazioni presenti in ogni categoria, denominate conteggi o frequenze. Inoltre, comprendere la percentuale o la percentuale delle osservazioni rappresentate da ogni categoria è fondamentale.

Quando si tratta di variabili numeriche, è necessario considerare diversi aspetti. La tendenza centrale della variabile, che può essere la media, la media o la modalità, fornisce un riepilogo della variabile.

Le misure di dispersione come l'intervallo, l'intervallo interquartile (IQR), la deviazione standard o la varianza forniscono informazioni dettagliate sulla diffusione della variabile. Infine, comprendere la distribuzione della variabile è fondamentale. Ciò comporta la determinazione se la variabile è in genere distribuita o segue un'altra distribuzione e l'identificazione di eventuali outlier.

Questi vengono spesso definiti statistiche di riepilogo delle variabili numeriche e categoriche.

### Statistiche di riepilogo

Le statistiche di riepilogo sono disponibili per i dataframe apache Spark quando si usa il parametro `summary=True`.

In alternativa, è possibile generare le statistiche di riepilogo usando Python.

```python
import pandas as pd

df = pd.DataFrame({
    'Height_in_cm': [170, 180, 175, 185, 178],
    'Weight_in_kg': [65, 75, 70, 80, 72],
    'Age_in_years': [25, 30, 28, 35, 32]
})

desc_stats = df.describe()
print(desc_stats)
```

## Analisi univariata

L'analisi univariata è la forma più semplice di analisi dei dati in cui i dati analizzati contengono una sola variabile. Lo scopo principale dell'analisi univariata è descrivere i dati e trovare modelli esistenti all'interno di esso.

Si tratta di tracciati comuni usati per eseguire un'analisi univariata.

- **Istogrammi:** Consente di visualizzare la frequenza di ogni categoria di una variabile continua. Possono aiutare a identificare la tendenza centrale, la forma e la diffusione dei dati.
- **Tracciati box:** Usato per visualizzare l'intervallo, l'intervallo interquartile (IQR), la median e i potenziali outlier di una variabile numerica.
- **Grafici a barre:** Sono simili agli istogrammi, ma vengono in genere usati per le variabili categoriche. Ogni barra rappresenta una categoria e l'altezza o la lunghezza della barra corrisponde alla relativa frequenza o proporzione.

Il codice seguente crea un tracciato box e un istogramma usando Python.

```python
import numpy as np
import matplotlib.pyplot as plt

# Let's assume these are the heights of a group in inches
heights_in_inches = [63, 64, 66, 67, 68, 69, 71, 72, 73, 55, 75]

fig, axs = plt.subplots(1, 2, figsize=(10, 5))

# Boxplot
axs[0].boxplot(heights_in_inches, whis=0.5)
axs[0].set_title('Box plot of heights')

# Histogram
bins = range(min(heights_in_inches), max(heights_in_inches) + 5, 5)
axs[1].hist(heights_in_inches, bins=bins, alpha=0.5)
axs[1].set_title('Frequency distribution of heights')
axs[1].set_xlabel('Height (in)')
axs[1].set_ylabel('Frequency')

plt.tight_layout()
plt.show()
```

Queste sono alcune conclusioni che possiamo trarre dai risultati.

- Nel tracciato box, la distribuzione delle altezze è asimmetrica a sinistra, vale a dire che ci sono molti individui con altezze significativamente al di sotto della media.
- Esistono due outlier potenziali: **55 pollici (4'7")** e **75 pollici (6'3")**. Questi valori sono inferiori e superiori al resto dei punti dati.
- La distribuzione delle altezze è approssimativamente simmetrica intorno alla mediano, presupponendo che gli outlier non asimmetrino significativamente la distribuzione.

## Analisi bivariata e multivariata

L'analisi bivariata e multivariata consente di comprendere le relazioni e le interazioni tra variabili diverse in un set di dati e spesso vengono presentate usando grafici a dispersione, matrici di correlazione o tabulazioni incrociate.

### Grafici a dispersione

Il codice seguente usa la `scatter()` funzione di [matplotlib](https://matplotlib.org/) per creare il grafico a dispersione. Si specifica `house_sizes` per l'asse x e `house_prices` per l'asse y.

```python
import matplotlib.pyplot as plt
import numpy as np

# Sample data
np.random.seed(0)  # for reproducibility
house_sizes = np.random.randint(1000, 3000, size=50)  # Size of houses in square feet
house_prices = house_sizes * 100 + np.random.normal(0, 20000, size=50)  # Price of houses in dollars

# Create scatter plot
plt.scatter(house_sizes, house_prices)

# Set plot title and labels
plt.title('House Prices vs Size')
plt.xlabel('Size (sqt)')
plt.ylabel('Price ($)')

# Display the plot
plt.show()
```

In questo grafico a dispersione ogni punto rappresenta una casa. Si noterà che le dimensioni della casa aumentano (spostandosi direttamente lungo l'asse x), il prezzo tende anche ad aumentare (spostandosi lungo l'asse y).

Questo tipo di analisi consente di comprendere in che modo le modifiche nelle variabili dipendenti influiscono sulla variabile di destinazione. Analizzando le relazioni tra queste variabili, è possibile eseguire stime sulla variabile di destinazione in base ai valori delle variabili dipendenti.

Inoltre, questa analisi può aiutare a identificare le variabili dipendenti che hanno un impatto significativo sulla variabile di destinazione. Ciò è utile per la selezione delle funzionalità nei modelli di Machine Learning, in cui l'obiettivo è usare le funzionalità più rilevanti per stimare la destinazione.

### Tracciato a linee

Lo script Python seguente usa la [`matplotlib`](https://matplotlib.org/) libreria per creare un tracciato di linee di prezzi di case simulati per un periodo di tre anni. Genera un elenco di date mensili dal 2020 al 2022 e un elenco corrispondente di prezzi delle case, che iniziano da $200.000 e aumentano ogni mese con una certa casualità.

L'asse x del tracciato viene formattato per visualizzare le date in formato '_Year-Month_' e impostare l'intervallo dei segni di graduazione sull'asse x su ogni sei mesi.

```python
import matplotlib.pyplot as plt
from datetime import datetime, timedelta
import random
import matplotlib.dates as mdates

# Generate monthly dates from 2020 to 2022
dates = [datetime(2020, 1, 1) + timedelta(days=30*i) for i in range(36)]

# Generate corresponding house prices with some randomness
prices = [200000 + 5000*i + random.randint(-5000, 5000) for i in range(36)]

plt.figure(figsize=(10,5))

# Plot data
plt.plot(dates, prices)

# Format x-axis to display dates
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m'))
plt.gca().xaxis.set_major_locator(mdates.MonthLocator(interval=6)) # set interval to 6 months
plt.gcf().autofmt_xdate() # Rotation

# Set plot title and labels
plt.title('House Prices Over Years')
plt.xlabel('Year-Month')
plt.ylabel('House Price ($)')

# Show the plot
plt.show()
```

I tracciati a linee sono semplici da comprendere e leggere. Forniscono una panoramica chiara e generale della progressione dei dati nel tempo, rendendoli una scelta comune per i dati delle serie temporali.

### Tracciato coppia

Un tracciato di coppia può essere utile quando si vuole visualizzare la relazione tra più variabili contemporaneamente.

```python
import seaborn as sns
import pandas as pd

# Sample data
data = {
    'Size': [1000, 2000, 3000, 1500, 1200],
    'Bedrooms': [2, 4, 3, 2, 1],
    'Age': [5, 10, 2, 6, 4],
    'Price': [200000, 400000, 350000, 220000, 150000]
}

df = pd.DataFrame(data)

# Create a pair plot
sns.pairplot(df)
```

In questo modo viene creata una griglia di tracciati in cui ogni caratteristica viene tracciata rispetto a ogni altra caratteristica. Sulla diagonale sono istogrammi che mostrano la distribuzione di ogni caratteristica. I tracciati fuori diagonale sono grafici a dispersione che mostrano la relazione tra due caratteristiche.

Questo tipo di visualizzazione può aiutarci a capire come le diverse caratteristiche sono correlate tra loro e potrebbero potenzialmente essere usate per informare le decisioni relative all'acquisto o alla vendita di case.

