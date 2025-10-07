---
autore: Kevin Milli
---

Microsoft Fabric fornisce una piattaforma unificata per l'analisi dei dati, e i **Flussi di Dati Gen2** (Dataflows Gen2) rappresentano uno strumento **Extract, Transform, Load (ETL)** low-code per l'ingestione e la preparazione dei dati, potenziato da **Power Query Online**. Sono fondamentali per creare un **modello semantico** (dati puliti e pronti all'uso) riutilizzabile e coerente all'interno dell'organizzazione.

---

## Concetti Chiave e Architettura

Un Flusso di Dati Gen2 è un elemento di Microsoft Fabric che incapsula la logica di connessione, trasformazione e caricamento dei dati in una destinazione supportata.

### Funzionalità Core

- **ETL Low-Code:** Semplificano il processo ETL/ELT grazie all'interfaccia visuale di Power Query Online.
- **Destinazione Dati (Data Destination):** Una caratteristica distintiva di Gen2 è la capacità di scrivere i dati trasformati direttamente in una destinazione di Fabric (Lakehouse, Warehouse) o esterna (Azure SQL Database, Azure Data Explorer, Azure Synapse Analytics).
    - L'impostazione della destinazione è gestibile singolarmente per ogni query all'interno del flusso.
    - Per le destinazioni Fabric, i dati vengono scritti in formato **Delta Parquet** (migliore per l'integrazione a valle come Direct Lake), a differenza di Gen1 che scriveva in CSV.
- **Integrazione con Data Factory:** I flussi di dati Gen2 risiedono nel carico di lavoro **Data Factory** (o Power BI/Lakehouse) e possono essere incorporati come attività nelle **Pipeline di Dati** per l'orchestrazione.
- **Staging:** Per impostazione predefinita, Gen2 usa la gestione temporanea (staging), che utilizza elementi Lakehouse e Warehouse nell'area di lavoro per migliorare le prestazioni. Tuttavia, è un fattore chiave nell'utilizzo delle **Capacity Units (CU)** e può essere disabilitato per query leggere per ottimizzare i costi.

### Differenze Cruciali con Dataflow Gen1

|Caratteristica|Dataflow Gen1 (Power BI/Power Apps)|Dataflow Gen2 (Microsoft Fabric)|
|---|---|---|
|**Destinazione Dati**|Archiviazione in Dataverse o Azure Data Lake Storage Gen2 (ADLS Gen2) con formato CSV/CDM.|Scrittura nativa in **Lakehouse** e **Warehouse** (in formato **Delta Parquet**) o altre destinazioni Azure.|
|**Motore di Calcolo**|Motore mashup (M) con calcolo Power BI.|Motore mashup (M) standard con ottimizzazioni per la scrittura su Fabric (Lakehouse/Warehouse).|
|**Aggiornamento Incrementale**|Configurato _dopo_ la pubblicazione nell'elemento (Dataflow Settings).|Funzionalità di **prima classe** configurabile _direttamente nell'editor_ di Power Query; non richiede parametri manuali.|
|**Integrazione CI/CD**|Meno nativa.|Supporto nativo per **Git Integration** e **Deployment Pipelines** in Fabric (richiede la modalità CI/CD).|
|**Monitoraggio**|Cronologia aggiornamenti.|Integrazione con l'**Hub di Monitoraggio** per una migliore visibilità.|

### Gestione del Costo (Capacity Units - CU)

L'utilizzo delle **Capacity Units (CU)** in Fabric è un fattore cruciale. I Flussi di Dati Gen2 consumano CU in base alla durata e al tipo di calcolo:

- **Calcolo Standard (Motore Mashup):** Tariffa a due livelli. Esecuzioni brevi (meno di 10 minuti) hanno un costo fisso in CU, mentre quelle più lunghe sono tariffate al secondo.
- **Calcolo su Larga Scala (con Staging):** Se lo staging è abilitato (necessario per scrivere in Warehouse o quando si usa Fast Copy), il calcolo coinvolge il motore SQL/Lakehouse e ha una tariffa specifica.
- **Fast Copy:** Utilizza una tariffa separata, basata sul tempo di esecuzione, per ottimizzare lo spostamento dei dati in Lakehouse/Warehouse (quando le trasformazioni sono leggere).

**Intuizione per l'Ottimizzazione:** Per ridurre l'utilizzo di CU:

1. **Disabilita lo Staging** se non strettamente necessario (ad esempio, se le trasformazioni sono molto leggere e non scrivi su un Warehouse).
2. Sfrutta l'**Aggiornamento Incrementale** per ridurre la quantità di dati elaborati ad ogni esecuzione.
3. Utilizza l'opzione **Fast Copy** per l'ingestione bulk quando possibile.

---

## Interfaccia e Sviluppo

La creazione di un Flusso di Dati Gen2 si basa sull'interfaccia di **Power Query Online**, che rende l'ETL un'esperienza **visiva** e intuitiva, familiare agli utenti di Power BI e Power Platform.

### Componenti dell'Editor Power Query Online

|Componente|Descrizione|Utilizzo Pratico|
|---|---|---|
|**Barra Multifunzione**|Accesso a connettori, trasformazioni, gestione dei parametri e configurazione della Destinazione Dati.|**Trasformazioni Avanzate:** _Merge_ e _Append_ di query, _Pivot/Unpivot_, _Split Conditional_.|
|**Riquadro Query**|Elenco delle origini dati (chiamate **Query**) che diventano tabelle nella destinazione.|**Riusabilità:** Si possono duplicare o fare riferimento (reference) alle query esistenti per creare tabelle dimensionali e dei fatti separate (schema star) partendo da un'unica ingestione.|
|**Vista Diagramma**|Visualizzazione grafica del flusso di dati e delle relazioni tra le query.|Utile per comprendere e documentare la **dipendenza** tra le query (ad esempio, se una query fa riferimento a un'altra).|
|**Riquadro Anteprima Dati**|Mostra un sottoinsieme dei dati per applicare e visualizzare le trasformazioni.|**Ispezione:** Clic con il tasto destro sulle colonne per filtrare, modificare tipi, o apportare veloci modifiche.|
|**Riquadro Impostazioni Query**|Mostra i **Passaggi Applicati** (ogni trasformazione è un passaggio) e le impostazioni della **Destinazione Dati**.|**Query Folding (Riduzione Query):** Se l'origine lo supporta, la visualizzazione dei passaggi applicati mostra se le trasformazioni vengono "ridotte" e inviate all'origine dati per un'esecuzione più efficiente.|
|**Editor Avanzato**|Accesso diretto al codice **M** (linguaggio Power Query) per trasformazioni personalizzate e complesse.|Permette l'ottimizzazione manuale della logica e l'uso di funzioni Power Query avanzate.|

### Tecniche di Uso Avanzato

1. **Orchestrazione ELT vs. ETL:**
    - **ETL (Dataflow Gen2 Completo):** Il flusso esegue (E)xtract, (T)ransform, e (L)oad in un unico passaggio. Ottimo per modelli semantici curati.
    - **ELT (Pipeline + Dataflow Gen2):** La Pipeline esegue (E)xtract e (L)oad su un Lakehouse (raw data), e successivamente un Dataflow Gen2 si connette al Lakehouse per (T)ransform e scrivere su un'area curata. Migliore per architetture a medaglione.
2. **Integrazione CI/CD e ALM:** L'integrazione nativa con **Git** e **Deployment Pipelines** (sviluppo ![](data:,) test ![](data:,) produzione) assicura **coerenza** nella logica di trasformazione e consente una **configurazione specifica per la fase** (es. connettendosi a diverse origini dati in Sviluppo e Produzione).
3. **Parametrizzazione:** L'uso di **Parametri Pubblici** consente di rendere dinamici i flussi. Si possono parametrizzare elementi come le origini dati (stringa di connessione), le destinazioni o parti della logica di trasformazione. Questo permette alle pipeline a valle di richiamare lo stesso flusso di dati con valori diversi a seconda dell'ambiente (Dev, Test, Prod).

---

## Vantaggi e Scenari

|Vantaggi|Scenari di Applicazione|
|---|---|
|**Logica ETL Riutilizzabile**|Creazione di una **Data Source of Truth** (DSOT) per i dati puliti (modello semantico) accessibile da più team o report.|
|**Semplificazione del Self-Service**|Consentire agli analisti di dati (già esperti in Power Query) di curare autonomamente i dati senza ricorrere a codice complesso.|
|**Consistenza e Qualità dei Dati**|Pulizia e trasformazione dei dati a monte per garantire la coerenza tra i vari report.|
|**Ottimizzazione del Refresh**|Riduzione del carico sull'origine dati sottostante, poiché l'estrazione avviene una sola volta per il riutilizzo. L'uso dell'**Aggiornamento Incrementale** riduce notevolmente i tempi e i costi di aggiornamento.|

---

## Integrazione con le Pipeline di Dati (Orchestrazione)

L'attività **Dataflow** all'interno di una Pipeline di Dati consente di orchestrare l'esecuzione del Flusso di Dati Gen2.

- **Sequenza:** È la chiave per i processi multi-passaggio. Ad esempio: `[Attività 1: Esegui Dataflow Gen2]` $\longrightarrow$ `[Attività 2: Esegui Notebook/Stored Procedure su Lakehouse]`.
- **Pianificazione e Trigger:** La pipeline gestisce l'automazione, consentendo di eseguire il flusso di dati con una **pianificazione** o in base a un **trigger** (es. l'arrivo di un file).
- **Monitoraggio:** La pipeline fornisce un punto centrale per il monitoraggio di tutti i passaggi, inclusa l'esecuzione del Dataflow.

L'inserimento dei dati è fondamentale nell'analisi. Guarda questo video per una spiegazione dettagliata delle funzionalità di Dataflow Gen2: [Microsoft Fabric Learn Together Ep05: Inserire dati con flussi di dati Gen2 in Microsoft Fabric](https://learn.microsoft.com/it-it/shows/learn-live/learn-together-microsoft-fabric-wave-1-ep205-ingest-data-with-dataflows-gen2-in-microsoft-fabric).

