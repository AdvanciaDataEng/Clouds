---
autore: Kevin Milli
---

La **Data Science** è la disciplina che combina **matematica**, **statistica** e **ingegneria informatica** per estrarre _insight_ significativi da dati complessi, guidando decisioni informate. In ambito aziendale, questo si traduce spesso nella creazione di modelli di **Intelligenza Artificiale (IA)**, in particolare **Machine Learning (ML)**.

Microsoft Fabric è una piattaforma di analisi end-to-end che unifica le fasi del ciclo di vita dei dati e dei progetti di Data Science.
## Comprendere il Processo di Data Science

Un progetto di Data Science è un processo iterativo e collaborativo, focalizzato sulla risoluzione di un problema aziendale.

### Modelli Comuni di Machine Learning

L'obiettivo del Machine Learning è addestrare modelli che identificano **modelli complessi** (patterns) in grandi volumi di dati per generare **previsioni** o **insight** utili. I quattro tipi fondamentali di modelli ML sono:

1. **Classificazione:** Prevede un **valore categorico** (es. _Sì/No_, _Alto/Medio/Basso_).
    - _Esempio:_ Prevedere se un cliente farà _churn_ (abbandonerà il servizio) in futuro.
2. **Regressione:** Prevede un **valore numerico continuo** (es. prezzo, quantità, temperatura).
    - _Esempio:_ Stimare il prezzo di vendita di un immobile.
3. **Clustering:** Raggruppa **punti dati simili** in cluster o gruppi. È una tecnica di apprendimento _non supervisionato_.
    - _Esempio:_ Segmentare i clienti per indirizzare meglio le offerte personalizzate.
4. **Previsione (Forecasting):** Prevede **valori numerici futuri** basati su dati di **serie temporali**.
    - _Esempio:_ Stimare le vendite previste per il mese successivo.

### Fasi del Processo di Data Science in Fabric

1. **Definire il Problema:** Collaborare con _business users_ per stabilire cosa il modello deve prevedere e definire i **criteri di successo** (KPI).
2. **Ottenere i Dati (Ingestion):** Identificare le sorgenti dati (locali o cloud, es. Azure Data Lake Storage Gen2) e caricarli in una **Lakehouse** di Fabric, che funge da _posizione centrale_ per l'archiviazione di dati strutturati, semi-strutturati e non strutturati.
3. **Preparare i Dati (Exploration & Wrangling):** Esplorare, pulire e trasformare i dati in un formato adatto al training del modello. Questa è spesso la fase che richiede più tempo.
4. **Eseguire il Training del Modello:** Selezionare un algoritmo (es. `scikit-learn`, `PyTorch`) e ottimizzare gli **iperparametri**, tracciando il lavoro con **MLflow**.
5. **Generare Insight (Scoring & Operationalization):** Usare il modello addestrato per generare previsioni su nuovi dati (**assegnazione dei punteggi batch**) e integrare i risultati in report di **Power BI**.

## Esplorare ed Elaborare i Dati con Microsoft Fabric

Microsoft Fabric sfrutta la potenza di **Apache Spark** e offre strumenti integrati per l'elaborazione dei dati su larga scala.

### Notebooks e Spark

- I **Notebooks** in Fabric offrono un'esperienza familiare (simili a Jupyter) basata sul _calcolo Spark_.
- **Apache Spark** è un _framework di elaborazione parallela_ open source, essenziale per l'analisi e l'elaborazione di dati su larga scala.
- Si possono utilizzare linguaggi come **PySpark (Python)** o **SparkR (R)**, sfruttando librerie come **Pandas** e **Numpy** per la preparazione dei dati.

### Data Wrangler: Pulizia Dati Visuale

**Data Wrangler** è uno strumento immersivo basato su notebook che facilita l'**analisi esplorativa dei dati (EDA)** e la **preparazione dei dati** con un'interfaccia grafica.

|Funzionalità Avanzate di Data Wrangler|Descrizione|
|---|---|
|**Anteprima in Tempo Reale**|Mostra immediatamente l'effetto di un'operazione di pulizia o trasformazione prima dell'applicazione effettiva.|
|**Generazione Automatica di Codice**|Ogni operazione selezionata (es. gestione valori mancanti, codifica _One-Hot_) genera automaticamente il codice **Python** (o **PySpark**) corrispondente, che può essere esportato nel notebook.|
|**Statistiche Riassuntive**|Fornisce una panoramica descrittiva dettagliata e statistiche specifiche per colonna, inclusi valori mancanti e distribuzioni, per identificare problemi di qualità dei dati.|
|**Integrazione AI (Anteprima)**|Utilizza **Copilot** per generare codice di trasformazione da descrizioni in linguaggio naturale e offre funzioni AI integrate per arricchimenti (es. sentiment analysis).|

## Eseguire il Training e Assegnare Punteggi con Microsoft Fabric

Il training del modello è un processo di **sperimentazione iterativa** che richiede un tracciamento rigoroso per confrontare le prestazioni.

### Gestione degli Esperimenti con MLflow

**MLflow** è una piattaforma open-source nativamente integrata in Fabric per la gestione del ciclo di vita del Machine Learning (_ML lifecycle_).

|Concetto MLflow|Descrizione|
|---|---|
|**Esperimento (Experiment)**|Un contenitore logico per raggruppare le esecuzioni correlate (Runs). Corrisponde a un obiettivo di modellazione (es. "Previsione Vendite").|
|**Esecuzione (Run)**|Una singola esecuzione del codice di training, che rappresenta l'addestramento di un modello specifico con un dato set di iperparametri e dati.|
|**Tracciamento (Tracking)**|In ogni _Run_, si tracciano automaticamente o manualmente: **Parametri** (gli iperparametri del modello), **Metriche** (es. accuratezza, RMSE), e **Artefatti** (il modello serializzato stesso, grafici, ecc.).|

Il tracciamento consente di confrontare le _Run_ utilizzando l'**Elenco Esecuzioni** (Run List) nell'interfaccia utente di Fabric, permettendo al Data Scientist di scegliere la configurazione ottimale. Si può usare l'**Autologging** per catturare automaticamente parametri e metriche standard.

### Modelli Registrati e Inferenza

Dopo aver identificato il miglior modello tramite gli _Esperimenti_, i suoi artefatti vengono salvati in Fabric come **Modello ML** (_ML Model_).

- **Registro Modelli (Model Registry):** Il Modello ML in Fabric è un contenitore che gestisce diverse **Versioni** dello stesso modello (ogni versione è associata a una specifica _Run_ di MLflow). Questo facilita la gestione e la riproducibilità.
- **Inferenza Batch (Batch Scoring):** Per utilizzare il modello su nuovi dati per generare previsioni, si usa la funzione **`PREDICT`** integrata in Fabric.
    - **Meccanismo:** `PREDICT` è ottimizzata per l'integrazione con i modelli MLflow registrati e consente l'applicazione del modello su _tabelle intere_ di dati in una Lakehouse.
    - **Esempio Pratico:** Dopo aver addestrato un modello di previsione delle vendite, si usa `PREDICT` ogni settimana per applicare il modello ai nuovi dati di vendita e salvare le previsioni risultanti come una nuova tabella nel Lakehouse, che può poi alimentare un report Power BI.
