---
autore: Kevin Milli
---

Microsoft Fabric offre un ambiente unificato, basato su **Apache Spark**, per l'intero ciclo di vita del Machine Learning (ML), dall'esplorazione dei dati al deployment del modello. L'approccio centrale per l'addestramento dei modelli si basa sui **Notebook** integrati e sull'uso di **MLflow** per la tracciabilità e la gestione degli esperimenti.

---

## Framework e Strumenti di Machine Learning in Fabric

I data scientist lavorano prevalentemente in **Python**, e Fabric supporta nativamente i framework ML più diffusi.

|Framework|Scopo Principale|Applicazioni Tipiche|Intuizione Avanzata|
|---|---|---|---|
|**Scikit-learn**|ML Tradizionale (Classificazione, Regressione, Clustering)|Previsione di base, analisi dei dati tabellari|Ottimo per la fase iniziale (baseline) e problemi con dati strutturati. Implementazioni efficienti e API coerente.|
|**PyTorch / TensorFlow**|Deep Learning (DL)|Visione artificiale, Elaborazione del Linguaggio Naturale (NLP)|Essenziali per reti neurali complesse. **PyTorch** è noto per la sua natura "eager" e la facilità di debug; **TensorFlow** per l'ecosistema di produzione.|
|**SynapseML (SML)**|ML Scalabile e Distribuito|Pipeline ML su larga scala, Integrazione con servizi cognitivi|Estende le capacità di Spark (PySpark) per il ML. Permette di creare pipeline ML **estremamente scalabili** e distribuite. Utile quando i dati non entrano nella memoria di una singola macchina.|
|**PySpark**|Elaborazione Dati Distribuita|Ingegneria delle Funzionalità (Feature Engineering) su Big Data|La base computazionale dei Notebook Fabric. Permette di manipolare e preparare set di dati di dimensioni massicce prima di passare l'output a framework ML tradizionali.|

### Notebook in Microsoft Fabric

I Notebook sono l'interfaccia principale per il training e sono simili ai **Jupyter Notebook**.

- **Base Computazionale:** Sono eseguiti su un cluster **Spark**, quindi l'ambiente di default è **PySpark** e **Python**.
- **Compatibilità:** La maggior parte dei framework ML (scikit-learn, PyTorch, TensorFlow) lavora con i **Pandas Dataframe**. È necessario convertire i dati Spark (Dataframe PySpark) in Dataframe Pandas quando si utilizzano librerie non native per la distribuzione.

---

## Workflow Standard per il Training di un Modello

Il training di un modello ML tradizionale segue un processo iterativo e ben definito.

### 1. Caricamento e Esplorazione dei Dati

- **Caricamento:** Rendere i dati disponibili nel Notebook, tipicamente come un DataFrame Pandas o Spark.
- **Esplorazione (EDA - Exploratory Data Analysis):** Comprendere la relazione tra le **funzionalità (features)** (input) e **l'etichetta (label)** (output) tramite visualizzazioni e statistiche.

### 2. Preparazione dei Dati (Feature Engineering)

- Trasformazione dei dati grezzi in funzionalità adatte al modello (es. **normalizzazione/standardizzazione**, codifica di variabili categoriche, gestione dei valori mancanti).

### 3. Suddivisione dei Dati

Dividere i dati in set di **Training** e **Test** (spesso anche **Validazione** per l'ottimizzazione degli iperparametri).
**Esempio Pratico (Scikit-learn):**

```Python
from sklearn.model_selection import train_test_split

# X sono le feature, y è l'etichetta
X, y = df[['feature1','feature2','feature3']].values, df['label'].values

# Split 70% Training, 30% Test, con seed per riproducibilità
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=42) # random_state cambiato per una convenzione più comune
```

- **Output:** `X_train`, `X_test` (solo feature), `y_train`, `y_test` (solo etichette).

### 4. Addestramento del Modello

Selezionare un algoritmo e addestrarlo sui dati di training (`X_train`, `y_train`).
**Esempio Pratico (Regressione Lineare - Scikit-learn):**

```Python
from sklearn.linear_model import LinearRegression

model = LinearRegression() 
model.fit(X_train, y_train) # Il modello "impara" dai dati di training
```

### 5. Valutazione

Generare previsioni sul set di test (`X_test`) e calcolare le **metriche di prestazione** per la regressione, Accuracy per la classificazione).

- **Intuizione:** L'obiettivo non è massimizzare le prestazioni sul set di training (rischio di **overfitting**), ma ottenere una buona generalizzazione sul set di test.

---

## Tracciamento e Riproducibilità con MLflow

**MLflow** è una libreria open source cruciale per l'intero ciclo di vita del ML, ed è **preinstallata** e **integrata** in Microsoft Fabric. Il suo componente principale è **MLflow Tracking**, che garantisce la **riproducibilità** del lavoro.

### Concetti Chiave di MLflow Tracking

|Concetto|Descrizione|Corrispondenza in Fabric|
|---|---|---|
|**Esperimento (Experiment)**|L'unità di organizzazione primaria. Raggruppa le diverse esecuzioni di un progetto.|Oggetto di prima classe nell'area di lavoro Fabric.|
|**Esecuzione (Run)**|Una singola esecuzione di codice di training, registrata con i suoi parametri, metriche e artefatti.|Corrisponde a un singolo tentativo di training di un modello.|
|**Artefatti (Artifacts)**|File di output (il modello serializzato, grafici, file di ambiente).|Salvati nel percorso associato all'esecuzione.|

### Uso di MLflow nel Notebook

#### 1. Creare/Impostare l'Esperimento

Si definisce l'esperimento attivo. Se non esiste, viene creato.

```Python
import mlflow

# Imposta l'esperimento come attivo.
mlflow.set_experiment("Nome_Mio_Esperimento_Avanzato")
```

#### 2. Registrazione Personalizzata

Per un controllo granulare su cosa viene tracciato, si usa un blocco `with mlflow.start_run():`.

```Python
import mlflow
from xgboost import XGBClassifier
from sklearn.metrics import accuracy_score

# Avvia una nuova 'Run' all'interno dell'esperimento attivo
with mlflow.start_run(run_name="Run_XGBoost_Custom_01"): 
    # Log di un parametro specifico
    mlflow.log_param("num_boost_round", 100)
    
    # Training del modello
    model = XGBClassifier(n_estimators=100, use_label_encoder=False, eval_metric="logloss")
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    
    # Calcolo e Log della metrica
    accuracy = accuracy_score(y_test, y_pred)
    mlflow.log_metric("accuracy", accuracy) # Registra la metrica 'accuracy'
    
    # Registrazione del modello in formato MLflow (include metadati, ambiente, e artefatti)
    mlflow.sklearn.log_model(model, "xgboost_model") 
    
# L'esecuzione si conclude automaticamente uscendo dal blocco 'with'
```

- **Intuizione:** La registrazione sotto `start_run()` assicura che parametri, metriche e artefatti siano collegati in modo univoco a quell'esecuzione.

#### 3. Recupero delle Esecuzioni (Recupero delle Metriche)

È possibile recuperare i dati delle esecuzioni per il confronto e l'analisi.

```Python
# Recupera l'oggetto Esperimento tramite il nome
exp = mlflow.get_experiment_by_name("Nome_Mio_Esperimento_Avanzato")

# Cerca le ultime 5 esecuzioni, ordinate per la metrica 'accuracy' in ordine decrescente
best_runs = mlflow.search_runs(
    exp.experiment_id, 
    order_by=["metrics.accuracy DESC"], 
    max_results=5
)

print(best_runs[['run_id', 'metrics.accuracy', 'params.num_boost_round']])
```

- **Filtri Avanzati:** È possibile filtrare le esecuzioni in base a parametri o metriche specifiche (es. `filter_string="params.num_boost_round='100' AND metrics.accuracy > 0.8"`).

---

## Gestione del Modello con il Formato MLmodel

Dopo il training, l'obiettivo è rendere il modello utilizzabile (**deployment** o **scoring**). MLflow standardizza questo processo con il formato **MLmodel**.

### Il File `MLmodel`

Quando si registra un modello con `mlflow.log_model()`, viene creata una cartella (tipicamente chiamata `model/`) che contiene tutti gli artefatti necessari, incluso il file cruciale **`MLmodel`**.

- **Scopo:** Agisce come un **manifesto** per il modello, specificando come caricarlo e come utilizzarlo. È l'unica fonte di verità.
- **Contenuto Chiave:**
    - `artifact_path`: Percorso dove si trova il modello serializzato (es. `model.pkl`).
    - `flavor`: La libreria ML usata (es. `sklearn`, `tensorflow`). Specifica la modalità di caricamento.
    - `run_id`: L'ID dell'esecuzione che ha prodotto il modello (per la tracciabilità).
    - `signature`: **Schema del modello**. Definisce in modo rigido gli **input** e gli **output** attesi (tipi di dati e forme delle strutture dati), essenziale per l'integrità del deployment.

|Artefatto del Modello|Scopo|
|---|---|
|**`model.pkl`** (o equivalente)|Il modello addestrato serializzato (il "cervello" del modello).|
|**`MLmodel`**|Metadati e istruzioni per il caricamento.|
|**`conda.yaml` / `requirements.txt`**|Definiscono l'ambiente software (librerie e versioni) necessario per eseguire il modello in qualsiasi ambiente, garantendo la **compatibilità**.|

### Registrazione e Controllo delle Versioni in Fabric

L'integrazione con MLflow consente di **salvare il modello** direttamente dall'interfaccia utente dell'Esperimento o tramite codice nella **Registro Modelli (Model Registry)** di Fabric.

- **Registro Modelli:** Un repository centralizzato per la gestione dei modelli pronti per la produzione.
- **Controllo delle Versioni:** Quando si salva un modello esistente, si crea una **nuova versione**. Questo è cruciale per confrontare le prestazioni (es. $V_2$ e meglio di $V_1$ ) e per il rollback in caso di problemi di produzione. Ogni versione eredita gli artefatti e i metadati dell'esecuzione MLflow che l'ha generata.