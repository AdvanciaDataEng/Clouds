---
autore: Kevin Milli
---

## Obiettivi di Apprendimento Operativi

Dopo aver completato questa guida, sarai in grado di:

- **Identificare e Controllare** i principali fattori che influenzano i costi in Azure (CapEx vs OpEx).
- **Confrontare e Utilizzare** il Calcolatore Prezzi e il Calcolatore Costo Totale di Proprietà (TCO) per le stime.
- **Sfruttare** lo strumento di Gestione Costi Microsoft per il monitoraggio e la creazione di budget.
- **Implementare** i tag per l'organizzazione, il reporting dei costi e la governance delle risorse.


## Fattori Chiave che Influenzano i Costi in Azure

Azure sposta i costi dallo schema **Spesa in Conto Capitale (CapEx)** (acquisto e gestione dell'infrastruttura) allo schema **Spesa Operativa (OpEx)** (noleggio dell'infrastruttura in base al consumo). I costi OpEx sono dinamici e influenzati da:

| Fattore                         | Descrizione Operativa                                                                                                                                                                                                               |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Tipo di Risorsa**             | Il costo varia in base al servizio (VM, Archiviazione, Rete, ecc.) e al suo livello/dimensione.                                                                                                                                     |
| **Consumo/Utilizzo**            | Il principio **"paga per ciò che usi"** (pay-as-you-go) è la base. Più consumi (es. tempo di calcolo), più paghi.                                                                                                                   |
| **Geografia/Area**              | Le risorse hanno costi diversi a seconda dell'**Area di distribuzione** a causa di variazioni nei costi di energia, lavoro, tasse e imposte locali.                                                                                 |
| **Traffico di Rete**            | I trasferimenti di dati **in uscita** (dati che lasciano i data center di Azure) sono a pagamento e i prezzi si basano sulle **Zone di Fatturazione**. I trasferimenti **in ingresso** (dati che entrano) sono per lo più gratuiti. |
| **Tipo di Sottoscrizione**      | Diverse offerte di sottoscrizione (es. Developer, Enterprise) possono avere diverse strutture di prezzo.                                                                                                                            |
| **Azure Marketplace**           | I prodotti di terze parti acquistati tramite il Marketplace aggiungono costi.                                                                                                                                                       |
| **Manutenzione/Deprovisioning** | La mancata disattivazione delle risorse non più necessarie (es. l'archiviazione e la rete rimaste attive dopo il deprovisioning di una VM) genera costi residui.                                                                    |

### Strategie Operative per l'Ottimizzazione dei Costi

|Concetto|Descrizione e Beneficio|Uso Operativo|
|---|---|---|
|**Pagamento in Base al Consumo**|Modello base: paghi le risorse utilizzate nel ciclo di fatturazione. **Massima flessibilità**.|Ideale per carichi di lavoro variabili o temporanei.|
|**Capacità Riservata (Riserve)**|Impegno anticipato per l'utilizzo di una quantità prefissata di risorse (in genere 1 o 3 anni) in cambio di **sconti significativi (fino al 72%)**.|Usa per carichi di lavoro **stabili e prevedibili** (es. database, calcolo, archiviazione di base).|
|**Gestione e Pulizia**|Controlla regolarmente i **Gruppi di Risorse** per mantenere organizzazione.|**Azione:** Assicurati sempre che tutte le risorse associate (archiviazione, rete) vengano disattivate/eliminate quando la risorsa principale (es. VM) non è più necessaria.|
|**Variazione Geografica**|Il traffico di rete è meno costoso all'interno di una regione (es. Europa) rispetto a quello intercontinentale.|**Azione:** Scegli l'area più conveniente che soddisfi i requisiti di prossimità ai clienti/prestazioni.|

---

## Strumenti per la Stima e la Previsione dei Costi

### Calcolatore Prezzi Azure

Questo strumento è focalizzato sulla **stima del costo di risorse specifiche che intendi approvvigionare in Azure**.

|Funzionalità|Dettaglio Operativo|
|---|---|
|**Scopo**|Fornisce una stima dei costi mensili per le **nuove risorse/soluzioni** di cui si prevede il provisioning in Azure.|
|**Cosa Stima**|Calcolo (VM), Archiviazione, Rete e i costi associati, includendo opzioni (tipo di archiviazione, livello di accesso, ridondanza).|
|**Azione Operativa**|Prima di eseguire il calcolatore, **definire i requisiti** in dettaglio (es. numero di VM, ore di esecuzione mensili (730 ore = continuo), potenza di calcolo, volume di dati mensili).|

> **Esempio d'Uso (Configurazione di Riferimento):** Un'applicazione web interna con 2 VM Windows (ASP.NET), bilanciamento del carico (Azure Application Gateway) e database (Database SQL di Azure). I requisiti fondamentali (es. 730 ore/mese, 1 TB di rete, 32 GB di DB) devono essere inseriti nel calcolatore per ottenere una stima accurata.

### Calcolatore Costo Totale di Proprietà (TCO)

_(Nota: è lo strumento complementare al Calcolatore Prezzi.)_

|Funzionalità|Dettaglio Operativo|
|---|---|
|**Scopo**|Confronta i costi di **esecuzione della propria infrastruttura on-premise** rispetto all'**esecuzione su Azure**.|
|**Beneficio**|Dimostra il risparmio potenziale della migrazione al cloud.|


## Gestione Costi Microsoft (Microsoft Cost Management)

Questo è lo strumento operativo centrale per il **controllo, l'analisi e l'ottimizzazione** dei costi delle risorse **già in esecuzione** su Azure.

|Componente|Descrizione Operativa|
|---|---|
|**Analisi dei Costi**|Fornisce una **panoramica visiva rapida** (dashboard) dei costi totali. Permette di visualizzare e suddividere i costi in diversi modi (ciclo di fatturazione, area geografica, risorsa, **tag**).|
|**Budget**|Permette di creare budget per le sottoscrizioni, i gruppi di risorse o i tag.|
|**Avvisi di Spesa**|Consente di creare **avvisi automatizzati** basati sulla spesa effettiva o prevista, utili per controllare l'aumento improvviso della domanda o incidenti.|
|**Automazione**|Può essere utilizzato per automatizzare la gestione delle risorse in base al budget (es. spegnere servizi quando viene raggiunto un limite).|


## Tag delle Risorse Azure: Organizzazione e Reporting

I **tag** sono **metadati aggiuntivi (coppie chiave-valore)** che forniscono informazioni sulle risorse di Azure. Sono essenziali per l'organizzazione, il reporting dei costi e la governance operativa.

### Utilizzi Operativi dei Tag

| Categoria Operativa              | Scopo e Beneficio                                                                                                                                  | Esempio di Tag (Chiave:valore)                     |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| **Gestione dei Costi**           | Raggruppare le risorse per creare report di costo, allocare centri di costo interni, tracciare budget. **Essenziale per la fatturazione interna.** | Dipartimento: IT \|<br>CentroCosto: 1001           |
| **Gestione Risorse**             | Individuare rapidamente le risorse associate a specifici carichi di lavoro, ambienti o proprietari.                                                | Ambiente:Sviluppo \|<br>Applicazione:InventarioWeb |
| **Governance/Conformità**        | Identificare le risorse che rientrano in specifici requisiti normativi (es. ISO 27001).                                                            | Conformita:ISO27001 \|<br>Proprietario:MarioRossi  |
| **Gestione Operazioni**          | Raggruppare le risorse in base alla criticità per formulare **Accordi sul Livello di Servizio (SLA)**.                                             | Criticità:Alta \|<br>SLA:24x7                      |
| **Automazione Carico di Lavoro** | Visualizzare tutte le risorse in distribuzioni complesse; usare software come Azure DevOps per automatizzare attività.                             | Versione:v2.1<br>Fase:Testing                      |

### Gestione dei Tag

- **Implementazione:** I tag possono essere aggiunti, modificati o eliminati tramite **PowerShell, CLI di Azure, Modelli ARM, API REST o Portale di Azure**.
- **Non Ereditarietà:** Le risorse **non ereditano** automaticamente i tag da sottoscrizioni o gruppi di risorse, consentendo schemi di tag personalizzati a ogni livello.
- **Enforcement (Applicazione):** Utilizza **Criteri di Azure (Azure Policy)** per imporre regole e convenzioni di assegnazione di tag. Puoi:
    - Richiedere l'aggiunta di determinati tag alle nuove risorse.
    - Definire regole che riapplicano i tag rimossi.

## Descrivere lo strumento Gestione costi Microsoft

Gestione costi consente di controllare rapidamente i costi delle risorse di Azure, creare avvisi in base alla spesa delle risorse e creare budget che possono essere usati per automatizzare la gestione delle risorse.

L'analisi dei costi è un sottogruppo della gestione dei costi che fornisce una rapida panoramica visiva dei costi di Azure. Usando l'analisi dei costi, è possibile visualizzare rapidamente il costo totale in diversi modi, tra cui ciclo di fatturazione, area, risorsa e così via.



