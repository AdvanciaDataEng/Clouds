---
autore: Kevin Milli
---

La gestione del ciclo di vita (LCM) in Microsoft Fabric è l'insieme di pratiche e strumenti che consentono ai team di data engineer e sviluppatori di gestire, controllare la versione e distribuire in modo efficiente gli elementi di Fabric (come Lakehouse, Notebook, Pipeline, Modelli Semantici, ecc.) attraverso diversi ambienti (Sviluppo, Test, Produzione).

L'LCM in Fabric si basa su due pilastri fondamentali per implementare l'**Integrazione Continua (CI)** e il **Recapito Continuo (CD)**:

1. **Integrazione con Git** per il controllo della versione e la collaborazione.
2. **Pipeline di Distribuzione di Fabric** per l'automazione del rilascio.
3. **API REST di Fabric** per l'automazione avanzata del processo CI/CD.

---

## 1. Comprendere l'Integrazione Continua e il Recapito Continuo (CI/CD)

**CI/CD** è una metodologia che automatizza le fasi di sviluppo, testing e rilascio del software. L'obiettivo è ridurre i rischi di integrazione, velocizzare il rilascio delle funzionalità e garantire che le modifiche vengano testate e distribuite in modo rapido e affidabile.

### 1.1 Integrazione Continua (CI)

La **CI** si concentra sull'integrazione frequente e automatizzata del codice sviluppato da più membri del team in un repository centrale (ad esempio, un ramo principale o _trunk_).

|Concetto|Spiegazione|
|---|---|
|**Integrazione Frequente**|Gli sviluppatori eseguono il **commit** delle modifiche in rami di codice condivisi con cadenza regolare (più volte al giorno).|
|**Build e Test Automatici**|Ogni commit o **pull request** (richiesta di unione) attiva automaticamente processi di **build** e **test** (es. test unitari) per verificare che le nuove modifiche non introducano bug o conflitti con il codice esistente.|
|**Vantaggio in Fabric**|Identificare rapidamente i problemi di compatibilità tra gli elementi di Fabric (ad esempio, un Notebook modificato e la Lakehouse che utilizza) prima che vengano promossi in ambienti successivi.|

### 1.2 Recapito Continuo (CD)

Il **Recapito Continuo** (in inglese _Continuous Delivery_) e la **Distribuzione Continua** (in inglese _Continuous Deployment_) sono le fasi successive alla CI.

|Fasi di Distribuzione|Spiegazione|
|---|---|
|**Recapito Continuo (Continuous Delivery)**|Il codice che supera tutti i test di CI viene automaticamente preparato per il rilascio. È garantito che possa essere distribuito in un ambiente di _staging_ o di **Produzione** in qualsiasi momento, anche se la distribuzione finale in Produzione può richiedere un'approvazione manuale.|
|**Distribuzione Continua (Continuous Deployment)**|Estensione del CD in cui il codice che supera tutti i test e le approvazioni automatizzate viene **rilasciato automaticamente** nell'ambiente di **Produzione**, senza bisogno di intervento umano.|
|**In Fabric**|Le **Pipeline di Distribuzione** di Fabric automatizzano questo processo, consentendo di promuovere il contenuto da una fase (**Sviluppo**) alla successiva (**Test**, **Produzione**) con una logica predefinita e, se necessario, automatizzata tramite API.|

## 2. Implementare il Controllo della Versione e l'Integrazione con Git

L'integrazione con **Git** è il meccanismo primario in Fabric per il **controllo della versione** (Version Control) e la **collaborazione** tra sviluppatori.

### 2.1 Controllo della Versione a Livello di Area di Lavoro

L'integrazione di Git in Fabric opera a livello di **Area di Lavoro (Workspace)**. 
Ciò significa che un'intera Area di Lavoro viene associata a un repository e a un ramo Git specifico.

| Funzionalità            | Dettagli                                                                                                                                                                                                                                                                                                                                                      |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Provider Supportati** | **Azure DevOps** (Servizi Cloud) e **GitHub** (Cloud e Enterprise Cloud). L'uso delle versioni cloud è cruciale.                                                                                                                                                                                                                                              |
| **Processo**            | L'integrazione serializza il contenuto degli elementi di Fabric (come notebook, pipeline, definizioni di processi Spark, Lakehouse) in un formato leggibile e tracciabile (spesso JSON) all'interno del repository Git.                                                                                                                                       |
| **Vantaggi**            | **Backup e Cronologia**: Tracciamento di ogni modifica con possibilità di ripristinare versioni precedenti. **Collaborazione**: Uso di rami Git standard per lavorare in isolamento e unire il codice tramite **Pull Request (PR)**.  **Struttura Conservata**: La struttura dell'Area di Lavoro (comprese le sottocartelle) viene preservata nel repository. |

### 2.2 Flusso di Lavoro Standard di Git in Fabric

1. **Preparazione:** Assicurati di avere una **Capacità Fabric** e i prerequisiti Git (account Azure DevOps o GitHub).
2. **Connessione:** Dalle **Impostazioni dell'Area di Lavoro**, collega l'Area di Lavoro al tuo repository Git, specificando l'**Organizzazione**, il **Repository** e il **Ramo** di destinazione (spesso il ramo di sviluppo).
3. **Sincronizzazione Iniziale:** Fabric sincronizza il contenuto tra l'Area di Lavoro e il ramo Git, garantendo che abbiano lo stesso contenuto.
4. **Sviluppo e Commit:**
    - Apporta le modifiche agli elementi nell'Area di Lavoro.
    - Seleziona l'icona **Controllo del Codice Sorgente** (Source Control).
    - Visualizza gli elementi **modificati** (modified), **nuovi** (new), **conflitti** (conflict) o **eliminati** (deleted).
    - Seleziona gli elementi desiderati, aggiungi un **Commento** e premi **Esegui Commit** per sincronizzare le modifiche con il ramo Git.
5. **Aggiornamento:**
    - Quando altri sviluppatori eseguono il commit o le modifiche vengono unite al ramo Git connesso, vedrai **Aggiornamenti** disponibili nell'icona Controllo del Codice Sorgente.
    - Seleziona **Aggiorna tutto** per eseguire il pull (scaricare) le modifiche dal repository nell'Area di Lavoro.

### 2.3 Scenari di Diramazione (Branching) per lo Sviluppo Isolato

È fondamentale lavorare su un ambiente isolato per non influenzare il lavoro degli altri.

|Approccio|Strategia di Sviluppo|Vantaggi / Intuizione|
|---|---|---|
|**Area di Lavoro e Ramo Separati (Consigliato)**|1. **Sviluppo:** Collega una prima Area di Lavoro (**Dev Sandbox**) al **Ramo Principale** (es. `main` o `dev`). 2. **Diramazione (Branching):** Usa la funzionalità **Controllo del Codice Sorgente** in Fabric per **Diramare in una Nuova Area di Lavoro** (Branch to New Workspace). Questo crea un nuovo ramo Git (es. `feature-X`) e una nuova Area di Lavoro (**Dev Feature X**) isolata. 3. **Lavora e Commit:** Esegui le modifiche in _Dev Feature X_ e i commit nel ramo `feature-X`.|Offre un ambiente di authoring completamente isolato (Area di Lavoro dedicata) e un ramo Git dedicato. Ideale per la collaborazione in team.|
|**Strumenti Client e Clonazione Locale**|1. **Clona Localmente:** Clona il repository Git sul tuo computer locale (es. con VS Code per i Notebook). 2. **Sviluppa Localmente:** Lavora sulle modifiche nel tuo ramo isolato a livello locale. 3. **Push e Test:** Esegui il **push** delle modifiche nel ramo remoto. Collega un'Area di Lavoro di test separata al tuo ramo isolato per i test in Fabric. 4. **Merge:** Completa la **Pull Request** in Git.|Permette di usare strumenti di sviluppo locali avanzati e sfruttare le migliori pratiche standard di Git.|

> **Processo Pull Request (PR):** Indipendentemente dal metodo di sviluppo, dopo aver completato le modifiche nel ramo isolato, devi creare una **Pull Request** nel sistema di controllo della versione (GitHub o Azure DevOps) per richiedere che le tue modifiche vengano esaminate e unite nel ramo principale.

## 3. Pipeline di Distribuzione di Microsoft Fabric (CD)

Le **Pipeline di Distribuzione** sono lo strumento nativo di Fabric per gestire il **Recapito Continuo (CD)**, consentendo di alzare di livello gli elementi di Fabric attraverso un ciclo di vita a più fasi.

### 3.1 Struttura e Componenti

Una Pipeline di Distribuzione è composta da **fasi (stages)** che corrispondono agli ambienti in cui viene testato e rilasciato il contenuto.

|Componente|Dettagli|
|---|---|
|**Fasi (Stages)**|Da un minimo di **due** a un massimo di **dieci** fasi. Le fasi tipiche sono: **Sviluppo** (Dev), **Test** (Test), **Produzione** (Prod). Il numero e i nomi delle fasi sono **permanenti** dopo la creazione.|
|**Aree di Lavoro Assegnate**|Ogni fase è associata a un'**Area di Lavoro** specifica. L'Area di Lavoro è il contenitore effettivo degli elementi in tale fase.|
|**Distribuzione (Deployment)**|L'atto di **copiare** il contenuto dalla fase **sorgente** (source) alla fase **destinazione** (target). Fabric identifica il contenuto esistente nella destinazione e lo **sovrascrive**.|

### 3.2 La Distribuzione e le Configurazione

Durante la distribuzione, è essenziale che le configurazioni specifiche dell'ambiente (come le stringhe di connessione, i percorsi dei dati o i parametri) vengano aggiornate.

- **Regole dei Parametri:** Per gli elementi che supportano i parametri (es. alcuni report o modelli semantici), puoi definire **Regole dei Parametri** per applicare automaticamente valori diversi in base alla fase di distribuzione (es. un parametro `DataSource` per il database di Test nella fase di Test e il database di Produzione nella fase di Produzione). I parametri usati per la ribindings devono essere di tipo **Testo**.
- **Autobinding:** Le pipeline tentano di **ripristinare automaticamente le connessioni** tra gli elementi distribuiti e i loro cloni esistenti nella fase di destinazione.

|Tipi di Distribuzione tramite API|Descrizione|
|---|---|
|**Deploy All**|Distribuisce tutto il contenuto nell'Area di Lavoro sorgente alla fase successiva con una singola chiamata API.|
|**Selective Deploy**|Permette di distribuire solo **elementi specifici** (es. un singolo Notebook o un Modello Semantico) selezionati per la promozione.|
|**Backward Deploy**|Distribuzione all'indietro (**solo per i nuovi elementi**): distribuisce gli elementi alla fase **precedente** (es. da Test a Dev). Funziona solo se gli elementi non esistono già nella destinazione.|


## 4. API REST di Microsoft Fabric per l'Automazione (CI/CD Avanzata)

Per un'automazione CI/CD completa che si integra con strumenti esterni come **Azure DevOps Pipelines** o **GitHub Actions**, si utilizzano le **API REST di Fabric**.

### 4.1 Automazione dell'Integrazione Git (CI)

Le API Git di Fabric consentono di eseguire a livello di codice le azioni disponibili nell'interfaccia utente:

- **`Commit to Git`**: Automatizza l'azione di eseguire il commit delle modifiche da un'Area di Lavoro a un ramo Git. Utile per pipeline di **build** che preparano il codice.
- **`Update From Git`**: Sincronizza (esegue il pull) le modifiche da un ramo Git all'Area di Lavoro. Utile per garantire che l'Area di Lavoro di Sviluppo sia sempre aggiornata con il ramo principale dopo un merge.

### 4.2 Automazione delle Pipeline di Distribuzione (CD)

Le API di Distribuzione di Fabric (in gran parte basate sulle API Power BI estese) consentono di orchestare la promozione tra fasi:

- **`Deploy Stage Content`**: L'API chiave per innescare la distribuzione del contenuto da una fase all'altra. Può distribuire tutti gli elementi o solo quelli selezionati.
- **Monitoraggio**: Le API si integrano con le **API per le Operazioni a Lunga Esecuzione (Long Running Operations)** per monitorare lo stato della distribuzione asincrona.
    - **`Get Operation State`**: Ottiene lo stato di avanzamento della distribuzione.
    - **`Get Operation Result`**: Ottiene i risultati estesi dopo il completamento.
- **Gestione della Pipeline**: Le API supportano operazioni **CRUD** (Create, Read, Update, Delete) per:
    - Creare e Eliminare Pipeline.
    - Assegnare e Rimuovere Aree di Lavoro alle fasi.
    - Ottenere informazioni e cronologia della distribuzione.

### 4.3 Prerequisiti per l'Automazione API

Per autenticare le chiamate API, è necessario:

- Un **Token Microsoft Entra** (precedentemente Azure AD) per il servizio Fabric, da utilizzare nell'header di autorizzazione della chiamata API (spesso ottenuto tramite un'**Entità Servizio** in scenari CI/CD, che deve avere le autorizzazioni adeguate).

> **Intuizione CI/CD Completa:** Un processo end-to-end tipico di CI/CD in Azure DevOps o GitHub Actions potrebbe includere:
> 
> 1. **Build Pipeline:** Innescata dal merge in `main` → Usa le API Git per **aggiornare l'Area di Lavoro di Sviluppo (Dev)** dal ramo `main`.
> 2. **Release Pipeline:** Innescata dal successo della Build → Usa l'API **`Deploy Stage Content`** per distribuire il contenuto dall'Area di Lavoro Dev all'Area di Lavoro Test.
> 3. Dopo i test automatizzati → Usa la stessa API per distribuire l'Area di Lavoro Test all'Area di Lavoro Produzione, spesso dopo un'approvazione manuale.

---

# CI/CD in Microsoft Fabric: Pipeline di Distribuzione e Integrazione Git (REST API)

I flussi di lavoro di **Integrazione Continua (CI)** e **Recapito Continuo (CD)** sono fondamentali per gestire in modo efficiente e controllato il ciclo di vita delle soluzioni dati in **Microsoft Fabric**. Fabric offre due strumenti principali per questo scopo: l'integrazione nativa con **Git (CI)** per il controllo della versione e le **Pipeline di Distribuzione (CD)** per la promozione del contenuto tra gli ambienti.

## Pipeline di Distribuzione in Fabric

Le Pipeline di Distribuzione sono lo strumento di **Recapito Continuo (CD)** nativo di Fabric, che consente di promuovere gli elementi di Fabric (come Lakehouse, Data Pipeline, Semantic Model, Report) attraverso ambienti dedicati.

> [!Definition]
> 
> Una Pipeline di Distribuzione in Fabric mappa le aree di lavoro (Workspaces) a diverse fasi del ciclo di vita del contenuto (tipicamente Sviluppo, Test e Produzione), automatizzando la copia dei metadati degli elementi tra queste fasi.

### Fasi Chiave del Processo

Le pipeline garantiscono che il contenuto sia:

1. **Sviluppato** (Development): L'ambiente in cui gli sviluppatori creano e modificano gli elementi.
2. **Testato** (Test): L'ambiente in cui vengono eseguiti test di integrazione e di convalida prima del rilascio.
3. **Prodotto** (Production): L'ambiente in cui gli utenti finali interagiscono con il contenuto stabile e convalidato.

---

## Creazione e Utilizzo di una Pipeline (UI)

La creazione e l'utilizzo base di una Pipeline di Distribuzione avvengono direttamente nell'interfaccia utente di Fabric.

### Creazione della Pipeline

1. Seleziona l'icona **Aree di lavoro** nel riquadro di spostamento a sinistra.
2. Seleziona **Pipeline di distribuzione** e poi **Nuova pipeline**.
3. Assegna un **Nome** alla pipeline.
4. **Definisci e nomina le Fasi** (es. Sviluppo, Test, Produzione).
5. **Assegna un'Area di Lavoro** ad ogni fase (l'area di lavoro deve esistere e non essere già assegnata a un'altra fase).

### Distribuzione del Contenuto (Promozione)

Il processo di distribuzione copia gli elementi dall'Area di Lavoro sorgente (es. Sviluppo) all'Area di Lavoro di destinazione (es. Test).

1. Nel riquadro della Pipeline, seleziona la fase di destinazione (es. **Test**).
2. Seleziona la fase sorgente (es. **Sviluppo**) nel menu a discesa **Distribuisci da**.
3. Premi il pulsante **Distribuisci**.

> [!Nota]
> 
> Durante la distribuzione, vengono copiati solo i metadati (la definizione e la configurazione dell'elemento, es. un report o una Data Pipeline). I dati effettivi non vengono copiati.

---

## Integrazione tra Pipeline di Distribuzione e Git

L'approccio più robusto per la CI/CD in Fabric combina l'uso delle **Pipeline di Distribuzione** (per il CD tra ambienti) con l'**Integrazione Git** (per il CI e il controllo della versione).

> [!Tip]
> 
> Un modello di CI/CD comune in Fabric prevede che l'Area di Lavoro di Sviluppo sia l'unica direttamente connessa a un repository Git. Le Pipeline di Distribuzione promuovono quindi il contenuto da Sviluppo a Test e Produzione.

### Flusso Operativo Misto

1. **Imposta la Pipeline:** Crea la Pipeline di Distribuzione e assegna le aree di lavoro alle fasi (Sviluppo ![](data:,) Test ![](data:,) Produzione).
2. **Configura Git:** Nelle **Impostazioni dell'Area di Lavoro** di **Sviluppo**, connettila a un repository Git (es. un ramo `dev`).
3. **Sviluppo e Commit:**
    - Le modifiche vengono apportate nell'Area di Lavoro **Sviluppo**.
    - Utilizza la sezione **Controllo del codice sorgente** nell'Area di Lavoro Sviluppo per fare il **Commit** delle modifiche al ramo `dev` del repository Git.
4. **Promozione (CD):**
    - Utilizza il pulsante **Distribuisci** nella Pipeline per spostare il contenuto da Sviluppo a Test.
    - Il contenuto (metadati) viene clonato nell'Area di Lavoro **Test**.
5. **Sincronizzazione di Ritorno a Git (Passaggio Manuale):**
    - Quando si usa la promozione della pipeline (es. Sviluppo)








 Test), il contenuto viene aggiornato nell'Area di Lavoro di destinazione (Test), ma **il repository Git associato al ramo di Test NON viene aggiornato automaticamente**.
    - Devi accedere all'Area di Lavoro **Test** (o Produzione) e usare la funzione **Controllo del codice sorgente** per eseguire il **Commit** (o la Sincronizzazione) dell'Area di Lavoro al suo ramo Git associato (es. ramo `test`).

> [!Warning]
> 
> La distribuzione della pipeline sposta il contenuto in Fabric, ma NON propaga il commit nel repository Git della fase di destinazione. La sincronizzazione (commit) dall'area di lavoro al Git associato è un passaggio manuale o da automatizzare con le API.

---

## Automatizzare CI/CD con le API REST di Fabric

L'uso diretto dell'interfaccia utente introduce passaggi manuali (come la sincronizzazione Git nelle fasi Test/Prod) che possono essere eliminati automatizzando il processo tramite le **API REST di Fabric**.

> [!Definition]
> 
> API REST (Representational State Transfer) è un'architettura software che consente ai sistemi di comunicare su Internet. Le API REST di Fabric permettono l'interazione programmatica con il servizio.

### Vantaggi dell'Automazione tramite API

- **Coerenza:** Automatizza processi ripetitivi con esecuzione identica ad ogni ciclo.
- **Integrazione:** Permette l'integrazione con strumenti di orchestrazione esterni come **Azure DevOps Pipelines** o **GitHub Actions**.
- **Controllo Totale:** Abilita flussi di lavoro più complessi, come la modifica delle connessioni dati specifiche per l'ambiente (regole di distribuzione) dopo la promozione.

### Funzioni Chiave delle API REST per CI/CD

Le API REST di Fabric sono suddivise in categorie per gestire l'integrazione Git e le Pipeline di Distribuzione.

#### API REST per l'Integrazione Git (`/git`)

Queste API sono essenziali per il lato **Integrazione Continua (CI)**:

|Funzione API|Descrizione|
|---|---|
|`Commit to Git`|Esegue il commit delle modifiche apportate nell'area di lavoro al ramo remoto Git connesso (sincronizzazione **Area di Lavoro ![](data:,) Git**).|
|`Update From Git`|Aggiorna l'area di lavoro con i commit di cui è stato fatto il push nel ramo connesso (sincronizzazione **Git ![](data:,) Area di Lavoro**).|
|`Get Status`|Restituisce lo stato di sincronizzazione, mostrando quali elementi hanno modifiche in ingresso (da Git) e quali hanno modifiche non ancora sottoposte a commit (nell'Area di Lavoro).|

#### API REST per le Pipeline di Distribuzione (`/deployment-pipelines`)

Queste API sono essenziali per il lato **Recapito Continuo (CD)** e l'orchestrazione del rilascio:

|Funzione API|Descrizione|
|---|---|
|`List Deployment Pipeline Stages`|Ottiene l'elenco delle fasi e dei relativi ID di un'area di lavoro, utile per costruire le chiamate di distribuzione successive.|
|`Deploy Stage Content`|**Distribuisce il contenuto** dalla fase specificata alla fase successiva della pipeline. Consente la distribuzione di **tutti gli elementi** o solo di **elementi selezionati (Selective Deploy)**.|
|`Create pipeline`|Crea una nuova pipeline di distribuzione a livello programmatico.|
|`Assign workspace`|Assegna un'area di lavoro a una specifica fase della pipeline.|

### Intuizione Avanzata: Il Flusso di Lavoro Automato

Un flusso CI/CD completamente automatizzato in Azure DevOps o GitHub Actions utilizza le API REST di Fabric per orchestrare i processi:

1. **CI Trigger:** Una **Pull Request (PR)** viene approvata e unita al ramo principale del repository Git (es. da `feature` a `dev`).
2. **Build Pipeline (Orchestrazione):**
    - La pipeline di build esterna (es. Azure DevOps) viene avviata.
    - Chiama l'API **Git - Update From Git** sull'Area di Lavoro **Sviluppo** per assicurarsi che sia aggiornata con il codice unito.
3. **CD Trigger:** Viene avviata la pipeline di rilascio.
    - **Distribuzione a Test:** Chiama l'API **Deployment Pipelines - Deploy Stage Content** per promuovere il contenuto da **Sviluppo ![](data:,) Test**.
    - **Sincronizzazione Git Test:** Chiama l'API **Git - Commit to Git** sull'Area di Lavoro **Test** per sincronizzare lo stato di Fabric con il ramo Git `test`.
4. **Promozione a Produzione:** Dopo test e approvazioni, si ripete il processo: **Deploy Stage Content** da **Test ![](data:,) Produzione**, seguito da **Commit to Git** sull'Area di Lavoro **Produzione** per sincronizzare il ramo `prod`.

> [!Tip]
> 
> Per interagire con le API REST di Fabric è necessario ottenere un Token di Accesso Microsoft Entra (precedentemente Azure AD Token), che deve essere incluso nell'header di autorizzazione di ogni chiamata API.
> 
> **Esempio URI per l'esecuzione di una Pipeline di Dati (On-Demand):**
> 
> ```
> POST https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/items/{itemId}/jobs/instances?jobType=Pipeline
> ```
> 
> (Dove `{itemId}` è l'ID della tua Data Pipeline).

---

## Prerequisiti

Per utilizzare l'integrazione Git e le Pipeline di Distribuzione:

- È necessario un account tenant di **Microsoft Fabric** con una **Fabric capacity** attiva.
- È necessario essere un **Amministratore** dell'Area di Lavoro Fabric.
- L'integrazione Git richiede un repository in **Azure Repos** o **GitHub**.

