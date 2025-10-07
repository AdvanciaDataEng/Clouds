
Le pipeline di dati in **Microsoft Fabric** sono l'evoluzione delle pipeline di **Azure Data Factory (ADF)** e di **Azure Synapse Analytics**. Offrono un'esperienza unificata per l'orchestrazione dei processi di **Estrazione, Trasformazione e Caricamento (ETL)** o **Estrazione, Caricamento e Trasformazione (ELT)**.

L'obiettivo principale di una pipeline è definire una sequenza logica di **attività** per orchestrare un processo complessivo, come l'inserimento di dati transazionali in un archivio analitico come un **Lakehouse** o un **Data Warehouse** di Fabric.

---

## Concetti Chiave delle Pipeline

### Attività (Activities)

Le attività sono le azioni specifiche eseguite all'interno di una pipeline. Il flusso di esecuzione è definito collegando le attività, e il risultato di un'attività (successo, fallimento o completamento) può dirigere l'esecuzione all'attività successiva.

Le attività si dividono in tre categorie principali:

1. **Attività di Spostamento Dati (Data Movement)**
    - **Attività Copia Dati (Copy Data):** L'attività fondamentale per copiare dati da una **sorgente esterna** (un'ampia gamma di connettori è supportata, inclusi database SQL, archivi cloud, ecc.) a una **destinazione** (spesso un Lakehouse o un Data Warehouse in OneLake).
        - **Uso Tipico:** Inserimento di dati grezzi (_raw data_) in un'area di staging (ad esempio, la sezione _Files_ di un Lakehouse) o caricamento diretto in una tabella per scenari ETL/ELT semplici senza trasformazioni complesse.
        - **Assistente Copia Dati:** Uno strumento grafico semplifica la configurazione di sorgente e destinazione per l'attività **Copia Dati**.
    
2. **Attività di Trasformazione Dati (Data Transformation)**
    - **Flusso di Dati (Dataflow) / Flusso di Dati Gen2:** Incapsula i **Dataflow Gen2** basati su **Power Query**, consentendo di applicare trasformazioni complesse ai dati durante l'inserimento. È ideale per la pulizia, la preparazione e l'unione di dati da più sorgenti con un'interfaccia a basso codice.
    - **Notebook:** Esegue un **notebook Spark**, consentendo di eseguire codice in linguaggi come Python, Scala, o SQL per trasformazioni ELT/ETL avanzate e personalizzate (ad esempio, con librerie come _PySpark_).
    - **Stored Procedure:** Esegue un codice **SQL** predefinito in un database (ad esempio, in un Data Warehouse) per l'elaborazione dei dati.
    - **Elimina Dati (Delete Data):** Attività per eliminare dati esistenti, spesso usata prima di un'attività **Copia Dati** per implementare logiche di ricarico completo (_full refresh_).
    
3. **Attività del Flusso di Controllo (Control Flow)**
    - **ForEach (Per Ogni):** Implementa un ciclo per iterare su una collezione di elementi (ad esempio, nomi di file o cartelle) ed eseguire un set di attività per ogni elemento. Essenziale per l'automazione su larga scala.
    - **If Condition (Condizione Se):** Consente la diramazione condizionale del flusso, eseguendo diverse attività in base alla valutazione di un'espressione booleana.
    - **Append Variable (Aggiungi Variabile), Set Variable (Imposta Variabile):** Gestisce i valori di variabili e parametri.
    - **Execute Pipeline (Esegui Pipeline):** Permette a una pipeline di invocarne un'altra (riutilizzo e modularità).

### Parametri (Parameters)

La **parametrizzazione** è una pratica chiave che aumenta la **riutilizzabilità** e la **flessibilità** delle pipeline.

- I parametri consentono di fornire valori specifici (es. nome della cartella di output, data di inizio, credenziali) all'esecuzione della pipeline, rendendo un unico _design_ adatto a molteplici scenari.
- I valori dei parametri possono essere passati alle attività interne (ad esempio, specificando dinamicamente il percorso di origine per un'attività **Copia Dati**).

### Esecuzioni della Pipeline (Pipeline Runs)

Ogni avvio di una pipeline, sia esso **su richiesta** (manuale) o **pianificato** (automatico tramite un _trigger_), è chiamato **Esecuzione della Pipeline di Dati** (_Data Pipeline Run_).

- Ogni esecuzione riceve un **ID di Esecuzione** univoco, fondamentale per il **monitoraggio** e la risoluzione dei problemi (_troubleshooting_).

---

## Usare Modelli e Best Practice

### Modelli di Pipeline (Pipeline Templates)

Fabric fornisce **modelli predefiniti** per scenari comuni di integrazione dei dati (ad esempio, caricamento di dati da un database SQL a un Lakehouse), riducendo i tempi di configurazione.

- I modelli sono un ottimo punto di partenza e possono essere personalizzati nell'area di disegno (_canvas_) della pipeline.

### Esecuzione e Monitoraggio

1. **Convalida:** Prima dell'esecuzione, la funzione **Convalida** verifica che la configurazione della pipeline sia valida.
2. **Esecuzione Interattiva e Pianificazione:** È possibile eseguire la pipeline manualmente (**Run**) o configurare una **Pianificazione** (_Schedule_).
3. **Cronologia di Esecuzione (Run History):**
    - Il **Monitoraggio** delle pipeline (disponibile nell'interfaccia utente di Fabric o nel **Monitoring Hub** dell'area di lavoro) mostra lo stato, la durata e i dettagli di ogni esecuzione.
    - È possibile analizzare i log, i dati di input/output e i messaggi di errore per attività specifiche.
    - Una funzionalità utile è la possibilità di **Rieseguire** una pipeline intera o di rieseguirla **dall'attività fallita** (ottimizzazione dei tentativi).

### Intuizioni e Differenze con ADF

|Caratteristica|Azure Data Factory (ADF)|Microsoft Fabric Data Factory (Pipeline)|
|---|---|---|
|**Architettura**|Servizio autonomo di Azure. Richiede _Linked Services_ (Servizi Collegati) e _Datasets_ (Set di Dati). Richiede **Integration Runtimes (IR)** per il calcolo.|Esperienza **SaaS** integrata in Microsoft Fabric. Semplifica con **Connessioni** (sostituiscono _Linked Services_ e _Datasets_ in molte attività). **Nessuna gestione di IR** (calcolo gestito automaticamente da Fabric).|
|**Trasformazione**|**Mapping Data Flows** (Flussi di Dati di Mapping) per trasformazioni visuali.|**Dataflow Gen2** (Power Query) per trasformazioni visuali, con integrazione profonda in OneLake.|
|**Integrazione**|Necessita di configurazione esplicita per connettersi a Synapse, Storage Account, ecc.|Integrazione _seamless_ (senza soluzione di continuità) con Lakehouse, Data Warehouse e altri componenti di Fabric tramite **OneLake**.|
|**CI/CD**|Richiede l'integrazione con un repository Git esterno (Azure DevOps/GitHub).|Offre funzionalità CI/CD con **Pipeline di Distribuzione** (_Deployment Pipelines_) integrate in Fabric (opzionale, non richiede sempre Git esterno).|
