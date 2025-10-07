---
autore: Kevin Milli
---

## Introduzione e Architettura Fondamentale

Microsoft Fabric si fonda sull'architettura **Lakehouse**, una piattaforma unificata che combina la **flessibilità e la scalabilità** di un **Data Lake** con le **capacità di gestione e analisi strutturata** di un **Data Warehouse**.

|Componente|Descrizione|Vantaggio Chiave|
|---|---|---|
|**Lakehouse**|L'elemento centrale per l'archiviazione, la gestione e l'analisi dei dati, strutturati e non strutturati.|Unificazione dei dati in un unico luogo logico.|
|**OneLake**|L'unico **Data Lake logico e unificato** per l'intera organizzazione, integrato automaticamente con ogni tenant di Fabric.|Elimina i **silo di dati** e riduce la necessità di spostare o duplicare i dati. "OneDrive per i dati".|
|**Formato Delta Lake**|Formato open-source utilizzato per le tabelle nel Lakehouse. Consente l'applicazione delle proprietà **ACID** (Atomicity, Consistency, Isolation, Durability).|Garantisce l'**integrità e la coerenza** dei dati, essenziale per l'analisi e il Machine Learning.|

---

## Struttura e Elementi Correlati di Lakehouse

Quando si crea un Lakehouse, Fabric genera automaticamente tre elementi chiave nell'area di lavoro, garantendo un'esperienza end-to-end:
### Lakehouse Explorer

È l'interfaccia principale per interagire con il Lakehouse. Organizza l'archiviazione dati in due sezioni logiche in OneLake:

1. **Files:** Area di archiviazione non strutturata per tutti i formati di file (CSV, Parquet, JSON, ecc.).
2. **Tables:** Area che contiene le **tabelle in formato Delta Lake**. Queste tabelle sono ottimizzate per l'interrogazione ad alte prestazioni.

### Endpoint di Analisi SQL (SQL Analytics Endpoint)

- Un **endpoint TDS** (Tabular Data Stream) viene creato **automaticamente** per ogni Lakehouse.
- Offre accesso **solo in lettura** alle tabelle Delta del Lakehouse tramite **T-SQL** (Transact-SQL).
- Utilizza lo stesso motore di calcolo ad alte prestazioni del servizio Data Warehouse di Fabric.
- **Individuazione automatica dei metadati:** Legge i log Delta per mantenere aggiornati i metadati SQL (come le statistiche) senza intervento manuale.

### Modello Semantico predefinito (Default Semantic Model)

- Fornisce un'origine dati semplificata e pronta all'uso per gli sviluppatori di **Power BI**.
- Consente la creazione di report e dashboard basati sui dati del Lakehouse, sfruttando la modalità **DirectLake** per query veloci e bassa latenza.

---

## Inserimento, Trasformazione e Consumo dei Dati

### Inserimento (Ingest)

È il primo passo del processo **ETL** (Extract, Transform, Load) o **ELT** (Extract, Load, Transform). 
I dati possono essere in molti formati comuni (Parquet, CSV, Avro, JSON, ecc.) e provenienti da varie origini.

|Metodo di Inserimento|Ruolo Utente Target|Intuito Pratico|
|---|---|---|
|**Pipeline di Data Factory** (Attività Copia Dati)|Ingegneri dei Dati, Sviluppatori Dati|Interfaccia visuale per orchestrare e trasferire dati in blocco.|
|**Notebooks** (Apache Spark)|Ingegneri dei Dati, Data Scientist|Codice (PySpark, SQL, Scala) per inserimento, trasformazione e caricamento avanzato.|
|**Flussi di Dati Gen2** (Dataflows Gen2)|Sviluppatori Power BI/Excel, Citizen Developers|Interfaccia **Power Query** (M Language) per l'importazione e la trasformazione dei dati.|
|**Caricamento**|Qualsiasi Utente|Caricamento di file locali direttamente nell'Explorer.|

### Collegamenti (Shortcuts) in OneLake

I collegamenti sono un elemento cruciale dell'architettura OneLake:

- **Definizione:** Punti di riferimento che consentono di integrare dati da origini esterne (ad esempio, Azure Data Lake Store Gen2, S3 di AWS) o da altri elementi di Fabric **senza spostare o duplicare i dati**.
- **Vantaggio:** I dati rimangono nella loro posizione originale, riducendo i costi di uscita e mantenendo una **singola copia della verità**.
- **Accesso:** I motori di calcolo (Spark, SQL, Power BI) possono interrogare i dati tramite il collegamento, che appare come una cartella nell'Explorer.
- **Sicurezza:** Le autorizzazioni sono gestite da OneLake e le credenziali vengono gestite in modo trasparente.

### Trasformazione e Caricamento (Transform and Load)

La trasformazione avviene utilizzando gli stessi strumenti di inserimento, ma focalizzandosi sulla preparazione del dato:

- **Notebooks (Spark):** Ideali per trasformazioni complesse su larga scala e carichi di lavoro di Machine Learning.
- **Dataflows Gen2:** Ottimi per trasformazioni più semplici guidate dall'interfaccia utente.
- **Destinazione:** I dati trasformati possono essere caricati come **File** (per l'archiviazione non elaborata o intermedia) o come **Tabelle Delta** (per l'analisi finale).

### Consumo (Consumption)

Il Lakehouse funge da piattaforma centralizzata per tutti i ruoli analitici:

- **Data Analyst:** Utilizzano l'**Endpoint di Analisi SQL** per eseguire query T-SQL e l'interfaccia **Power BI** per la visualizzazione.
- **Data Scientist:** Utilizzano **Notebooks** e **Data Wrangler** per l'esplorazione, l'addestramento di modelli di Machine Learning e l'Intelligenza Artificiale.
- **Data Engineer:** Gestiscono il flusso ETL/ELT e la qualità dei dati utilizzando Spark/Notebooks e Dataflows.

---

## Protezione e Governance

- **Controllo Accessi:** L'accesso è gestito tramite **Ruoli dell'Area di Lavoro** (per i collaboratori con accesso completo) o **Condivisione a Livello di Elemento** (ideale per esigenze di sola lettura, come lo sviluppo di report).
- **Governance Dati:** Funzionalità di governance avanzata, incluse le **Etichette di Riservatezza (Sensitivity Labels)**, sono estese tramite **Microsoft Purview** integrato con il tenant di Fabric.
