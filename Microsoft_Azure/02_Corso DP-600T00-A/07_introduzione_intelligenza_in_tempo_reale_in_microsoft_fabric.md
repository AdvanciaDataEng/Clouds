---
autore: Kevin Milli
---

## 1. Il Paradigma dell'Analisi in Tempo Reale

### 1.1 Definizione e Necessità

L'**Analisi in Tempo Reale** (Real-Time Analytics - RTA) è la disciplina che si occupa dell'elaborazione, analisi e reazione ai dati nel momento stesso in cui vengono generati o subito dopo, tipicamente con una latenza che varia da pochi millisecondi a pochi secondi.

Contrariamente all'**Elaborazione Batch** (tradizionale), che analizza _snapshot statici_ di dati storici archiviati (spesso durante la notte), l'analisi in tempo reale opera sui **dati in movimento** (_data in motion_).

> **Intuizione: "Quasi in Tempo Reale"**
> 
> In pratica, l'RTA è quasi sempre definita **Near Real-Time** (NRT) o **Quasi in Tempo Reale**. Questo perché un certo grado di latenza (dovuta all'acquisizione, alla rete e all'elaborazione) è inevitabile. L'obiettivo non è latenza zero, ma una **latenza sufficientemente bassa** (spesso inferiore al secondo) da consentire una risposta **azionabile** e **tempestiva** all'evento che si è verificato.

#### Esempio Pratico: Logistica (Scenario Iniziale)

|Tipo di Analisi|Obiettivo e Latenza|Azione Risultante|
|---|---|---|
|**Batch**|Reportistica storica (ritardi di ieri). Latenza: **Ore/Giorni**.|Pianificazione strategica a lungo termine.|
|**Real-Time**|Monitoraggio GPS e sensori (ritardi attuali). Latenza: **Secondi**.|Notifica automatica al cliente e ri-routing del veicolo in corso.|

### 1.2 Eventi, Flussi e Architettura Event-Driven

Alla base dell'RTA ci sono due concetti fondamentali:

#### 1.2.1 Eventi (Events)

Un **Evento** è un record immutabile e atomico che documenta un _fatto_ o un _cambiamento_ avvenuto in un sistema in un preciso momento.

- **Caratteristiche:** Immutabile, sequenziale, timestamped.
- **Esempi:** Un click su un sito web, una lettura di un sensore (temperatura), una transazione completata, il log di un errore applicativo.

#### 1.2.2 Flussi (Streams)

Un **Flusso di Eventi** (Event Stream) è una sequenza continua, illimitata e ordinata cronologicamente di eventi. È il _meccanismo di consegna_ che trasporta gli eventi dal punto di generazione al punto di elaborazione.

- **Caratteristiche:** Continuo (non ha inizio né fine), ordinato per tempo, gestito da un **Broker di Messaggi** (ad esempio, Apache Kafka, che è spesso la base tecnologica).
- **Intuizione:** Pensa a un flusso come a un **nastro trasportatore** di eventi, dove ogni pacchetto è un evento che viene depositato e consumato in sequenza.

---

## 2. Architettura delle Soluzioni Real-Time

Per operare con i dati in movimento, qualsiasi soluzione di RTA, compresa Real-Time Intelligence di Microsoft Fabric, si basa su un framework a più livelli:

### 2.1 Componenti Generici Essenziali

|Componente|Funzione Principale|Strumenti (Esempi Generici)|
|---|---|---|
|**Inserimento (Ingestion)**|Raccogliere dati ad alto volume e alta velocità da fonti eterogenee (IoT, log, API, CDC).|Event Hubs, Kafka, Kinesis.|
|**Elaborazione del Flusso (Stream Processing)**|Trasformare, filtrare, aggregare e arricchire i dati _mentre_ sono in transito. Rileva modelli e anomalie.|Spark Streaming, Azure Stream Analytics.|
|**Archiviazione a Bassa Latenza**|Memorizzare il flusso di dati in un formato ottimizzato per query veloci e analisi temporali.|Database NoSQL, KQL Databases (Time Series).|
|**Presentazione & Visualizzazione**|Dashboard interattive che si aggiornano automaticamente per mostrare lo stato attuale (live).|Grafana, Real-Time Dashboards.|
|**Processo Decisionale Automatizzato**|Attivare risposte automatiche (alert, trigger) basate su condizioni o modelli rilevati nel flusso.|Motori di regole, Funzioni serverless.|

---

## 3. Real-Time Intelligence in Microsoft Fabric

**Real-Time Intelligence (RTI)** in Microsoft Fabric è una soluzione _Software as a Service_ (SaaS) end-to-end che unifica tutti i componenti architetturali necessari per gestire e analizzare i dati di streaming.

RTI si basa sul concetto di **Eventhouse**, che è un contenitore logico all'interno di Fabric progettato per ospitare i dati di streaming.

### 3.1 I Componenti Chiave di Fabric RTI

|Componente Fabric|Categoria Architetturale|Descrizione e Ruolo Avanzato|
|---|---|---|
|**Real-Time Hub**|Individuazione / Catalogo|Il punto centrale per la gestione e l'individuazione di tutte le fonti dati di streaming (event streams) e delle destinazioni in Fabric.|
|**Eventstream**|Inserimento e Processamento|Il motore **no-code** per l'acquisizione, la trasformazione _leggera_ (filtering, manage fields, routing) e l'instradamento degli eventi verso le destinazioni supportate.|
|**Eventhouse**|Archiviazione (Contenitore)|Il contenitore logico all'interno del workspace Fabric, ottimizzato per dati ad alto volume, telemetria, log ed eventi (Time Series).|
|**KQL Database**|Archiviazione / Querying|Il cuore dell'Eventhouse. È un database ad alte prestazioni basato sull'engine di Azure Data Explorer, ottimizzato per l'analisi di dati in tempo reale mediante il **Kusto Query Language (KQL)**.|
|**KQL Queryset**|Analisi / Esplorazione|Interfaccia per eseguire query KQL ad hoc sul KQL Database e visualizzare rapidamente i risultati prima di creare un dashboard.|
|**Real-Time Dashboard**|Presentazione / Visualizzazione|Dashboard interattivi che si aggiornano in tempo reale tramite query KQL continue. Forniscono una visualizzazione _up-to-the-second_ dello stato aziendale.|
|**Activator**|Processo Decisionale Automatizzato|Funzionalità **no-code** per definire regole e condizioni sui dati in movimento o su dati memorizzati (KQL Database) e attivare automaticamente azioni (e.g., inviare un'email, avviare un flusso Power Automate, notificare un sistema esterno).|

### 3.2 Linguaggio Kusto Query Language (KQL)

Il KQL è il linguaggio di query primario utilizzato nel KQL Database.

> **Insight su KQL**
> 
> KQL è progettato specificamente per interrogare grandi volumi di dati di log e telemetria. La sua sintassi è estremamente potente e intuitiva, utilizzando l'operatore pipeline (`|`) per concatenare operazioni (filtri, aggregazioni, join, ecc.), rendendolo ideale per l'analisi di serie temporali (time series) e per il rilevamento di pattern nel flusso.

### 3.3 Utilizzo e Vantaggi Strategici

L'adozione di Real-Time Intelligence trasforma la reattività aziendale:

1. **Risposta Immediata alle Anomalie:** Permette il rilevamento istantaneo di frodi finanziarie, guasti IT o violazioni di sicurezza non appena si verificano, non ore dopo.
2. **Ottimizzazione Operativa Continua:** Regolare processi e risorse (e.g., assegnazione di personale, gestione dell'inventario) in base alle condizioni operative correnti, massimizzando l'efficienza.
3. **Miglioramento dell'Esperienza Cliente (CX):** Offrire interazioni personalizzate e contestuali, come raccomandazioni di prodotto in tempo reale o aggiornamenti di consegna proattivi.
4. **Automazione Intelligente:** Sfruttare l'**Activator** per far sì che il sistema non solo _veda_ un problema (es. temperatura alta), ma _agisca_ autonomamente per risolverlo (es. spegnere il macchinario o inviare un avviso di manutenzione) senza intervento umano.

---

# 4. Intelligenza in Tempo Reale in Microsoft Fabric: Dettagli Applicativi

Man mano che le organizzazioni generano volumi crescenti di dati basati su eventi, la possibilità di elaborare, analizzare e agire sui dati in movimento diventa essenziale per il vantaggio competitivo. **Real-Time Intelligence (RTI)** in Microsoft Fabric offre un set integrato di funzionalità end-to-end per l'uso dei dati di streaming con una latenza minima.

## 4.1. Casi d'Uso (Use Cases) e Capacità di Risposta

A differenza dei sistemi batch che elaborano i dati a intervalli pianificati, Real-Time Intelligence consente di rispondere agli eventi man mano che si verificano, offrendo informazioni dettagliate _quasi_ in tempo reale.

|Tipo di Dati/Scenario|Esempio Pratico|Azione Guidata da RTI|
|---|---|---|
|**Tracciamento Logistica**|Monitorare le posizioni dei veicoli per avvisare i clienti in caso di ritardi.|**Activator** invia automaticamente un messaggio al cliente e notifica il centro operativo.|
|**Monitoraggio Apparecchiature (IoT)**|Tenere traccia della temperatura della turbina per evitare guasti costosi.|**KQL Queryset** rileva un'anomalia di temperatura e **Activator** spegne il macchinario.|
|**Rilevamento Frodi Finanziarie**|Analizzare i modelli di acquisto in tempo reale per bloccare transazioni sospette.|**Eventstream** filtra i pattern ad alto rischio e attiva un blocco immediato tramite un **Flusso Power Automate**.|
|**Prestazioni Web (Clickstream)**|Monitorare i tempi di caricamento delle pagine per migliorare l'esperienza utente.|**Real-Time Dashboard** visualizza un picco di latenza in una regione specifica, consentendo un intervento immediato sul server.|
|**Integrità del Sistema (Log)**|Tenere traccia degli errori applicativi per mantenere l'affidabilità del servizio.|Il **KQL Database** indicizza i log in tempo reale, permettendo al team DevOps di eseguire query KQL per la _Root Cause Analysis_ in pochi secondi.|

## 4.2. I Componenti Fondamentali di Fabric RTI

Microsoft Fabric Real-Time Intelligence è un set integrato di componenti che interagiscono per gestire lo streaming dei dati dall'acquisizione tramite risposta automatica.

### A. Inserimento ed Elaborazione: Eventstream

**Eventstream** è il motore di acquisizione e trasformazione _leggera_ dei dati in movimento.

- **Funzione:** Acquisisce i dati in streaming da varie origini (Azure Event Hubs, Custom App, Kafka, ecc.) e applica trasformazioni elementari in tempo reale _durante_ il flusso dei dati.
- **Capacità Chiave:**
    - **No-Code Processing:** Permette di **filtrare** (rimuovere eventi irrilevanti), **arricchire** (aggiungere campi, ad esempio un timestamp di sistema), e **trasformare** (modificare il nome dei campi) i dati prima dell'archiviazione.
    - **Routing:** Instrada lo stesso flusso a destinazioni diverse, ad esempio, inviare i dati di log completi a un database KQL e solo gli avvisi critici a un **Activator**.

### B. Archiviazione e Analisi: Eventhouse e KQL Database

Real-Time Intelligence archivia i dati nei database KQL (Kusto Query Language) all'interno degli **Eventhouses**.

- **Eventhouse:** Funge da **contenitore logico** o infrastruttura nel workspace Fabric, specificamente ottimizzato per gestire i carichi di lavoro di streaming, log e telemetria.
- **KQL Database:** È il database transazionale ad alte prestazioni all'interno dell'Eventhouse. È progettato per l'**inserimento rapido** (high-velocity ingestion) e le **query ad alta concorrenza** sui dati di serie temporali.
    - **Integrazione OneLake:** L'archiviazione sottostante sfrutta **OneLake**, il _Data Lake_ di Fabric, rendendo i dati di streaming immediatamente disponibili per altri carichi di lavoro (come Data Science o Power BI) senza movimentazione.
    - **Linguaggio KQL:** L'analisi primaria avviene tramite il **Kusto Query Language**, che eccelle nelle query di log, serie temporali e rilevamento di pattern temporali.

### C. Querying e Visualizzazione: KQL Queryset e Real-Time Dashboard

- **KQL Queryset:** Offre un ambiente interattivo per **eseguire e gestire le query KQL** sui KQL Database.
    - **Flessibilità:** Consente di salvare query per uso futuro, organizzare schede di query e supporta la sintassi **T-SQL** per gli utenti che preferiscono lo standard SQL.
    - **Obiettivo:** È l'area di lavoro per l'esplorazione dati e la _Root Cause Analysis_ (analisi della causa radice).
- **Real-Time Dashboard:** Sono il livello di presentazione. Si connettono direttamente al KQL Database e vengono **aggiornati automaticamente e continuamente** man mano che arrivano nuovi dati.
    - **Funzione:** Monitorano sia le condizioni correnti (_stato live_) sia le tendenze storiche con query ottimizzate.

### D. Azione e Automazione: Activator

**Activator** è lo strumento che abilita il **Processo Decisionale Automatizzato**.

- **Funzione:** Monitora continuamente i dati di streaming (o i risultati di query sul KQL Database) in base a **regole e soglie** definite dall'utente.
- **Esecuzione Azioni:** Quando le condizioni vengono soddisfatte (ad esempio, "se il numero di errori supera 100/minuto"), Activator può:
    - Inviare notifiche (e-mail, Teams).
    - Attivare flussi di lavoro in **Power Automate**.
    - Eseguire pipeline di dati o notebook di Fabric per un'elaborazione più complessa (es. richiamare un modello ML).

## 4.3. L'Hub Real-Time (Real-Time Hub): Il Catalogo Dati in Movimento

**L'Hub Real-Time** è la posizione centrale e unificata in Fabric per l'**individuazione e la gestione** di tutti i dati in movimento a cui si ha accesso.

### A. Ruolo Strategico

L'Hub Real-Time non è solo un elenco, ma il **catalogo dati di streaming** dell'organizzazione. Semplifica l'inserimento e la sottoscrizione ai dati di streaming, fungendo da base per il processo decisionale guidato dagli eventi.

### B. Organizzazione dei Dati in Movimento

L'hub organizza i dati in movimento in categorie chiare:

1. **Origini Dati (Data Sources):** Permette di connettersi e configurare feed di dati in ingresso da diverse origini, incluse sorgenti Microsoft, feed **Change Data Capture (CDC)** dai database e origini esterne (es. S3, GCS).
2. **Origini di Azure (Azure Sources):** Offre connettori rapidi per servizi Azure chiave come **Hub IoT di Azure**, **Bus di Servizio di Azure** e **Azure Data Explorer**.
3. **Eventi di Fabric (Fabric Events):** Eventi di sistema generati _all'interno_ di Fabric, come cambiamenti nello stato dei processi, azioni su file/cartelle in **OneLake** o modifiche agli elementi del workspace.
4. **Eventi di Azure (Azure Events):** Eventi di sistema generati dai servizi Azure, come le azioni su file nell'Archiviazione BLOB di Azure.

> **Flusso Operativo nell'Hub**
> 
> 1. **Individuazione:** Individui un flusso di eventi esistente nell'Hub.
> 2. **Esplorazione:** Selezioni il flusso e navighi al **KQL Database** sottostante per eseguire query approfondite.
> 3. **Azione:** Crei risposte automatizzate usando le regole di **Activator** che monitorano quel flusso per pattern specifici, attivando azioni downstream.

L'accesso all'Hub in tempo reale (tramite l'icona dedicata nella barra dei menu Fabric) garantisce che l'intera organizzazione possa visualizzare e sottoscrivere cosa sta accadendo _quasi in tempo reale_.

---

# 5. Inserimento e Trasformazione dei Dati in Tempo Reale

Real-Time Intelligence in Microsoft Fabric offre flessibilità nell'acquisizione dei dati di streaming, supportando sia l'elaborazione _in transito_ che l'elaborazione _post-inserimento_. Esistono due approcci principali per l'ingestione di dati di streaming: utilizzare **Eventstream** o inserire direttamente i dati in un **KQL Database**.

## 5.1. Approccio 1: Inserimento e Trasformazione tramite Eventstream

**Eventstream** è la pipeline di acquisizione _low-code/no-code_ che gestisce il flusso e la manipolazione iniziale degli eventi prima della loro archiviazione finale.

### 5.1.1. Architettura Concettuale del Flusso di Eventi

Un Eventstream è concettualmente un sistema a tre stadi, come un sistema di tubazioni:

1. **Origini (Sources):** Il "rubinetto" da cui l'acqua (i dati) entra nel sistema.
2. **Trasformazioni (Transformations):** I "filtri" e i "miscelatori" lungo il percorso.
3. **Destinazioni (Destinations):** Il "secchio" o "serbatoio" in cui i dati elaborati vengono raccolti per l'analisi.

### 5.1.2. Dettagli sulle Origini Dati

Un Eventstream può connettersi a un'ampia gamma di sorgenti dati in movimento:

|Tipo di Origine|Esempi Chiave e Contesto|Intuizione Pratica|
|---|---|---|
|**Origini Microsoft**|**Azure Event Hubs, Azure IoT Hub, Azure Service Bus, CDC (Change Data Capture)** dai database.|Fonti ad alto throughput da servizi cloud esistenti. Il **CDC** è cruciale per catturare le modifiche ai database transazionali e trasformarle in flussi di eventi.|
|**Origini Non-Microsoft**|**Apache Kafka, Google Cloud Pub/Sub, MQTT** (protocollo leggero per IoT).|Garantisce l'interoperabilità e la possibilità di connettersi a ecosistemi on-premises o multi-cloud.|
|**Eventi di Fabric & Azure**|Eventi di sistema (es. modifiche agli item, alterazioni in **OneLake**, eventi di Storage BLOB).|Utilizza l'architettura _event-driven_ interna a Fabric per monitorare e reagire alle attività del proprio ambiente dati.|

### 5.1.3. Trasformazioni degli Eventi _In-Flight_

L'elaborazione nel flusso di eventi avviene **in transito** (durante l'elaborazione del flusso), prima che i dati raggiungano la destinazione finale. Queste trasformazioni sono ideali per manipolazioni veloci e cruciali:

- **Filtro (Filtering):** Rimuove eventi irrilevanti o ridondanti (e.g., eliminare tutti i log di "livello informativo").
- **Gestione Campi (Manage Fields):** Aggiungere, rinominare, rimuovere o espandere campi (e.g., aggiungere un timestamp di sistema).
- **Aggregazione/Raggruppamento (Aggregation/Group By):** Riepilogare i dati in finestre temporali definite (e.g., contare il numero di click ogni 5 secondi).
- **Join (Opzionale):** Unire il flusso di eventi con dati di riferimento statici per arricchire l'evento (e.g., unire un ID sensore con un nome di modello statico).

### 5.1.4. Destinazioni Dati (Sinks)

Le destinazioni sono i punti di raccolta e utilizzo che consentono di preservare e agire sul valore dei dati di streaming:

- **KQL Database (in Eventhouse):** La destinazione più comune per l'analisi a bassa latenza e ad alta velocità.
- **Lakehouse:** Per l'archiviazione a lungo termine e l'analisi batch (livello Bronze di un Medallion Architecture).
- **Flusso Derivato (Derived Stream):** Creazione di un nuovo Eventstream con dati già filtrati o aggregati per altri consumatori.
- **Activator:** Per innescare risposte immediate basate sul flusso di dati.
- **Endpoint Personalizzato:** Per l'integrazione con sistemi esterni.

## 5.2. Approccio 2: Inserimento Diretto nel KQL Database

L'alternativa è bypassare la trasformazione in-flight di Eventstream e inserire i dati direttamente nel **KQL Database** all'interno dell'Eventhouse.

### 5.2.1. Metodi di Inserimento Diretto

L'inserimento diretto (tramite l'opzione **Recupera dati** o **connettori**) supporta il caricamento di dati sia in tempo reale che batch:

- **Streaming (Real-Time):** Da servizi come Hub eventi di Azure.
- **Batch/File:** Da Archiviazione di Azure (BLOB), Amazon S3, OneLake (file locali), o file da un'unità locale.

### 5.2.2. Trasformazione Post-Inserimento: Criteri di Aggiornamento

Quando i dati vengono inseriti direttamente, la trasformazione non avviene _prima_ di arrivare, ma _dopo_. Questo meccanismo si basa sui **Criteri di Aggiornamento (Update Policies)**:

- **Meccanismo:** Sono meccanismi di automazione definiti su una tabella. Vengono attivati **automaticamente** ogni volta che nuovi dati vengono scritti nella tabella di origine (Source Table).
- **Processo:**
    1. I dati non elaborati arrivano nella **Tabella di Origine**.
    2. Il Criterio di Aggiornamento si attiva, eseguendo una **Query KQL** (spesso complessa) sui dati appena inseriti.
    3. Il risultato della query trasformata viene salvato in una **Tabella di Destinazione** (Target Table).

> **Differenza Critica: Eventstream vs. Criteri di Aggiornamento**
> 
> - **Eventstream Transformation:** Pre-inserimento. Ideale per filtri e manipolazioni semplici e veloci, riducendo il volume di dati prima dell'archiviazione.
> - **Criteri di Aggiornamento:** Post-inserimento. Ideale per trasformazioni complesse che richiedono la piena potenza di **KQL** (e.g., aggregazioni analitiche, arricchimento con dati storici già presenti), operando in modo asincrono.


I Criteri di Aggiornamento sono fondamentali per mantenere la **Schema Evolution** e separare i dati grezzi (Raw data) dai dati trasformati e pronti per l'analisi (Curated data).

---

# 6. Archiviare ed Eseguire Query sui Dati in Tempo Reale

I **KQL Database** all'interno di un'**Eventhouse** sono il cuore pulsante dell'archiviazione e dell'analisi in Fabric RTI. Sono progettati per ospitare dati ad alta velocità (telemetria, log, serie temporali) e rispondere a query con latenza minima.

## 6.1. Componenti dell'Eventhouse

L'Eventhouse è il contenitore logico che supporta il carico di lavoro di analisi in tempo reale, composto da:

- **KQL Database:** L'archivio dati ottimizzato per il real-time, che ospita la collezione di tabelle, funzioni archiviate, viste materializzate e collegamenti.
- **KQL Queryset:** L'interfaccia interattiva che funge da area di lavoro per scrivere, salvare e gestire le query KQL o T-SQL sulle tabelle del KQL Database.

## 6.2. La Potenza del Kusto Query Language (KQL)

**KQL** è il linguaggio di query nativo e il vero punto di forza dei KQL Database.

### 6.2.1. Caratteristiche di KQL

- **Ottimizzazione per il Real-Time:** KQL è progettato per analizzare **grandi volumi** di dati strutturati, semistrutturati e non strutturati (perfetto per JSON e Log).
- **Architettura:** I KQL Database indicizzano i dati in ingresso **in base al tempo di inserimento** e li partizionano per garantire prestazioni di query ottimali su grandi finestre temporali.
- **Sintassi (Syntax):** La sua sintassi è **sequenziale e pipeline-based**, rendendola altamente leggibile e potente per l'analisi di flussi. Una query inizia con un nome di tabella seguito da operatori concatenati tramite la barra verticale (`|`).

### 6.2.2. Esempi di Sintassi KQL

Le query KQL sono composte da istruzioni che, in sequenza, **filtrano, trasformano, aggregano** o **uniscono** i dati.

|Operazione|Query KQL (Esempio)|Descrizione|
|---|---|---|
|**Visualizzazione**|`stock|take 10`|
|**Filtro e Aggregazione**|`stock \| where ["time"] > ago(5m) \| summarize avgPrice = avg(todouble(bidPrice)) by symbol \| project symbol, avgPrice`|Trova il prezzo medio (`avgPrice`) per simbolo (`symbol`) negli ultimi 5 minuti (`ago(5m)`), e proietta solo quei due campi.|

### 6.2.3. Automazione Avanzata con Comandi di Gestione

Oltre alle query di analisi, i **Comandi di Gestione** (Management Commands) automatizzano l'elaborazione dei dati e migliorano le performance:

- **Criteri di Aggiornamento (Update Policies):** Trasformano automaticamente i dati non elaborati e salvano il risultato trasformato in una tabella di destinazione separata, garantendo la separazione tra dati Raw e Curated.
- **Viste Materializzate (Materialized Views):** **Precalcolano e archiviano** i risultati di aggregazioni usate di frequente (e.g., aggregazione oraria). Questo accelera drasticamente le query ripetitive che altrimenti dovrebbero scansionare tutti i dati grezzi.
- **Funzioni Archiviate (Stored Functions):** Permettono di salvare e riutilizzare logica di query complessa in tutta l'organizzazione, garantendo coerenza nell'analisi.

### 6.2.4. Opzioni di Query Aggiuntive

- **T-SQL:** Il KQL Database supporta un **subset** del linguaggio **Transact-SQL** per i professionisti dei dati che preferiscono la sintassi SQL tradizionale (e.g., `SELECT TOP 10 * FROM stock;`).
- **Copilot per Real-Time Intelligence:** Utilizza l'intelligenza artificiale per comprendere le richieste in linguaggio naturale e **generare automaticamente il codice KQL** o T-SQL corrispondente, facilitando notevolmente la scrittura di query.

# 7. Visualizzare e Agire sui Dati in Tempo Reale

L'analisi in tempo reale culmina in una risposta o in una visualizzazione immediata che consente all'azienda di reagire.

## 7.1. Visualizzare con Real-Time Dashboard e Power BI

I **Real-Time Dashboard** consentono di assemblare visualizzazioni basate su query KQL in un'unica interfaccia visiva.

- **Riquadri (Tiles):** Ogni riquadro del dashboard è alimentato da una query KQL specifica e si aggiorna automaticamente in tempo reale.
- **Interattività:** I dashboard pubblicati consentono l'esplorazione interattiva (drill-down, filtri visivi) sui dati per comprendere rapidamente le tendenze.
- **Integrazione con Power BI:** È possibile sfruttare i connettori dedicati per creare report e dashboard **Power BI** direttamente sui dati del KQL Database, unendo la potenza di KQL con le capacità avanzate di Business Intelligence di Power BI.

## 7.2. Automatizzare Azioni con Activator

**Activator** è la tecnologia **no-code** di Microsoft Fabric che trasforma il rilevamento dei pattern in azioni aziendali automatizzate.

### 7.2.1. Concetti Chiave di Activator

L'automazione si basa sulla mappatura dei dati di streaming a entità aziendali logiche:

|Concetto|Definizione|Esempio (Monitoraggio Spedizioni)|
|---|---|---|
|**Evento**|Il record grezzo nel flusso (`{PackageID: '123', Status: 'Delayed'}`).|La singola notifica di ritardo del pacco.|
|**Oggetto**|Un'entità aziendale che si desidera monitorare.|Il **Pacco** (entità) con ID '123'.|
|**Proprietà**|Un campo mappato che rappresenta lo stato dell'Oggetto.|La proprietà `StatoConsegna` mappata al campo `Status`.|
|**Regola**|La condizione logica che innesca l'azione.|**Regola:** SE `StatoConsegna` = 'Delayed' PER PIÙ di 5 minuti, ALLORA...|

### 7.2.2. Funzionalità e Applicazioni di Activator

Activator monitora continuamente le **Regole** (soglie, deviazioni, anomalie) e, quando soddisfatte, attiva una risposta, realizzando un'architettura **event-driven** completa.

- **Casi d'Uso Esemplari:**
    - **Marketing in Tempo Reale:** Avviare una campagna di coupon quando il prezzo di un concorrente scende al di sotto di una soglia.
    - **Manutenzione Predittiva:** Inviare un ordine di lavoro (tramite Power Automate) quando la proprietà `Temperatura` di un sensore supera il 95%.
    - **Gestione dell'Inventario:** Inviare avvisi ai gestori dei negozi quando le vendite di un prodotto diminuiscono al di sotto di una soglia, evitando perdite (come nel caso del cibo deperibile).
    - **Esperienza Utente (UX):** Contrassegnare e notificare i team di sviluppo quando i tassi di errore in un'app superano una soglia critica.

Activator è lo strumento definitivo per qualsiasi scenario che richiede l'analisi _e_ l'azione automatizzata sui dati in tempo reale.
