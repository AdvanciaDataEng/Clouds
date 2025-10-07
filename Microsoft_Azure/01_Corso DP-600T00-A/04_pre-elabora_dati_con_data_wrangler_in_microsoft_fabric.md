
## Data Wrangler: Strumento per l'Esplorazione e la Pre-elaborazione

**Data Wrangler** è uno strumento di **Microsoft Fabric** integrato nei **notebook** che fornisce un'interfaccia grafica intuitiva per l'esplorazione e la pre-elaborazione dei dati.

### Funzionalità e Ruolo nel Workflow

Data Wrangler è essenzialmente un generatore di codice: le operazioni grafiche che esegui vengono tradotte automaticamente in codice Python riutilizzabile (**Pandas** o **PySpark**, supportando l'accelerazione con Spark per set di dati più grandi), che può essere salvato nel notebook.

|Funzionalità|Descrizione|Intuizione per lo Studio Avanzato|
|---|---|---|
|**Esplorazione Dati (EDA)**|Visualizzazione in griglia e statistiche di riepilogo dinamiche (istogrammi, conteggi, min/max).|Permette di identificare rapidamente **distribuzioni (es. non normali)**, **valori mancanti** e **outlier**, cruciali per la scelta del modello.|
|**Pulizia Dati**|Libreria di operazioni per gestire valori mancanti, duplicati, e tipi di dati errati.|Migliora la **qualità dei dati**, che è il fattore primario per l'accuratezza del modello. Un _garbage in, garbage out_!|
|**Feature Engineering**|Creazione di nuove _feature_ e trasformazione di quelle esistenti (es. codifica one-hot, ridimensionamento).|Facilita la **creazione di rappresentazioni dei dati più significative** per l'algoritmo di Machine Learning.|

### Vantaggi

- **Aumento dell'Efficienza:** Velocizza attività noiose e ripetitive come la pulizia dei dati.
- **Ripetibilità:** Il codice generato garantisce che le pipeline di pre-elaborazione siano riproducibili e facili da automatizzare.
- **Accessibilità:** Utile sia per gli sviluppatori esperti che vogliono accelerare, sia per i principianti che possono apprendere il codice Python/Spark generato.

---

## Avviare e Utilizzare Data Wrangler

### Procedura di Avvio

1. Assicurati di essere nell'esperienza **Data Science** di Microsoft Fabric.
2. Crea un nuovo **Notebook**.
3. Carica i dati in un DataFrame, ad esempio utilizzando `pandas`:
```Python
import pandas as pd
df = pd.read_csv("https://raw.githubusercontent.com/plotly/datasets/master/titanic.csv")
```
4. Seleziona **Dati** nella barra multifunzione del notebook e poi **Avvia Data Wrangler**, scegliendo il DataFrame desiderato (`df`).

### Collaborazione con gli Operatori

Data Wrangler offre una vasta gamma di operatori, raggruppati per categorie, che si applicano al DataFrame. Ogni operazione aggiorna la visualizzazione in tempo reale, mostrando i dati modificati in verde e quelli rimossi in rosso.

|Categoria Operatore|Esempi di Operazioni Chiave|Applicazione Pratica (Insights)|
|---|---|---|
|**Trova e Sostituisci**|Eliminare righe duplicate, gestire i valori mancanti (es. eliminazione, imputazione con media/mediana).|Essenziale per la **Data Cleansing**; l'imputazione è cruciale per non perdere troppe osservazioni.|
|**Formato**|Conversione in maiuscolo/minuscolo, rimozione di spazi bianchi, suddivisione del testo.|Uniforma i dati testuali, prevenendo problemi di corrispondenza causati da differenze di formattazione.|
|**Formule**|Creazione di nuove colonne tramite formule Python personalizzate, **Codifica One-Hot**, **Binarizer Multi-Etichetta**.|Funzionalità fondamentali di **Feature Engineering** per convertire variabili categoriche in un formato numerico che i modelli ML possono comprendere.|
|**Numerico**|Arrotondamento, **Ridimensionamento Min/Max (Normalization)**.|Il ridimensionamento è vitale per algoritmi basati sulla distanza (es. K-NN, K-Means) o per le reti neurali, assicurando che nessuna _feature_ domini a causa della sua scala.|
|**Schema**|Modifica del tipo di colonna, eliminazione/rinomina/clonazione di colonne.|Mantiene lo schema del DataFrame corretto (es. convertire una colonna numerica da `object` a `int/float`).|
|**Ordinare e Filtrare**|Filtrare righe in base a condizioni (es. rimuovere outlier), ordinare i dati.|Utile per l'analisi e per la preparazione di subset specifici.|
|**Altri**|**Raggruppamento e Aggregazione**, **Microsoft Flash Fill** per la creazione automatica di colonne da esempi.|Il raggruppamento (es. `groupby()`) è fondamentale per creare statistiche riepilogative (media, somma, conteggio) di gruppi di dati (es. _Prezzo Medio per Tipo di Casa_).|

### Generazione e Integrazione del Codice

Dopo aver applicato gli operatori desiderati, l'azione **+ Aggiungi codice al notebook** genera una **funzione Python** completa nel notebook. 
Questo codice non modifica il DataFrame originale finché non viene eseguito manualmente, garantendo un controllo completo sul flusso di lavoro di pre-elaborazione.

```Python
# Esempio di codice generato per l'aggregazione
def clean_data(df):
    # Passaggio 1: Raggruppa per 'Type' e calcola la media di 'Price'
    df_transformed = df.groupby('Type').agg(AvgPrice=('Price', 'mean')).reset_index()
    return df_transformed

# Esecuzione della funzione nel notebook
df_pulito = clean_data(df)
```

Questa funzionalità crea una **pipeline di dati** riutilizzabile che può essere applicata a nuovi set di dati, accelerando notevolmente il ciclo di vita del Machine Learning.
È inoltre possibile esplorare come usare [Data Wrangler in Microsoft Fabric (un'interfaccia immersiva per l'analisi esplorativa dei dati)](https://www.youtube.com/watch?v=EzSStWnyLkc).
Questo video illustra in dettaglio come Data Wrangler possa essere utilizzato come interfaccia immersiva per l'analisi esplorativa dei dati in Microsoft Fabric.

---

# Data Wrangling in Microsoft Azure: Gestione Dati Mancanti e Trasformazioni Essenziali

La **pre-elaborazione dei dati** è una fase cruciale nel Machine Learning, volta a trasformare i dati grezzi in un formato pulito e adatto all'addestramento del modello. 
**Azure Data Wrangler** in Microsoft Fabric semplifica questo processo, offrendo strumenti interattivi e generazione di codice per la gestione dei dati mancanti e la trasformazione delle variabili.

## 1. Gestire i Dati Mancanti (Missing Data)

I **dati mancanti** (Missing Values) si riferiscono all'assenza di un valore in una variabile o osservazione. La loro corretta gestione è vitale, poiché possono distorcere l'analisi e l'addestramento del modello.

### Identificazione dei Dati Mancanti

In **Data Wrangler** (all'interno di un notebook di Microsoft Fabric), l'identificazione è visuale e basata su statistiche:

1. **Intestazione Colonna:** Mostra direttamente il **numero** e la **percentuale** di valori mancanti per ogni variabile, offrendo un'immediata panoramica.
2. **Pannello di Riepilogo:** Fornisce statistiche dettagliate, inclusi i conteggi esatti dei valori mancanti per la colonna selezionata.
3. **Operatore di Filtro:** Permette di visualizzare e isolare rapidamente le righe che contengono valori mancanti per una data colonna.

> **Intuizione:** È fondamentale capire il **meccanismo di mancanza** (Missingness Mechanism). I dati mancanti si suddividono in:
> 
>**MCAR** (Missing Completely At Random): La mancanza non dipende né dal valore della variabile mancante né da altre variabili del dataset. 
>L'eliminazione è spesso l'opzione più sicura, se la percentuale è bassa (<5%).
>
>**MAR** (Missing At Random): La mancanza dipende da altre variabili osservate nel dataset (es. uomini che tendono a non rispondere a un sondaggio sul reddito). 
>In questo caso, metodi di imputazione basati su modelli (come KNN o Regressione) sono più appropriati.
>
>**MNAR** (Missing Not At Random): La mancanza dipende dal valore stesso che sarebbe mancante (es. persone con reddito molto alto o basso che tendono a non indicarlo). Questo è il caso più problematico e spesso richiede la modellazione esplicita del meccanismo di mancanza.
>

### Strategie di Gestione

Le principali strategie per affrontare i dati mancanti sono:

1. **Ignorare:** Se la quantità è minima e la mancanza è **MCAR**, l'impatto sul modello può essere trascurabile.
2. **Rimozione (Listwise Deletion):**
    - **Righe:** Eliminare le righe con valori mancanti (usando **Trova e sostituisci** > **Eliminare i valori mancanti** in Data Wrangler). È appropriato solo se la percentuale di dati persi è molto bassa, altrimenti si rischia un **Bias** significativo.
    - **Colonne:** Rimuovere intere colonne se contengono troppi valori mancanti.
3. **Imputazione:** Sostituire i valori mancanti con una stima.

#### Imputazione in Dettaglio

|Metodo|Descrizione|Tipo di Dati/Scenario Ideale|
|---|---|---|
|**Media**|Sostituisce con la media aritmetica della variabile.|Dati **continui** con distribuzione simmetrica, **senza Outlier**.|
|**Mediana**|Sostituisce con il valore mediano (il 50° percentile).|Dati **continui**, **robusto** in presenza di **Outlier**.|
|**Moda**|Sostituisce con il valore più frequente.|Dati **categoriali** o **numerici discreti**.|
|**Propaga Avanti (Forward Fill)**|Usa il valore valido **precedente** (_Pad/Forward Filling_).|Dati **Serie Storiche** (dove l'ordine temporale è rilevante).|
|**Propaga Indietro (Backward Fill)**|Usa il valore valido **successivo** (_Back Filling_).|Dati **Serie Storiche**.|
|**Valore Personalizzato**|Sostituisce con una costante specifica (es. 0, -1, 'Sconosciuto').|Può essere usato per creare una **nuova _feature_** indicando la mancanza.|
|**KNN Imputation**|Stima il valore mancante usando la media ponderata dei ![](data:,) **vicini più prossimi** (basato sulla distanza).|Metodo **avanzato**, più accurato in presenza di correlazioni complesse.|
|**Imputazione di Regressione**|Addestra un modello di regressione per prevedere il valore mancante in funzione delle altre variabili.|Metodo **avanzato**, se esiste una forte relazione lineare.|

**Data Wrangler** facilita l'imputazione (es. con la Mediana) tramite l'operatore **Trova e sostituisci** > **Compilare i valori mancanti**, con la possibilità di generare codice Python (Pandas) nel notebook.

4. **Usare la Mancanza come Caratteristica:** Creare una nuova variabile binaria (es. `OriginalCol_Is_Missing`) che indichi se il valore originale era mancante. Questo è utile se la mancanza è **MNAR**.

---

## 2. Trasformazione delle Variabili (Feature Engineering)

La trasformazione delle variabili è fondamentale per adattare i dati ai requisiti degli algoritmi di Machine Learning e migliorare le loro prestazioni.

### Codifica delle Variabili Categoriali

Molti modelli richiedono input numerici. I dati categoriali devono essere convertiti.

#### Codifica One-Hot (One-Hot Encoding)

Usata quando la variabile categoriale **NON** ha un ordine intrinseco (**Variabili Nominali**).

|Concetto|Spiegazione|
|---|---|
|**Logica**|Crea una nuova colonna binaria (0 o 1) per **ogni categoria univoca**. Solo la colonna corrispondente alla categoria effettiva avrà valore **1**.|
|**Esempio**|Variabile `Pet` (_dog_, _cat_, _bird_) diventa tre colonne: `Pet_dog`, `Pet_cat`, `Pet_bird`. Un'osservazione con _dog_ sarà: `[1, 0, 0]`.|
|**Data Wrangler**|Operatore **Formule** > **Codifica one-hot**.|
|**Attenzione**|Rischio di **Aumento della Dimensionalità** se la variabile ha molte categorie univoche. Inoltre, introduce **Multicollinearità** (una colonna è ridondante, la **Dummy Variable Trap**); è prassi comune ometterne una colonna (N-1) o usare la codifica con variabile fittizia.|

#### Binarizzatore Multi-Etichetta (Multi-Label Binarizer)

Usato quando una singola osservazione può appartenere a **più categorie** contemporaneamente.

|Concetto|Spiegazione|
|---|---|
|**Logica**|La colonna originale è una stringa di categorie separate da un **delimitatore** (es. `|
|**Esempio**|Variabile `Category` (*Italian|
|**Data Wrangler**|Operatore **Formule** > **binarizer con più etichette**, specificando il **delimitatore** (es. `|
|**Applicazione**|Set di dati con tag, generi di film, o categorie multiple di prodotti.|

### Ridimensionamento (Scaling) delle Variabili Numeriche

Il ridimensionamento è essenziale per gli algoritmi basati sulla **distanza** (come KNN, K-Means, SVM) o sulla **discesa del gradiente** (come Reti Neurali, Regressione Logistica/Lineare), poiché impedisce che variabili con scale più ampie dominino la funzione di costo.

#### Ridimensionamento Min-Max (Min-Max Scaling / Normalizzazione)

| Concetto          | Spiegazione                                                                                                             |     |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------- | --- |
| **Formula**       | Trasforma un valore  nell'intervallo usando:                                                                            |     |
| **Obiettivo**     | Ridimensiona l'intervallo dei dati in un range predefinito, tipicamente.                                                |     |
| **Data Wrangler** | Operatore **Numeric** > **Scale min/max values**.                                                                       |     |
| **Vantaggio**     | Preserva la forma della distribuzione originale e i rapporti tra i valori.                                              |     |
| **Svantaggio**    | **Non è robusto agli Outlier**; un valore estremo può comprimere tutti gli altri dati in un intervallo molto ristretto. |     |

> Contrasto: Standardizzazione (Z-Score Scaling)
> 
> Un'alternativa comune è la Standardizzazione, che trasforma i dati in modo che abbiano una media $\mu=0$ e deviazione standard $\sigma=1$. È più robusta agli outlier rispetto al Min-Max Scaling ed è spesso preferita negli algoritmi che assumono una distribuzione normale (es. Regressione Lineare).

Questo video della Microsoft spiega come usare Data Wrangler in Microsoft Fabric per velocizzare la preparazione dei dati: [Accelerate data prep with Data Wrangler in Microsoft Fabric](https://www.google.com/search?q=https://www.youtube.com/watch%3Fv%3D38qLh18W0lE).
