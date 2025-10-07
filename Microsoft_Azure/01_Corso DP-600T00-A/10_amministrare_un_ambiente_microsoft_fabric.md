
## Introduzione al Ruolo di Amministratore di Fabric

L'amministrazione di un ambiente **Microsoft Fabric** è cruciale per garantire l'uso efficiente, sicuro e conforme della piattaforma di analisi end-to-end. Fabric è una soluzione **Software-as-a-Service (SaaS)** che mira a unificare gli strumenti di analisi.

Gli amministratori di Fabric devono possedere competenze e conoscenze fondamentali in aree chiave:

- **Architettura di Fabric:** Comprensione dei componenti unificati, in particolare **OneLake** e le **Capacità**.
- **Sicurezza e Governance:** Gestione degli accessi, delle licenze e applicazione dei criteri di conformità.
- **Funzionalità di Analisi:** Familiarità con le diverse _esperienze_ (Data Engineering, Data Science, Power BI, ecc.).
- **Opzioni di Distribuzione e Licenze:** Gestione delle licenze utente e di capacità.

Il ruolo, precedentemente associato all'**Amministratore di Power BI**, collabora strettamente con utenti aziendali, analisti dei dati e professionisti IT per allineare l'uso di Fabric agli obiettivi aziendali e agli standard di governance.

---

## Informazioni sull'Architettura di Fabric

Microsoft Fabric è progettato per semplificare l'architettura di analisi, offrendo un'unica soluzione unificata.

### **Servizi Fondamentali**

Fabric offre una suite completa di servizi (_esperienze_) che coprono l'intero ciclo di vita dell'analisi dei dati:

1. **Data Factory:** Integrazione e orchestrazione dei dati (pipeline ETL/ELT).
2. **Data Engineering:** Ambiente Spark per la lavorazione di Big Data (notebook, trasformazioni).
3. **Data Warehouse:** Data warehousing cloud-native basato su **T-SQL** con separazione tra calcolo e storage.
4. **Real-Time Analytics:** Elaborazione di flussi di dati continui utilizzando **KQL** (Kusto Query Language).
5. **Data Science:** Sviluppo, addestramento e distribuzione di modelli di Machine Learning.
6. **Power BI:** Business Intelligence per la creazione di report e dashboard.
7. **Data Activator (Preview):** Attivazione automatica di azioni/notifiche in base a condizioni sui dati.

### **OneLake: Il "OneDrive per i Dati"**

Il cuore di Fabric è **OneLake**, un **data lake unificato e logico** per l'intera organizzazione.

- **Base Tecnica:** Basato sull'architettura **Azure Data Lake Storage (ADLS) Gen2**.
- **Caratteristica Unica:** È l'unica posizione per tutti i dati di analisi (**Single Data Lake**) ed è automaticamente provisionato con ogni **Tenant** di Fabric.
- **Formato Aperto:** Archivia i dati nel formato **Delta Parquet**, consentendo l'accesso e la query tramite molteplici motori analitici (T-SQL, Apache Spark, Power BI) senza la necessità di spostare o duplicare i dati.
- **Obiettivo:** Rimuovere i silo di dati e ridurre drasticamente lo spostamento/duplicazione dei dati, promuovendo il concetto di **"One Copy"**.
- **Integrazione:** Utilizza **Collegamenti (Shortcuts)** per fare riferimento a dati archiviati in altre posizioni (in altri cloud o ADLS Gen2), facendoli apparire come se fossero archiviati localmente in OneLake.

### **Concetti Chiave di Fabric (Gerarchia di OneLake)**

|Concetto|Descrizione|Livello Gerarchico|
|---|---|---|
|**Tenant**|Spazio dedicato all'organizzazione, allineato con **Microsoft Entra ID**. Corrisponde alla **radice** di OneLake.|Livello Superiore (Root)|
|**Capacità**|Set dedicato di risorse di calcolo disponibili per l'esecuzione dei carichi di lavoro. **Definisce l'abilità di eseguire attività**. La licenza di capacità è organizzativa.|Risorsa di Calcolo|
|**Dominio**|Raggruppamento logico di Aree di Lavoro (**Workspaces**), usato per l'organizzazione e la governance dei dati a livello aziendale.|Livello di Raggruppamento Logico|
|**Area di Lavoro (Workspace)**|Contenitore per gli **Elementi** di Fabric, funge da cartella in OneLake. Utilizza la capacità per il lavoro eseguito e gestisce i controlli di accesso.|Livello Cartella|
|**Elementi (Items)**|I blocchi predefiniti della piattaforma (es. Data Warehouse, Lakehouse, Notebook, Report, Dataset).|Contenuto (Files/Tabelle)|

---

## Strumenti e Attività dell'Amministratore di Fabric

L'amministratore di Fabric utilizza una varietà di strumenti per gestire l'ambiente, con compiti che spaziano dalla sicurezza al monitoraggio.

### **Attività Principali**

|Categoria di Attività|Descrizione|
|---|---|
|**Controllo Sicurezza e Accesso**|Gestione di chi può accedere, visualizzare o modificare il contenuto (tramite **RBAC**, Microsoft Entra ID), inclusa la configurazione di gateway dati e l'applicazione di **Accesso Condizionale** per restrizioni di rete (es. IP).|
|**Governance dei Dati**|Applicazione di criteri, monitoraggio delle metriche di utilizzo/prestazioni, gestione della residenza dei dati e garanzia della conformità (es. **Microsoft Purview**).|
|**Personalizzazione e Configurazione**|Configurazione di collegamenti privati (per la sicurezza di rete), definizione dei criteri di classificazione, e gestione delle impostazioni a livello di tenant (es. Interruttore Fabric On/Off).|
|**Monitoraggio e Ottimizzazione**|Supervisione delle prestazioni (es. ottimizzazione delle query), gestione delle risorse di capacità e risoluzione dei problemi.|

### **Strumenti Amministrativi Essenziali**

1. **Portale di Amministrazione di Fabric:**
    - Portale web centralizzato per la gestione di tutte le impostazioni del tenant e della capacità.
    - Consente di attivare/disattivare funzionalità, gestire utenti e gruppi (in combinazione con Microsoft 365 Admin Center), e accedere ai log di controllo.
    - Per accedere, è necessaria una licenza di Fabric e il ruolo di **Amministratore di Fabric**.
        
2. **Interfaccia di Amministrazione di Microsoft 365:** Gestisce le **licenze utente** per Fabric e i ruoli amministrativi di alto livello (Amministratore Globale, Amministratore Licenze).
3. **Area di Lavoro Monitoraggio Amministratore:** Accessibile agli amministratori tenant, include report su **utilizzo e adozione delle funzionalità** per identificare tendenze e risolvere problemi di performance.
4. **Cmdlet di PowerShell & API/SDK di Amministrazione (REST):**
    - Consentono l'**automazione** delle attività amministrative comuni (creazione/gestione gruppi, configurazione origini dati/gateway).
    - Le API richiedono autenticazione **OAuth 2.0** (es. con Postman o Entità Servizio per operazioni in background).
5. **Sicurezza e Conformità Microsoft 365 / Portale di Conformità Microsoft Purview:** Utilizzato per il controllo, la classificazione dei dati, i criteri di prevenzione della perdita dei dati (**DLP**) e la gestione della sicurezza.

---

## Sicurezza e Governance dei Dati in Fabric

La governance dei dati in Fabric è supportata da funzionalità native che aiutano a costruire **fiducia (trust)** nei dati e a garantirne la conformità.
### **Gestione Utenti: Licenze e Ruoli**

- **Licenze:** Sono di due tipi:
    - **Licenza di Capacità (Organizzativa):** Fornisce il pool di risorse di calcolo per l'organizzazione (SKU F, P, EM, A).
    - **Licenza per Utente (Individuale):** Controlla l'accesso e le funzionalità per l'utente (es. Power BI Pro/Premium per Utente).
- **Ruoli Amministrativi:** Oltre all'Amministratore di Fabric (che gestisce le funzionalità e le impostazioni di tenant/capacità), ci sono i **Ruoli di Amministratore della Capacità** (gestiscono le aree di lavoro all'interno di una capacità) e i ruoli di Microsoft 365/Entra ID (per licenze, utenti, ecc.).

### **Gestione Elementi e Condivisione**

- La prassi migliore è concedere i **diritti permissivi minimi**.
- **Distribuzione:** Avviene tramite l'**App dell'Area di Lavoro** (ideale per l'accesso in sola lettura ai report) o direttamente tramite l'Area di Lavoro (per la collaborazione e lo sviluppo).
- È possibile gestire la condivisione **interna ed esterna** all'organizzazione in base ai criteri.

### **Verifica dell'Autenticità del Contenuto (Endorsement)**

L'approvazione (Endorsement) è una funzionalità chiave per la governance che crea fiducia negli asset di dati. Gli elementi approvati sono etichettati e ottengono precedenza nelle ricerche.

1. **Promosso (Promoted):**
    - Indica che l'autore (o un collaboratore/amministratore dell'Area di Lavoro) ritiene che l'elemento sia **pronto per la condivisione e il riutilizzo**.
    - Livello di fiducia di base, gestito dai creatori.
        
2. **Certificato (Certified):**
    - Indica che un **revisore autorizzato** dall'organizzazione (designato dall'Amministratore di Fabric) ha certificato che l'elemento soddisfa gli standard di **qualità, affidabilità e autorevolezza** dell'organizzazione.
    - Richiede un processo più formale.
        
3. **Dati Master (Master Data) - _Nuovo badge d'approvazione_**:
    - Indica che i dati nell'elemento sono considerati una **fonte primaria** e l'**unica fonte di riferimento autorevole** per specifici tipi di dati aziendali (es. elenchi clienti, codici prodotto).
        

### **Tracciamento e Analisi dei Dati Sensibili**

- **Derivazione dei Dati (Data Lineage) / Analisi dell'Impatto:**
    - Consente di tracciare il **flusso di dati** attraverso Fabric, visualizzando la provenienza, le trasformazioni e la destinazione dei dati (analisi dell'impatto).
        
- **Analisi dei Metadati (API Scanner):**
    - Un set di **API REST di amministrazione** che consente di **analizzare gli elementi** di Fabric (Data Warehouse, Report, ecc.) per estrarre metadati, inclusa la riservatezza e lo stato di approvazione.
    - Facilita la catalogazione e la creazione di report sui metadati.
        
- **Hub di Microsoft Purview:**
    - Fornisce un gateway in Fabric per gestire e governare il patrimonio di dati.
    - Contiene report sui dati sensibili, sull'autenticità degli elementi e sui domini, fornendo accesso a funzionalità avanzate di **Microsoft Purview** (es. Protezione delle Informazioni, DLP).
