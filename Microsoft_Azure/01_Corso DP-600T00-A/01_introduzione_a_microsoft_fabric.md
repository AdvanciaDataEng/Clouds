---
autore: Kevin Milli
---

**Microsoft Fabric** è una piattaforma di analisi **end-to-end** e **SaaS (Software as a Service)** che unifica l'intera esperienza di analisi dei dati (data engineering, data warehousing, business intelligence, data science e real-time analytics) in un **unico ambiente integrato**.
Questa unificazione mira a semplificare l'analisi scalabile, che tradizionalmente era complessa, frammentata e costosa.

## I Sei Carichi di Lavoro (Workloads) di Fabric

Fabric consolida i seguenti sei carichi di lavoro (precedentemente servizi o prodotti separati) in un'unica suite, tutti costruiti attorno a **OneLake**:

| Carico di Lavoro | Obiettivo Principale | Strumenti Principali |
| :--- | :--- | :--- |
| **Data Engineering** | Preparazione, trasformazione e gestione dei dati su larga scala. | **Lakehouse**, Notebook (Spark/Python), Pipeline di Dati (Data Factory). |
| **Data Factory** | Integrazione e orchestrazione dei dati (ETL/ELT). | Pipeline, Dataflows Gen2. |
| **Data Warehouse** | Soluzione di data warehousing scalabile basata su SQL. | **Warehouse** (T-SQL Endpoint), Querying. |
| **Real-Time Intelligence** | Ingestione, elaborazione e analisi di dati in streaming (IoT, log). | KQL (Kusto Query Language) Database/Queries, Event Streams. |
| **Data Science** | Sviluppo, addestramento e operationalizzazione di modelli di Machine Learning. | Notebook, MLflow, Integrazione con Azure Machine Learning. |
| **Power BI** | Business Intelligence, visualizzazione, e creazione di report interattivi. | Semantic Models (Modelli Semantici), Report, Direct Lake Mode. |

## Architettura Unificata: OneLake e Aree di Lavoro

L'architettura di Fabric si basa sul concetto di **Lakehouse** e sull'adozione di un formato di dati aperto.

### OneLake: Il Data Lake Unico

**OneLake** è il cuore di Fabric, fungendo da **architettura centralizzata di archiviazione dati** per l'intera organizzazione, basata su **Azure Data Lake Storage (ADLS) Gen2**.

* **Unicità Logica:** Viene spesso definito come "OneDrive per i dati". Unifica tutti i dati dell'organizzazione in un **unico data lake logico**, eliminando i silo di dati e la necessità di spostare o copiare dati tra i diversi carichi di lavoro di Fabric.
* **Formato Aperto:** Tutti i motori di calcolo di Fabric leggono e scrivono dati in un formato standardizzato e aperto, principalmente **Delta Parquet** per i dati tabulari. Questo garantisce l'**interoperabilità** e previene il **vendor lock-in**.
* **Accesso Universale:** Qualsiasi motore analitico in Fabric può accedere direttamente agli stessi dati in OneLake senza modifiche.
* **Shortcuts (Collegamenti):** Sono riferimenti logici a dati esistenti archiviati all'esterno di OneLake (es. in ADLS Gen2, AWS S3). Consentono di includere dati esterni nel lake logico di Fabric **senza duplicarli**, garantendo coerenza e sincronizzazione con l'origine.

> [!Tip] Intuizione
> L'adozione del formato Delta Parquet e l'architettura OneLake rendono Fabric una piattaforma **"Lake-centric"**, dove il **Lakehouse** diventa lo standard per l'archiviazione dati, a differenza delle architetture tradizionali dove il Data Warehouse era centrale.

### Aree di Lavoro (Workspaces)

Le aree di lavoro sono **contenitori logici** fondamentali in Fabric che organizzano e gestiscono dati, modelli, report e altri **elementi (Items)**.

* **Organizzazione e Separazione:** Forniscono una netta separazione delle risorse, supportando la logica del **Data Mesh** (dati come prodotti gestiti da domini distinti).
* **Sicurezza e Governance:** Ogni area di lavoro ha il proprio set di autorizzazioni per il controllo degli accessi. L'amministrazione centralizzata è gestita tramite il **Portale di Amministrazione** di Fabric.
* **Gestione delle Risorse:** Consentono di gestire le risorse di calcolo e le capacità. Si integrano con **Git** (controllo versione) per una collaborazione e un tracciamento delle modifiche professionali.

## Flussi di Lavoro Collaborativi e Ruoli

Fabric è progettato per superare le sfide dei flussi di lavoro di analisi tradizionali, in cui i ruoli erano isolati (**silo di dati e di competenze**). 
L'approccio unificato di Fabric facilita la **collaborazione inter-funzionale**.

| Ruolo Professionale | Contributo in Fabric | Vantaggi Chiave |
| :--- | :--- | :--- |
| **Data Engineer** | Inserisce, trasforma e carica dati (via Pipeline o Notebook) in **Lakehouse** e **Warehouse**. | Sfrutta il formato Delta-Parquet in OneLake; automazione e scalabilità con Data Factory. |
| **Analytics Engineer** | Crea **Modelli Semantici (Semantic Models)** in Power BI per curare e organizzare i dati per l'analisi self-service. | Colma il divario tra dati grezzi e analisi aziendale; garantisce la qualità dei dati. |
| **Data Analyst** | Esegue trasformazioni leggere e crea report interattivi. | **Modalità Direct Lake** (Power BI) elimina la necessità di importare/duplicare dati, leggendo direttamente da OneLake, riducendo i tempi e la complessità. |
| **Data Scientist** | Sviluppa e testa modelli di Machine Learning usando Notebook integrati (Spark, Python). | Accesso diretto ai dati in Lakehouse; integrazione nativa con **MLflow** per il tracciamento degli esperimenti e con Azure ML per l'**MLOps**. |
| **Utenti Business/Citizen Developers** | Utilizza set di dati curati (tramite il **Catalogo OneLake**) e strumenti **low-code/no-code** (es. Dataflows Gen2, Power BI) per analisi self-service. | Accesso facilitato e governato ai "prodotti dati" curati; creazione rapida di dashboard senza dipendenza dall'IT. |

### Copilot in Microsoft Fabric

**Copilot** è un assistente di intelligenza artificiale generativa integrato in tutta la piattaforma Fabric. Migliora la produttività in tutti i carichi di lavoro:

* **Data Engineering:** Generazione di codice Spark/Python nei Notebook.
* **Data Warehousing:** Conversione da linguaggio naturale a query **SQL** (es. "mostrami le vendite totali per regione").
* **Power BI:** Creazione automatica di report e visualizzazioni, oltre a generazione di DAX (Data Analysis Expressions).
* **General:** Fornisce assistenza contestuale, spiegazioni e informazioni dettagliate automatizzate.

>[!Nota] Governance e Catalogo
>Il **Catalogo OneLake** è lo strumento centrale per la governance, consentendo agli utenti di trovare set di dati curati. Fornisce indicazioni su **etichette di riservatezza**, metadati e stato di aggiornamento, garantendo che i dati siano protetti e facilmente accessibili.


---
# Abilitazione e Utilizzo di Microsoft Fabric

Prima di poter sfruttare le funzionalità end-to-end di Microsoft Fabric, l'abilitazione e la configurazione della piattaforma richiedono l'intervento di ruoli amministrativi specifici e la comprensione della struttura delle licenze.

## Abilitazione a Livello di Organizzazione

L'abilitazione di Fabric è un'attività amministrativa che si svolge nel **Portale di Amministrazione** di Power BI.

* **Ruoli Amministrativi:** L'abilitazione è tipicamente gestita da:
    * **Amministratore dell'infrastruttura** (in precedenza Amministratore di Power BI).
    * **Amministratore di Power Platform**.
    * **Amministratore di Microsoft 365**.
* **Procedura:** Gli amministratori abilitano Fabric nelle **Impostazioni tenant** del portale di amministrazione.
* **Ambito di Applicazione:** L'abilitazione può essere configurata per:
    * L'intera organizzazione.
    * Specifici **Gruppi di sicurezza** di Microsoft 365 o Microsoft Entra (per un rilascio controllato).
* **Licenze (Capacità):** Fabric richiede una **capacità di licenza** dedicata. L'amministratore può delegare la capacità di gestire l'abilitazione ad altri utenti a livello di capacità.

>[!Note] Versione di Prova
>Per esplorare Fabric senza un impegno immediato, gli utenti possono iscriversi per una **versione di prova gratuita di Fabric**, che fornisce una capacità temporanea per testare tutti i carichi di lavoro.

## Creazione e Configurazione delle Aree di Lavoro

Le aree di lavoro (Workspaces) sono il punto di partenza per lo sviluppo e la collaborazione in Fabric.

### Ruoli e Controllo degli Accessi

L'accesso all'area di lavoro è gestito tramite quattro ruoli standard, applicabili a **tutti gli elementi** all'interno dell'area di lavoro:

| Ruolo | Permessi Chiave | Utilizzo Tipico |
| :--- | :--- | :--- |
| **Amministratore** | Gestione completa dell'area di lavoro, incluse licenze, permessi, eliminazione. | Proprietari del progetto, IT Governance. |
| **Collaboratore** | Creazione, modifica e gestione della maggior parte degli elementi di Fabric. | Data Engineers, Data Scientists. |
| **Membro** | Condivisione, visualizzazione e contributi limitati ad alcuni elementi. | Analytics Engineers, Analisti che lavorano sui dati. |
| **Visualizzatore** | Solo visualizzazione dei report e degli elementi. | Utenti aziendali, Consumatori di dati. |

> [!Tip] Controllo Granulare
> I ruoli dell'area di lavoro sono pensati per la **collaborazione del team di sviluppo**. Per la sicurezza dei dati e dei report per gli utenti finali (es. solo per la visualizzazione), è raccomandato l'uso di **autorizzazioni a livello di elemento** o la gestione dei permessi in Power BI.

### Impostazioni Avanzate

Nelle impostazioni dell'area di lavoro è possibile configurare aspetti cruciali per l'operatività e la gestione del ciclo di vita del progetto:

* **Tipo di Licenza:** Assegnazione della capacità Fabric appropriata.
* **Integrazione Git:** Fondamentale per il **controllo della versione** (Version Control), tracciando e gestendo le modifiche al codice e agli elementi analitici.
* **Impostazioni Carico di Lavoro Spark:** Ottimizzazione delle prestazioni per le attività di Data Engineering e Data Science.
* **Connessioni:** Accesso a OneDrive e ad Archiviazione di Azure Data Lake Gen2 (se necessario, anche se OneLake è la norma).
* **Derivazione dei Dati (Data Lineage):** Visualizzazione delle dipendenze e dei flussi di dati, essenziale per la trasparenza e la *Data Governance*.

## Individuare i Dati con il Catalogo OneLake

Il **Catalogo OneLake** (spesso integrato con Microsoft Purview) è lo strumento di **Data Governance** e **Data Discovery** di Fabric.

* **Scopo:** Consente agli utenti di trovare, comprendere e accedere ai "prodotti dati" curati all'interno dell'organizzazione.
* **Funzionalità di Ricerca:** Gli utenti possono restringere i risultati usando:
    * Filtri per **Aree di Lavoro** o **Domini** (se la logica del Data Mesh è implementata).
    * Filtri per **Parola Chiave** o **Tipo di Elemento** (es. Lakehouse, Modello Semantico).
    * **Categorie predefinite** per l'esplorazione.
* **Sicurezza:** Gli utenti visualizzano **solo gli elementi condivisi con loro**, rispettando i criteri di accesso configurati.

## Creazione di Elementi e Architettura Data Mesh

Dopo la configurazione, gli utenti possono creare elementi specifici per ciascun carico di lavoro.

| Carico di Lavoro | Elementi Esempi | Obiettivo |
| :--- | :--- | :--- |
| **Data Engineering** | **Lakehouse**, Pipeline di Dati, Notebook. | Costruire il patrimonio di dati grezzi e curati. |
| **Data Warehouse** | **Warehouse**, Endpoint di analisi SQL (SQL Endpoint). | Data warehousing tradizionale basato su T-SQL. |
| **Real-Time Intelligence** | KQL Database, Event Streams. | Gestione e analisi di dati in streaming ad alta velocità. |
| **Power BI** | Modelli Semantici, Report, Dashboard. | Business Intelligence e visualizzazione. |

> [!Tip] Integrazione e Semplificazione
> Fabric **integra** funzionalità di strumenti Microsoft esistenti (Power BI, Azure Synapse Analytics, Azure Data Factory) in un'unica interfaccia. Questa unificazione elimina la necessità di gestire direttamente le singole risorse di Azure sottostanti, semplificando notevolmente il flusso di lavoro dei dati.

### Data Mesh e Fabric

Fabric supporta l'architettura **Data Mesh** (rete di dati), dove l'organizzazione dei dati si basa su domini logici. 
Questo permette:

1.  **Proprietà Decentrata:** I team di dominio possiedono e gestiscono i propri "prodotti dati" (elementi Fabric).
2.  **Governance Centralizzata:** OneLake e il Catalogo OneLake forniscono la governance e la sicurezza unificate a livello di piattaforma.

## Migliorare la Produttività con Copilot in Fabric

**Microsoft Copilot** in Fabric è un potente assistente di **Intelligenza Artificiale Generativa** che accelera le operazioni per tutti gli utenti, dai professionisti dei dati agli utenti aziendali.

### Abilitazione e Funzionalità Trasversali

* **Abilitazione:** Deve essere abilitato dagli amministratori nelle **Impostazioni del tenant** nel Portale di amministrazione di Power BI.
* **Principio:** Sfrutta i modelli linguistici di grandi dimensioni (LLM) per tradurre il **linguaggio naturale** in codice e insight tecnici, mantenendo la conformità con le policy di sicurezza.

### Funzionalità Specifiche per Carico di Lavoro

| Carico di Lavoro | Funzionalità di Copilot | Impatto sulla Produttività |
| :--- | :--- | :--- |
| **Data Engineering/Data Science** | Completamento e suggerimenti di codice intelligenti, automazione di routine, generazione di modelli di codice Spark/Python. | Veloce preparazione dei dati e creazione di pipeline complesse. |
| **Data Warehouse/SQL Database** | Conversione da **Linguaggio Naturale a SQL (NL2SQL)**, completamento del codice, generazione di query complesse. | Permette agli utenti non SQL di interrogare il Warehouse e accelera la scrittura del codice per gli sviluppatori. |
| **Power BI** | Generazione automatica di report, riepilogo delle pagine, miglioramento delle funzionalità Q&A (domande e risposte) tramite sinonimi, chat con i dati. | Democratizza la creazione di report e permette agli utenti aziendali di estrarre insight in modo conversazionale. |
| **Real-Time Intelligence** | Conversione da **Linguaggio Naturale a KQL (Kusto Query Language)**. | Semplifica l'analisi dei dati di streaming per tutti gli utenti, inclusi i *citizen data scientist*. |


# Conclusione: Microsoft Fabric e il Futuro dell'Analisi

Microsoft Fabric rappresenta un **cambiamento paradigmatico** nel panorama dell'analisi dei dati. Integrando l'intera suite di strumenti di analisi (dall'ingestione alla Business Intelligence) in un'unica piattaforma SaaS unificata, Fabric elimina la complessità infrastrutturale e la frammentazione dei dati.

**OneLake** è il catalizzatore di questa unificazione, fornendo un'unica fonte di verità in un formato aperto (Delta Parquet) e garantendo che tutti i carichi di lavoro operino sugli stessi dati senza spostamenti o duplicazioni. Questo approccio non solo riduce i costi e la complessità, ma abilita una collaborazione senza precedenti tra Data Engineers, Data Scientists e Business Users.

L'integrazione pervasiva di **Copilot (AI generativa)** in ogni carico di lavoro accelera ulteriormente la produttività, consentendo agli utenti di tutte le competenze di convertire rapidamente le intenzioni aziendali (in linguaggio naturale) in codice, query e insight operativi. In sintesi, Fabric non è solo una raccolta di servizi; è una **piattaforma di dati e analisi completa** progettata per democratizzare l'accesso ai dati e accelerare il processo decisionale basato sui dati in modo governato e scalabile.