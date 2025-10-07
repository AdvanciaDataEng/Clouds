---
autore: Kevin Milli
---

Il monitoraggio e l'automazione sono fasi cruciali in una **soluzione di analisi dei dati** per garantire che il framework di raccolta, archiviazione, elaborazione e analisi dei dati sia **integro, efficiente e resiliente**. I dati grezzi, infatti, diventano significativi per le decisioni aziendali solo dopo essere stati ingeriti, combinati, trasformati e caricati in archivi dati.

---

## 1. Fondamenti di Monitoraggio in Fabric

Il **monitoraggio** è il processo di raccolta continua di **dati di sistema e metriche** per determinare se un sistema o un processo (come l'ingestione e la trasformazione dei dati) stia operando come previsto.

### 1.1. Scopo del Monitoraggio

- **Esposizione degli errori:** Identificare quando e dove si sono verificati fallimenti.
- **Analisi dei problemi:** Utilizzare dati storici (log e metriche) per analizzare le cause profonde e correggere gli errori.
- **Affidabilità e Resilienza:** Gestire i processi end-to-end per garantire che i dati arrivino tempestivamente e che errori o ritardi non influenzino le attività _downstream_ o gli utenti.
- **Ottimizzazione delle Prestazioni:** Identificare colli di bottiglia o problemi di lentezza per migliorare l'efficienza.

### 1.2. Attività di Fabric da Monitorare

In Microsoft Fabric, le attività che eseguono spostamento e trasformazione dei dati (come le attività ETL/ELT) devono essere monitorate per mantenere l'integrità del sistema.

|Attività di Fabric|Scopo Principale|Metriche Critiche da Monitorare|
|---|---|---|
|**Pipeline di Dati (Data Pipeline)**|Orchestrazione e gestione di un insieme di attività (es. ETL) che eseguono processi di ingestione.|Successo/Fallimento, durata dell'esecuzione (storico), errori specifici.|
|**Flussi di Dati (Dataflows)**|Strumento _low-code_ per ingestione, caricamento e trasformazione (eseguibili manualmente, pianificati o tramite pipeline).|Ora di inizio/fine, stato, durata, attività di caricamento delle tabelle.|
|**Aggiornamenti del Modello Semantico**|Aggiornamento del modello di dati (trasformazioni, calcoli, relazioni) per la creazione di report e visualizzazioni.|Tentativi di aggiornamento (per identificare problemi temporanei), esito positivo/negativo.|
|**Processi Spark, Notebook e Lakehouse**|Interfaccia per lo sviluppo di processi Apache Spark per caricare o trasformare dati per i Lakehouse.|Stato del processo Spark, esecuzione delle attività, utilizzo delle risorse, log di Spark.|
|**Flussi di Eventi (Eventstreams)**|Inserimento e trasformazione di eventi in tempo reale/streaming (dati perpetui e temporizzati).|Dati degli eventi di streaming, stato di ingestione, prestazioni di ingestione.|

### 1.3. Procedure Consigliate per il Monitoraggio

1. **Definizione:** Identificare chiaramente **cosa monitorare** (metriche specifiche).
2. **Rilevazione di Anomalie:** Raccogliere e analizzare i dati a intervalli regolari per stabilire il **comportamento normale (baseline)**, in modo da individuare rapidamente le anomalie.
3. **Azione:** Intervenire tempestivamente per risolvere i problemi quando metriche e log mostrano **deviazioni significative** dal comportamento normale.
4. **Revisione Continua:** Esaminare regolarmente log e metriche per affinare la baseline e ottimizzare le prestazioni (identificazione di colli di bottiglia).

---

## 2. Utilizzo dell'Hub di Monitoraggio di Microsoft Fabric (Monitoring Hub)

L'**Hub di Monitoraggio** è lo strumento di **visualizzazione e aggregazione centralizzata** per il monitoraggio in Microsoft Fabric. 
Esso raccoglie e aggrega i dati delle attività e dei processi di Fabric in un'unica interfaccia.

### 2.1. Funzionalità e Vantaggi

- **Centralizzazione:** Visualizza lo stato di diverse attività di integrazione, trasformazione, spostamento e analisi dati (come esecuzioni di Pipeline, Dataflows, aggiornamenti di modelli semantici, processi Spark/Notebook) in un unico luogo.
- **Visualizzazione:** Semplifica l'identificazione di tendenze o anomalie.
- **Accesso:** Si apre selezionando **Monitor** nel riquadro di navigazione di Fabric.

### 2.2. Dettagli delle Attività

Selezionando un'attività nell'Hub (tramite i puntini di sospensione **(...)**), è possibile visualizzare i **metadati** specifici:

- **Stato dell'impegno** (Successo, Fallimento, In corso, ecc.)
- **Ora di inizio e fine**
- **Durata**
- **Statistiche sulle attività**
- **Drill-down:** Alcune attività permettono di eseguire il _drill-down_ nei dettagli di esecuzione per ottenere informazioni precise su errori, successo e log di sistema (es. dettagli sulle applicazioni Spark).

**Intuizione:** L'Hub di Monitoraggio è primariamente uno **strumento di _visibilità_** per lo stato delle attività, fornendo dati storici e in corso. Per un monitoraggio più avanzato e l'automazione in tempo reale basata sui dati in streaming, si usa Activator.

---

## 3. Intervenire con Microsoft Fabric Activator

**Microsoft Fabric Activator** è una tecnologia di **Real-Time Intelligence** che funge da "sistema nervoso digitale" di Fabric. 
Il suo scopo è l'**elaborazione automatizzata di eventi** che attivano azioni (_re-attive_) quando vengono rilevate condizioni, modelli o anomalie nei dati in modifica (spesso _streaming_).

### 3.1. Ruolo e Funzionamento di Activator

Activator è il **motore di rilevamento eventi e regole** del _Real-Time Intelligence_ di Fabric. Connesso direttamente a flussi di eventi (_eventstreams_) o a visualizzazioni Power BI/Real-Time Dashboards, valuta continuamente le condizioni definite dall'utente.

- **Fonti Dati:** Può osservare dati in tempo reale da _Eventstreams_ (sensori IoT, dati ad alta frequenza) o dati da report Power BI.
- **Automazione No-Code:** Offre un'esperienza _no-code_ per definire trigger visivi.
- **Azioni:** Quando una condizione è soddisfatta, Activator può intraprendere azioni come:
    - Invio di notifiche (E-mail, Teams).
    - Esecuzione di elementi di processo Fabric (Pipeline, Notebook, Processi Spark).
    - Avvio di flussi di lavoro Power Automate (per l'integrazione con sistemi esterni).

### 3.2. Concetti Chiave di Activator (Reflex)

Activator opera sulla base di quattro concetti fondamentali che definiscono una sua istanza (chiamata anche **Reflex**):

|Concetto|Descrizione|Esempio|
|---|---|---|
|**Eventi**|Ogni **record** in un flusso di dati che rappresenta un'osservazione sullo stato di un oggetto in un momento specifico.|Una lettura di temperatura da un sensore.|
|**Oggetti**|Le **entità di business** (fisiche o concettuali) che vengono monitorate. I dati dell'evento vengono mappati agli oggetti.|Un sensore specifico, un ordine di vendita, un cliente.|
|**Proprietà**|I **campi/valori** nei dati dell'evento che rappresentano un aspetto dello stato dell'oggetto.|Il campo `temperatura` o `total_amount`.|
|**Regole**|Le **condizioni** logiche che, se soddisfatte dai valori delle proprietà degli oggetti, **attivano un'azione**. Le regole possono essere basate su soglie, modelli o logica KQL.|"Se la proprietà `temperatura` supera i ![](data:,)".|

### 3.3. Casi d'Uso di Activator

Activator è essenziale per scenari che richiedono **analisi e azioni in tempo reale**:

- **Manutenzione Predittiva:** Notificare un tecnico quando la temperatura di un macchinario supera una soglia critica.
- **Logistica:** Allertare se una spedizione non è stata aggiornata entro un intervallo di tempo previsto.
- **Marketing/Vendite:** Avviare un'azione di marketing o pubblicitaria quando le vendite di un prodotto in un dato negozio scendono.
- **Controllo Qualità:** Fermare automaticamente un processo produttivo se un sensore rileva un difetto (anomalia).
- **Monitoraggio Tecnico:** Avvisare in tempo reale in caso di fallimenti nelle pipeline di dati critiche (limitazione attuale: l'attivazione è a livello di singolo elemento, non globale, spingendo verso un approccio ibrido con Log Analytics per il monitoraggio _bulk_).

Questo video di Microsoft Mechanics illustra l'utilizzo di Data Activator in Microsoft Fabric per automatizzare le azioni sui dati in tempo reale: Automate data-driven actions | Data Activator in Microsoft Fabric.

