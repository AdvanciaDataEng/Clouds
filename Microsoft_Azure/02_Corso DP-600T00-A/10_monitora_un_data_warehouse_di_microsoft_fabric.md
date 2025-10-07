---
autore: Kevin Milli
---

Un data warehouse è un asset aziendale cruciale per l'analisi e il reporting. Monitorarlo è fondamentale per **gestire i costi**, **ottimizzare le prestazioni delle query** e ottenere **informazioni sull'utilizzo** dei dati. Microsoft Fabric offre strumenti integrati per coprire questi aspetti.

## 1. Monitorare le Metriche di Capacità (Costi e Utilizzo)

In Microsoft Fabric, la **capacità** è un pool di risorse di calcolo fornite dalla licenza acquistata (misurate in **Unità di Capacità** o **CU**). Ogni attività nel tuo data warehouse (letture, scritture, query) consuma CU, influenzando direttamente i costi. Il monitoraggio della capacità è cruciale per la **gestione finanziaria** e l'**ottimizzazione delle risorse**.

### Lo Strumento: App Microsoft Fabric Capacity Metrics

L'**App Microsoft Fabric Capacity Metrics** è una soluzione Power BI predefinita che l'amministratore può installare per un monitoraggio approfondito. Fornisce dashboard visivi e tendenze storiche sull'utilizzo delle CU.

- **Scopo:** Tracciare il consumo di CU, identificare i carichi di lavoro più intensi e diagnosticare problemi di sovraccarico.
- **Filtro per Data Warehouse:** Per concentrarsi solo sulle attività del data warehouse, gli amministratori possono filtrare l'app per il tipo di elemento **"Warehouse"** o la categoria di carico di lavoro **"DMS"** (Data Management Services).
- **Metriche Chiave:**
    - **CU Utilization Trends (14 giorni):** Mostra i picchi di utilizzo e i pattern di consumo.
    - **Throttling (Limitazione):** Indica quando i processi superano i limiti di capacità acquistata, portando a ritardi (il che suggerisce la necessità di ottimizzare o di scalare la capacità).
    - **Storage Page (30 giorni):** Monitora l'utilizzo dello spazio di archiviazione.
    - **Timepoint Page:** Permette di fare il **drill-down** in specifici intervalli di 30 secondi per vedere quali operazioni (interattive o in background) hanno consumato più CU, ideale per diagnosticare i sovraccarichi e il comportamento di **autoscale** o **throttling**.

|Intuizione per l'Ottimizzazione|Spiegazione|
|---|---|
|**Debito di CU (Carryforward Debt)**|Quando la capacità si sovraccarica, il consumo in eccesso viene "addebitato" come debito (colore Rosso/Verde nell'app). Questo debito deve essere "bruciato" (Burndown - colore Blu) quando l'utilizzo scende sotto il 100%. Un elevato debito cumulativo indica una capacità insufficiente per i carichi di lavoro.|
|**Durata dei Dati**|L'app fornisce dati sulle prestazioni di calcolo per gli **ultimi 14 giorni** e i dati di archiviazione per gli ultimi **30 giorni**. Per un'analisi storica più lunga, è necessario estrarre e archiviare questi dati in modo indipendente (ad esempio, in un Lakehouse di Fabric).|

## 2. Monitorare l'Attività Corrente (DMV)

Per ottenere una visione in **tempo reale** delle attività in corso nel data warehouse, si utilizzano le **Viste a Gestione Dinamica (DMV - Dynamic Management Views)**, un insieme di viste di sistema specifiche per l'endpoint SQL e il Warehouse.

### Viste a Gestione Dinamica (DMV) di Riferimento

Queste DMV risiedono nello schema di sistema `sys` e sono fondamentali per il monitoraggio del ciclo di vita delle query:

1. **`sys.dm_exec_connections`**: Fornisce dettagli su tutte le **connessioni** stabilite tra il warehouse e il motore di esecuzione.
2. **`sys.dm_exec_sessions`**: Restituisce informazioni su ogni **sessione** autenticata (chi è l'utente, quando è iniziata).
3. **`sys.dm_exec_requests`**: Mostra informazioni su ogni **richiesta attiva** in una sessione (il comando T-SQL in esecuzione, il tempo trascorso).

|Ruoli e Permessi per le DMV|Azioni Eseguibili|
|---|---|
|**Admin** di Workspace|Può eseguire tutte e tre le DMV e vedere le informazioni di tutti gli utenti. Solo l'Admin può eseguire il comando **`KILL`** per terminare una query a esecuzione prolungata.|
|**Member, Contributor, Viewer**|Possono eseguire solo `sys.dm_exec_sessions` e `sys.dm_exec_requests` per vedere i _propri_ risultati.|

### Esempio Pratico: Identificazione di Query in Esecuzione

La seguente query T-SQL mostra le richieste attive (`status = 'running'`) nel database corrente (`requests.database_id = DB_ID()`), ordinate per tempo di esecuzione totale, aiutandoti a identificare immediatamente potenziali colli di bottiglia.
```SQL
SELECT 
    sessions.session_id, 
    sessions.login_name,
    connections.client_net_address,
    requests.command, 
    requests.start_time, 
    requests.total_elapsed_time,
    -- Aggiungendo questa colonna si ottiene l'ID della richiesta distribuita, utile per il join con altre viste (es. Query Insights)
    requests.distributed_statement_id 
FROM sys.dm_exec_connections AS connections
INNER JOIN sys.dm_exec_sessions AS sessions
    ON connections.session_id = sessions.session_id
INNER JOIN sys.dm_exec_requests AS requests
    ON requests.session_id = sessions.session_id
WHERE requests.status = 'running' 
    AND requests.database_id = DB_ID()
ORDER BY requests.total_elapsed_time DESC;
```

## 3. Monitorare le Tendenze delle Query (Query Insights)

La funzionalità **Query Insights (QI)** offre un archivio centralizzato di dati storici sulle query completate, fornendo analisi aggregate per l'ottimizzazione delle prestazioni. A differenza delle DMV, che mostrano solo l'attività _corrente_, QI conserva i dati per **30 giorni**.

- **Accesso:** Le informazioni sono esposte tramite viste autogenerate nello schema **`queryinsights`** all'interno dell'endpoint SQL o del Warehouse.
- **Filtro utente:** QI considera solo le query eseguite nel **contesto di un utente** (query T-SQL generate dall'utente), escludendo le query di sistema interne.

### Viste Query Insights di Riferimento

Le viste principali per l'analisi storica delle query sono:

1. **`queryinsights.exec_requests_history`**: Dettagli di ogni singola richiesta SQL **completata** (successo o fallimento), inclusi i tempi di esecuzione e il consumo di risorse.
2. **`queryinsights.exec_sessions_history`**: Informazioni sulle sessioni utente **completate**.
3. **`queryinsights.long_running_queries`**: Viste aggregate basate sulla forma della query che identificano le query con il **tempo di esecuzione mediano più lungo**.
4. **`queryinsights.frequently_run_queries`**: Viste aggregate che mostrano le query **eseguite più spesso**.

|Intuizione: Aggregazione per "Query Shape"|Spiegazione|
|---|---|
|Per fornire metriche aggregate, QI considera le query "simili" se hanno la stessa struttura, anche se i valori dei predicati (`WHERE`) sono diversi (vengono parametrizzati internamente). Questo è fondamentale per identificare i problemi di prestazione in una _classe_ di query, non solo in un'esecuzione specifica. L'identificazione della somiglianza si basa su una colonna **`query_hash`**.||
|**Esempio:** `SELECT * FROM sales WHERE orderdate > '01/01/2023'` e `SELECT * FROM sales WHERE orderdate > '12/31/2021'` sono considerate la stessa query.||

### Esempi Pratici di Query Insights

#### A. Analisi delle Query Eseguite Recentemente

Questa query recupera l'attività completata nell'ultima ora, utile per una revisione immediata post-carico di lavoro:
```SQL
SELECT 
    start_time, 
    login_name, 
    command, 
    total_elapsed_time_ms,
    distributed_statement_id
FROM queryinsights.exec_requests_history 
WHERE start_time >= DATEADD(MINUTE, -60, GETUTCDATE())
ORDER BY start_time DESC;
```

#### B. Identificazione delle Query Aggregate a Lunga Esecuzione

Questa query è vitale per l'ottimizzazione. Identifica i comandi SQL (aggregati per forma) che vengono eseguiti più di una volta e li ordina in base al loro **tempo mediano** di completamento, fornendo la priorità per l'ottimizzazione:
```SQL
SELECT 
    last_run_command, 
    number_of_runs, 
    median_total_elapsed_time_ms, 
    last_run_start_time
FROM queryinsights.long_running_queries
WHERE number_of_runs > 1
ORDER BY median_total_elapsed_time_ms DESC;
```

#### C. Identificazione delle Query Eseguite più Frequentemente

Questa query aiuta a comprendere i modelli di accesso ai dati e i carichi di lavoro più comuni, permettendo di concentrare gli sforzi di ottimizzazione sulle query ad alto impatto:
```SQL
SELECT 
    last_run_command, 
    number_of_runs, 
    number_of_successful_runs, 
    number_of_failed_runs,
    number_of_canceled_runs -- Aggiunge i dettagli sulle query annullate
FROM queryinsights.frequently_run_queries
ORDER BY number_of_runs DESC;
```

_**Nota sulla Latency:** Le query completate possono richiedere fino a 15 minuti per apparire nelle viste di Query Insights, a seconda del carico di lavoro simultaneo._

## Riepilogo Strumenti di Monitoraggio in Fabric

|Strumento|Dati|Scopo Principale|Dettaglio|
|---|---|---|---|
|**Fabric Capacity Metrics App**|Storico (14/30 giorni)|Gestione di **Costi** e **Capacità**|Individua sovraccarichi (throttling), consumi per carico di lavoro (CU) e tendenze.|
|**Dynamic Management Views (DMV)**|Tempo Reale (Live)|Monitoraggio di **Attività Corrente**|Traccia connessioni attive, sessioni e richieste in esecuzione (es. `sys.dm_exec_requests`).|
|**Query Insights (QI)**|Storico (30 giorni)|Analisi e **Ottimizzazione delle Query**|Aggrega dati sulle query completate (lunga esecuzione, frequenza) per il _tuning_.|

