
Gli scienziati dei dati dedicano la maggior parte del tempo al training dei modelli di Machine Learning per identificare modelli complessi nei dati. Dopo il training, l'obiettivo è utilizzare i modelli per **generare stime** su nuovi dati, un processo noto come **inferenza** o **assegnazione dei punteggi** (scoring).

In **Microsoft Fabric**, è possibile operazionalizzare i modelli sottoposti a training per l'assegnazione dei punteggi batch utilizzando l'integrazione nativa con **MLflow**.

## Le Fasi Chiave del Progetto di Data Science

Un progetto di data science segue un ciclo di vita ben definito. L'attenzione qui è sull'uso del modello.

1. **Definire il problema**: Stabilire cosa deve prevedere il modello e i criteri di successo aziendale.
2. **Ottenere i dati**: Reperire le origini dati e archiviare i dati in una **Lakehouse** di Fabric.
3. **Preparare i dati**: Esplorare, pulire e trasformare i dati in un notebook (solitamente usando PySpark) in base ai requisiti del modello.
4. **Eseguire il training del modello**: Scegliere un algoritmo, ottimizzare gli iperparametri e monitorare gli esperimenti con **MLflow**.
5. **Generare stime (Scoring Batch)**: Utilizzare il modello sottoposto a training e salvato per generare stime su un nuovo set di dati.

> **Intuizione**: L'assegnazione dei punteggi batch è l'atto di applicare un modello a un _intero set di dati_ (batch) alla volta, spesso in modo pianificato (es. ogni notte o settimana), a differenza dell'inferenza in tempo reale, che valuta singole richieste immediatamente.

---

# Personalizzare il Comportamento del Modello per il Punteggio Batch

Dopo il training, è essenziale che il modello conosca esattamente la **struttura dei dati in ingresso** e la **struttura dei dati in uscita** che genererà. Queste informazioni sono cruciali per l'operazionalizzazione e sono definite nella **Firma del Modello MLflow**.

## La Firma del Modello MLflow (`MLmodel` file)

La **firma del modello** (Model Signature) è un metadato che descrive lo schema di input e output previsto dal modello. 
Viene archiviata nel file `MLmodel` all'interno della cartella `model` registrata con l'esecuzione dell'esperimento MLflow.

La firma è fondamentale per gli strumenti a valle (come l'utility di assegnazione dei punteggi in Fabric) per:

1. Garantire che i dati di input siano **allineati** al modello.
2. Facilitare la **mappatura delle colonne** durante il punteggio batch.

MLflow supporta due tipi principali di firma:

|Tipo di Firma|Ideale Per|Struttura dei Dati|
|---|---|---|
|**Basata su Colonne**|Dati tabulari (es. DataFrame Pandas/Spark)|Sequenza di colonne nominate con tipo di dato MLflow (es. `integer`, `double`, `string`). **Raccomandata per Fabric Data Science.**|
|**Basata su Tensori**|Dati deep learning (es. immagini, audio)|Sequenza di tensori definiti da `dtype` (tipo di dato NumPy) e `shape`.|

### Creazione di una Firma del Modello Basata su Colonne

Se l'**Autologging di MLflow** non deduce automaticamente la firma in modo ottimale (spesso deducendo un input generico come tensore), è possibile **definirla manualmente** e registrarla:

```Python
from mlflow.models.signature import ModelSignature
from mlflow.types.schema import Schema, ColSpec
import mlflow
from sklearn.tree import DecisionTreeRegressor

# 1. Definizione dello schema di input
input_schema = Schema([
    ColSpec("integer", "AGE"),
    ColSpec("double", "BMI"),
    # ... altre colonne di input
    ColSpec("integer", "S6"),
])

# 2. Definizione dello schema di output (la variabile target)
output_schema = Schema([ColSpec("double", "prediction")]) 
# Usa il tipo di dato previsto per l'output (es. double per la regressione)
# ed è buona prassi dare un nome esplicito.

# 3. Creazione dell'oggetto firma
signature = ModelSignature(inputs=input_schema, outputs=output_schema)

with mlflow.start_run():
    # Disabilita l'autologging dei modelli per loggarlo manualmente con la firma
    mlflow.autolog(log_models=False) 

    model = DecisionTreeRegressor(max_depth=5)
    model.fit(X_train, y_train)

    # 4. Registrazione manuale del modello con la firma definita
    # La cartella del modello (es. "model") e il file MLmodel vengono creati
    mlflow.sklearn.log_model(
        sk_model=model, 
        artifact_path="model", 
        signature=signature
    )
```

## Salvare il Modello nell'Area di Lavoro

Un modello tracciato (tracked) è registrato nell'esecuzione dell'esperimento. Per renderlo disponibile per l'assegnazione dei punteggi, deve essere **salvato (o registrato)** nell'**Area di Lavoro** di Microsoft Fabric (nel Model Registry).

Per salvare un modello:

- **Interfaccia Utente**: Accedere all'esecuzione dell'esperimento in Fabric e selezionare l'opzione per **Salvare il Modello**.
- **Tramite Codice**: Utilizzare le API MLflow per registrare il modello, specificando l'URI che punta alla cartella `model` dell'esecuzione completata.

```Python
# Passaggi per trovare l'ultima esecuzione (run) e registrare il modello
exp = mlflow.get_experiment_by_name(experiment_name)
last_run = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=1)
last_run_id = last_run.iloc[0]["run_id"]

# URI che punta alla cartella 'model' all'interno dell'esecuzione MLflow
model_uri = f"runs:/{last_run_id}/model" 

# Registra il modello nell'Area di Lavoro sotto un nome specifico
mv = mlflow.register_model(model_uri, "diabetes-model") 
# "diabetes-model" sarà il nome del modello nel Model Registry
```

---

# Preparazione dei Dati e Coerenza dello Schema

La coerenza dello schema dei dati di input con la firma del modello è l'aspetto più critico per un'assegnazione dei punteggi batch di successo.

## Utilizzo dei Dati in Tabelle Delta

In Microsoft Fabric, i dati su cui si desidera eseguire l'assegnazione dei punteggi (e le stime risultanti) devono essere archiviati come **Tabelle Delta** in una Lakehouse.

- **Scrittura in Delta**:
```Python
df.write.format("delta").mode("overwrite").save(f"Tables/new_data_for_scoring")
```
    
- **Lettura da Delta**:
```Python
df = spark.read.format("delta").load(f"Tables/new_data_for_scoring")
```


>[!Nota]
>Il percorso delle tabelle Delta è spesso nel formato _`abfss://<id-storage>@msitonelake.dfs.fabric.microsoft.com/<id-workspace>/Tables/nome_tabella`_, ma nell'ambiente di un notebook
> Fabric è sufficiente il percorso relativo **`Tables/nome_tabella`** se la Lakehouse è allegata.


## Coerenza dei Tipi di Dato (Schema Enforcement)

Quando si utilizza una firma del modello basata su colonne, i dati di input devono avere gli stessi nomi di colonna e tipi di dato definiti.

### Mappatura Tipi di Dato

È importante allineare i tipi di dato MLflow (usati nella firma) con i tipi di dato **Spark/PySpark** (usati nel DataFrame per il punteggio).

|Tipo di Dato MLflow|Tipo di Dato PySpark Equivalente (Esempi)|
|---|---|
|`Integer` (32b)|`IntegerType()`|
|`Long` (64b)|`LongType()`|
|`Float` (32b)|`FloatType()`|
|`Double` (64b)|`DoubleType()`|
|`String`|`StringType()`|
|`Boolean`|`BooleanType()`|

### Impostare i Tipi di Dato in PySpark

Prima di eseguire l'inferenza, ispezionare e modificare i tipi di dato del DataFrame Spark se necessario:

```Python
# 1. Visualizzare i tipi di dato attuali
df.dtypes 

# 2. Modificare il tipo di dato di colonne specifiche
from pyspark.sql.types import IntegerType, DoubleType

df = df.withColumn("S1", df["S1"].cast(IntegerType()))
df = df.withColumn("BMI", df["BMI"].cast(DoubleType())) 
# È fondamentale che lo schema del DataFrame corrisponda esattamente alla firma del modello MLflow.
```

---

# Generare Stime con la Funzione PREDICT

Microsoft Fabric fornisce la funzione scalabile **PREDICT** per l'assegnazione dei punteggi batch. Questa funzione può essere utilizzata in diversi modi, ma l'approccio principale nei notebook è tramite l'oggetto **MLFlowTransformer** di `synapse.ml.predict`.

## L'Oggetto `MLFlowTransformer`

`MLFlowTransformer` agisce da **wrapper** attorno al modello MLflow registrato, consentendo di applicarlo a un DataFrame Spark. Fa parte della libreria **SynapseML** (integrata in Fabric).

Per istanziare `MLFlowTransformer` sono necessari i seguenti parametri:

- `inputCols`: Elenco delle colonne del DataFrame da passare al modello.
- `outputCol`: Nome della nuova colonna che conterrà le stime.
- `modelName`: Nome del modello salvato nel Model Registry di Fabric.
- `modelVersion`: Versione specifica del modello da utilizzare.

### Esecuzione del Punteggio Batch

```Python
from synapse.ml.predict import MLFlowTransformer

# 1. Creazione dell'oggetto Transformer
model_transformer = MLFlowTransformer(
    inputCols=["AGE","SEX","BMI","BP","S1","S2","S3","S4","S5","S6"], # Le feature del modello
    outputCol='predictions', # Nome della colonna di output
    modelName='diabetes-model', # Nome del modello registrato in Fabric
    modelVersion=1 # Versione specifica (sempre meglio specificarla)
)

# 2. Applicazione della trasformazione al DataFrame
df_predictions = model_transformer.transform(df)

# Visualizzazione delle prime righe (incluse le stime)
df_predictions.show()
```

>[!Nota]
>**Dettagli Tecnici**: `MLFlowTransformer` gestisce automaticamente il caricamento del modello e l'applicazione in modo distribuito sul cluster Spark, garantendo l'efficienza nell'assegnazione dei punteggi batch su set di dati di grandi dimensioni.

## Salvare i Risultati del Punteggio

Dopo aver generato le stime (`df_predictions`), è necessario salvarle nuovamente in una **Tabella Delta** per l'analisi o l'utilizzo a valle (es. report Power BI).

```Python
# Salvare il DataFrame con le stime in una nuova tabella Delta
df_predictions.write.format("delta").mode("overwrite").save(f"Tables/diabetes_predictions")

# O, per accodare i risultati ad una tabella esistente (se lo schema è compatibile)
# df_predictions.write.format("delta").mode("append").save(f"Tables/existing_predictions_log")
```

>[!Nota] 
>**Procedura Guidata**: Microsoft Fabric fornisce una **procedura guidata** (accessibile dalla pagina del modello salvato) che
>semplifica questi passaggi:
> 
> 1. Selezione della tabella di input.
> 2. Mappatura delle colonne di input.
> 3. Definizione della tabella e della colonna di output.
> 4. Generazione automatica del codice PySpark con `MLFlowTransformer`. Questo è un ottimo punto di partenza e una verifica visiva per la mappatura delle colonne.

