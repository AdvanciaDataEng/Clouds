---
autore: Kevin Milli
---
 
## Introduzione all'Account di Archiviazione di Azure

Un **Account di Archiviazione di Azure** è il contenitore fondamentale per tutti i tuoi oggetti di dati di Archiviazione di Azure: **BLOB**, **File**, **Code** e **Tabelle**. Fornisce un **namespace** univoco e accessibile a livello globale (tramite HTTP o HTTPS) per i tuoi dati, garantendo **durabilità**, **disponibilità elevata**, **sicurezza** e **scalabilità massiva**.

### Tipi di Account di Archiviazione (Kind)

Il tipo di account (o Kind) che scegli al momento della creazione determina i servizi di archiviazione disponibili, le opzioni di ridondanza e il livello di prestazioni (Standard o Premium).

|**Tipo di Account**|**Kind (Nome Tecnico)**|**Livello**|**Servizi Supportati**|**Ridondanza Supportata**|**Utilizzo/Casi d'Uso**|
|---|---|---|---|---|---|
|**Utilizzo Generale v2 (GPv2)**|`StorageV2`|**Standard**|BLOB (Block/Append/Page), Data Lake Gen2, File, Code, Tabelle|LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS|**Raccomandato** per la maggior parte degli scenari, offre la massima flessibilità e supporto per i livelli di accesso BLOB (Hot, Cool, Cold, Archive).|
|**BLOB in Blocchi Premium**|`BlockBlobStorage`|**Premium**|Solo BLOB in Blocchi e di Accodamento|LRS, ZRS|Scenari con **frequenze di transazione elevate**, oggetti più piccoli o che richiedono una **latenza di archiviazione costantemente bassa** (es. Big Data Analytics, cache, logging).|
|**Condivisioni File Premium**|`FileStorage`|**Premium**|Solo File di Azure (SMB/NFS)|LRS, ZRS|Applicazioni aziendali o ad **alte prestazioni** che necessitano di condivisioni file.|
|**BLOB di Pagine Premium**|`StorageV2` (con Page Blobs abilitati)|**Premium**|Solo BLOB di Pagine|LRS|Ottimizzato per gli **Azure Managed Disks** (Dischi Gestiti di Azure) per le VM.|

## Endpoint dell'Account di Archiviazione

L'account di archiviazione fornisce un **namespace univoco** in Azure. 
L'indirizzo URL per ogni oggetto memorizzato è formato dalla combinazione del nome univoco dell'account e dell'**endpoint di servizio** specifico.

### Struttura dell'Endpoint Standard

Un endpoint di servizio standard include il protocollo (`HTTPS` raccomandato), il nome dell'account di archiviazione come sottodominio e un dominio fisso che include il nome del servizio.

|**Servizio di Archiviazione**|**Formato Endpoint Standard**|
|---|---|
|Archiviazione BLOB|`https://<storage-account-name>.blob.core.windows.net`|
|Data Lake Storage Gen2|`https://<storage-account-name>.dfs.core.windows.net`|
|File di Azure|`https://<storage-account-name>.file.core.windows.net`|
|Archiviazione Code|`https://<storage-account-name>.queue.core.windows.net`|
|Archiviazione Tabelle|`https://<storage-account-name>.table.core.windows.net`|

**Regole di Naming:**

- Tra 3 e 24 caratteri.
- Può contenere solo numeri e lettere minuscole.
- Deve essere **univoco** a livello globale in Azure.

### Intuizione: Endpoint Pubblici vs. Privati

Oltre agli endpoint standard (**pubblici**), Azure supporta:

- **Endpoint Privati (Private Endpoints)**: Forniscono un indirizzo IP privato dallo spazio degli indirizzi della tua **Azure Virtual Network (VNet)**. Questo permette alle VM nella tua VNet di accedere all'account di archiviazione in modo sicuro tramite un **Azure Private Link**, mantenendo il traffico all'interno della rete Microsoft, bypassando l'Internet pubblico.

## Livelli di Accesso per i BLOB (BLOB Access Tiers)

I livelli di accesso di Azure Blob Storage (disponibili principalmente per gli account **GPv2**) consentono di ottimizzare i costi in base alla frequenza di accesso ai dati. Il compromesso è tra un costo di archiviazione mensile più basso e un costo di accesso/recupero (transazione) più alto e una maggiore latenza.

|**Livello**|**Frequenza di Accesso**|**Costo di Archiviazione**|**Costo di Accesso/Transazione**|**Latenza di Recupero**|**Permanenza Minima**|**Casi d'Uso**|
|---|---|---|---|---|---|---|
|**Hot (Online)**|**Frequente**|Più alto|Più basso|Millisecondi (Immediato)|Nessuna|Dati in uso attivo, dati per analisi in tempo reale.|
|**Cool (Online)**|**Infrequente**|Più basso|Più alto di Hot|Millisecondi (Immediato)|30 giorni|Backup a breve termine, dati che devono essere immediatamente disponibili ma a cui si accede raramente.|
|**Cold (Online)**|**Raro**|Basso|Alto|Millisecondi (Immediato)|90 giorni|Backup a lungo termine, dati di conformità. Offre una latenza simile a Cool, ma a un costo di archiviazione inferiore e un costo di accesso più elevato.|
|**Archive (Offline)**|**Quasi mai**|Più basso in assoluto|Più alto in assoluto|Ore (**Reidratazione** necessaria)|180 giorni|Archiviazione storica, dati di conformità a lungo termine che possono tollerare ritardi nel recupero.|

Intuizione: Lifecycle Management

Puoi automatizzare la transizione dei BLOB tra i livelli (ad esempio, da Hot a Cool dopo 30 giorni) utilizzando le policy di gestione del ciclo di vita (Lifecycle Management policies) per ottimizzare automaticamente i costi.

## Ridondanza di Archiviazione di Azure

La ridondanza è la base della durabilità di Archiviazione di Azure, garantendo che i dati siano protetti da guasti di rack, data center, zone o intere regioni. I dati vengono sempre replicati tre volte nell'area primaria.

### Tipi di Ridondanza

La scelta del tipo di ridondanza è un bilanciamento tra **costo** (LRS è il meno costoso) e **livello di durabilità/disponibilità** (RA-GZRS è il massimo).

| **Opzione**                                              | **Replica nell'Area Primaria**                                | **Geo-Replicazione (Area Secondaria)**                          | **Accesso in Lettura Secondaria** | **Livello di Protezione**                                                                                    |
| -------------------------------------------------------- | ------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **LRS (Localmente Ridondante)**                          | 3 copie sincrone in **un singolo Data Center**.               | **No**                                                          | No                                | Contro guasti di rack e unità. Durabilità).                                                                  |
| **ZRS (Zona-Ridondante)**                                | 3 copie sincrone in **3 Zone di Disponibilità** (AZ) diverse. | **No**                                                          | No                                | Contro guasti di Data Center (Zona). Durabilità.                                                             |
| **GRS (Geo-Ridondante)**                                 | LRS (3 copie in 1 Data Center).                               | 3 copie LRS asincrone in una Regione Secondaria **accoppiata**. | **No**                            | Contro guasti a livello di Regione. Durabilità.                                                              |
| **RA-GRS (Geo-Ridondante con Accesso in Lettura)**       | LRS.                                                          | 3 copie LRS asincrone in una Regione Secondaria.                | **Sì**                            | Come GRS, ma consente di **leggere** i dati dalla regione secondaria anche senza failover.                   |
| **GZRS (Geo-Zona-Ridondante)**                           | ZRS (3 copie in 3 AZ).                                        | 3 copie LRS asincrone in una Regione Secondaria **accoppiata**. | **No**                            | Contro guasti di Data Center (Zona) e a livello di Regione. **Massima disponibilità/durabilità.** Durabilità |
| **RA-GZRS (Geo-Zona-Ridondante con Accesso in Lettura)** | ZRS.                                                          | 3 copie LRS asincrone in una Regione Secondaria.                | **Sì**                            | Come GZRS, ma consente di **leggere** i dati dalla regione secondaria anche senza failover.                  |

### Dettagli sulla Ridondanza

#### Ridondanza nell'Area Primaria (Alta Disponibilità)

- **LRS:** Fornisce la durabilità più bassa e il costo più basso. Non protegge da un disastro all'interno del data center.
- **ZRS:** Richiede regioni che supportino le **Zone di Disponibilità** (separate fisicamente, con alimentazione, raffreddamento e rete indipendenti). Garantisce che i dati siano accessibili anche se un'intera zona fallisce.

#### Ridondanza in un'Area Secondaria (Disaster Recovery)

- **Geo-Replicazione (GRS/GZRS):** Protegge dai disastri a livello regionale, replicando i dati in un'area secondaria a centinaia di chilometri di distanza.
    - La replica verso l'area secondaria è **asincrona** (non istantanea).
    - **Obiettivo del Punto di Ripristino (RPO):** L'intervallo tra le scritture più recenti nell'area primaria e l'ultima scrittura nell'area secondaria. Azure mira a un RPO inferiore a 15 minuti.
    - Per impostazione predefinita, i dati nell'area secondaria sono in sola lettura e non disponibili se non viene eseguito un **Failover** esplicito.

#### Accesso in Lettura all'Area Secondaria (RA-GRS / RA-GZRS)

- I tipi **RA-** (Read-Access) offrono un endpoint aggiuntivo per l'area secondaria. Questo consente alle applicazioni di **leggere** i dati replicati nella regione secondaria anche quando la regione primaria è pienamente operativa.
- **Intuizione Pratica:** L'uso di RA-GRS/RA-GZRS è essenziale per le applicazioni che devono mantenere la disponibilità delle operazioni di lettura durante un'interruzione della regione primaria, sebbene i dati letti possano non essere aggiornati a causa dell'RPO.

---

# Servizi Core di Archiviazione di Azure

Questo modulo descrive in dettaglio i principali servizi di archiviazione dati offerti dalla piattaforma Azure, le loro caratteristiche uniche e i casi d'uso ideali.

## Panoramica e Vantaggi di Archiviazione di Azure

La piattaforma **Archiviazione di Azure** è una soluzione di archiviazione gestita, sicura e scalabile per l'era del cloud.

### I Servizi Dati Core

La piattaforma include cinque servizi dati principali, tutti ospitati all'interno di un unico **Account di Archiviazione**:

1. **Azure BLOB Storage (Object Store):** Archiviazione non strutturata per dati di testo o binari.
2. **Azure Files (File Shares):** Condivisioni file gestite (SMB/NFS) accessibili da distribuzioni cloud e locali.
3. **Azure Queues (Messaging):** Un archivio di messaggistica per comunicazioni asincrone tra componenti di applicazioni.
4. **Azure Disks (Block Storage):** Volumi di archiviazione a livello di blocco per le Macchine Virtuali (VM) di Azure.
5. **Azure Tables (NoSQL):** Datastore NoSQL per grandi quantità di dati strutturati e non relazionali.

### Vantaggi Fondamentali

|Vantaggio|Descrizione Sintetica e Tecnica|
|---|---|
|**Durabilità & Disponibilità**|Garantita da **ridondanza** integrata (LRS, ZRS, GRS, GZRS), proteggendo da guasti hardware e disastri regionali.|
|**Sicurezza**|Tutti i dati sono crittografati **at rest** (crittografia lato servizio) e **in transito** (HTTPS). Controllo granulare degli accessi (RBAC, SAS, ACL).|
|**Scalabilità Massiva**|Progettato per essere **massively scalable**, supportando petabyte di dati e gestendo migliaia di richieste concorrenti.|
|**Gestione**|Soluzione **gestita**; Azure si occupa della manutenzione hardware, degli aggiornamenti e della risoluzione dei problemi critici.|
|**Accessibilità**|Accesso globale tramite **HTTP/HTTPS** via API REST e librerie client avanzate in linguaggi multipli (.NET, Java, Python, ecc.).|

## Azure BLOB Storage (Object Storage)

**BLOB Storage** è la soluzione di archiviazione oggetti non strutturata di Azure, ideale per dati come immagini, video, file di log e backup. Offre scalabilità e accessibilità illimitate.

### Tipologie di BLOB (Intuizione Avanzata)

I dati BLOB non sono tutti uguali; la scelta del tipo dipende dal pattern di I/O (Input/Output):

|Tipo di BLOB|Utilizzo Principale|Caratteristiche|
|---|---|---|
|**Block BLOBs**|Archiviazione di oggetti discreti (immagini, documenti, file multimediali, backup). **Il tipo più comune.**|Ottimizzato per operazioni di **upload e download** massive. Composto da blocchi, gestisce file fino a ~4.75 TB (inclusi i file usati in Data Lake Storage Gen2).|
|**Page BLOBs**|Utilizzato per i **Dischi Gestiti di Azure (Managed Disks)**.|Ottimizzato per operazioni di **lettura/scrittura casuale (random read/write)** in-place, come i dischi rigidi virtuali (VHD).|
|**Append BLOBs**|Archiviazione di dati di **logging** e **telemetria**.|Ottimizzato per operazioni di **append only** (aggiunta alla fine). Utile quando i dati vengono aggiunti continuamente senza modifiche alle parti esistenti.|

### Livelli di Accesso BLOB (BLOB Tiers)

I livelli di accesso permettono di bilanciare costi di archiviazione e accesso in base alla frequenza di utilizzo dei dati (disponibili per **Block BLOBs** negli account GPv2).

|Livello|Frequenza di Accesso|Costo di Archiviazione|Costo di Accesso|Permanenza Minima|
|---|---|---|---|---|
|**Hot (Frequente)**|Frequente (es. sito web)|Più Alto|Più Basso|Nessuna|
|**Cool (Sporadico)**|Infrequente (es. fatture, dati operativi)|Più Basso|Più Alto|30 giorni|
|**Cold (Saltuario)**|Raro, ma accesso veloce necessario|Basso|Alto|90 giorni|
|**Archive (Archivio)**|Quasi mai (es. backup a lungo termine)|Più Basso in assoluto|Più Alto in assoluto|180 giorni|

>[!Nota] Intuizione Tecnica: Livello Archive
> 
> I dati in Archive sono offline e non accessibili immediatamente. 
> Richiedono un processo di Reidratazione (Rehydration) (di solito ore) per essere spostati in un livello Hot o Cool prima di poter essere letti.

## Azure Files (File Shares)

**File di Azure** offre condivisioni file completamente gestite accessibili tramite i protocolli **Server Message Block (SMB)** o **Network File System (NFS)**. Sostituisce i server file locali.

### Protocolli e Casi d'Uso

- **SMB (Server Message Block):** Utilizzato principalmente da client Windows. Supporta integrazione con Azure AD/on-prem AD per l'autenticazione.
- **NFS (Network File System):** Utilizzato da client Linux e macOS.

|Caratteristica|Vantaggio Chiave|Caso d'Uso|
|---|---|---|
|**Accesso Condiviso**|Consente il montaggio simultaneo.|Lift-and-shift di applicazioni legacy che si basano su percorsi di file condivisi.|
|**Sincronizzazione File**|Sincronizzazione con server Windows locali (tramite **Sincronizzazione file di Azure**).|Caching dei dati più caldi a livello locale per un accesso veloce (Edge Computing).|

## Azure Disks (Block Storage)

**Dischi di Azure** fornisce l'archiviazione a livello di blocco virtualizzata necessaria per le Macchine Virtuali (VM). 
I dischi sono persistenti e gestiti da Azure (**Managed Disks**), eliminando la necessità di gestire gli account di archiviazione sottostanti e garantendo elevata resilienza.

- **Managed Disks:** Archiviazione ottimizzata per le VM. Azure gestisce la replica e l'alta disponibilità per te.
- **Tipi di Dischi:** Offre diverse opzioni di prestazioni: Ultra Disk, Premium SSD, Standard SSD e Standard HDD, per soddisfare ogni esigenza di latenza e IOPS.

## Azure Queues (Messaging) e Azure Tables (NoSQL)

Questi servizi soddisfano esigenze specifiche di archiviazione per applicazioni distribuite e dati strutturati non relazionali.

### Azure Queues

- **Funzione:** Archiviazione di un elevato numero di **messaggi (fino a 64 KB)** per la messaggistica asincrona tra i componenti di un'applicazione.
- **Scopo:** Creare un **backlog di lavoro** da elaborare in un secondo momento (decoupling tra produttori e consumatori).
- **Esempio:** Un'app web invia un messaggio (es. "elabora immagine") alla coda, e un'**Azure Function** preleva il messaggio per eseguire l'elaborazione in background.

### Azure Tables

- **Funzione:** Un datastore **NoSQL** per l'archiviazione di grandi volumi di dati **strutturati e non relazionali**.
- **Scalabilità:** Offre un'archiviazione massiva a costi molto bassi.
- **Differenza chiave:** Non offre funzionalità di join, stored procedure o relazioni complesse come un database relazionale (SQL). Ideale per metadati, rubriche utente o altri dati che possono essere modellati come **entità** e **proprietà**.

# Opzioni di Migrazione e Movimentazione dei Dati

Comprendere come portare i dati e le infrastrutture in Azure (e viceversa) è cruciale. 
Azure offre soluzioni che vanno dalla migrazione completa dell'infrastruttura (**Azure Migrate**) al trasferimento offline di Big Data (**Azure Data Box**) e alla gestione quotidiana dei file (**AzCopy, Storage Explorer, Sincronizzazione file**).

## Azure Migrate: L'Hub per la Migrazione Cloud

**Azure Migrate** è un servizio centralizzato (hub) che fornisce strumenti unificati per l'**assessment (valutazione)**, la **migrazione** e la **modernizzazione** dei carichi di lavoro on-premises (VMware, Hyper-V, server fisici) verso Azure. Funge da piattaforma di gestione per l'intero percorso di migrazione.

### Strumenti Integrati e Funzionalità

|Categoria|Strumento Integrato|Scopo e Funzione|
|---|---|---|
|**Assessment (Valutazione)**|**Individuazione e Valutazione**|Scopre i server e le dipendenze in esecuzione on-premises. Valuta l'idoneità ad Azure, l'ottimizzazione delle dimensioni (sizing) e stima i costi.|
|**Migrazione Server**|**Migrazione del Server**|Esegue la migrazione effettiva di macchine virtuali e server fisici ad Azure (migrazione _lift-and-shift_). Supporta la replica in tempo reale.|
|**Database**|**Data Migration Assistant (DMA)**|Strumento **autonomo** per valutare le istanze di SQL Server per la migrazione. Individua problemi di compatibilità e funzionalità non supportate.|
|**Database (Esecuzione)**|**Servizio Migrazione del database (DMS)**|Esegue la migrazione di database on-premises (SQL Server, Oracle, MySQL, ecc.) a servizi Azure come Azure SQL Database, Istanza Gestita di SQL o SQL su VM.|
|**Web Apps**|**Assistente di Migrazione per Servizio app**|Valuta e migra siti Web locali (.NET, PHP) direttamente ad **Azure App Service**.|

> **Intuizione Migrazione:** Azure Migrate non è un singolo strumento, ma un **framework orchestrativo** che coordina più servizi per garantire una migrazione coesa e tracciabile.

## Azure Data Box: Migrazione Offline di Big Data

**Azure Data Box** è un servizio di trasferimento dati **fisico** (offline), ideale per spostare quantità di dati superiori a **40 TB** in scenari con connettività di rete scarsa o costosa. Consiste in un dispositivo di archiviazione sicuro e resistente spedito da e verso il data center Microsoft.

|Prodotto Data Box|Capacità (Utilizzabile)|Tipo di Trasferimento|Utilizzo Ideale|
|---|---|---|---|
|**Data Box (Dispositivo)**|80 TB|Importazione/Esportazione fisica|Trasferimento una tantum di Big Data, Farm VM, librerie multimediali.|
|**Data Box Disk**|35 TB (per set di 5 dischi)|Importazione fisica|Set di dischi crittografati per carichi di dati più piccoli e distribuiti.|
|**Data Box Heavy**|770 TB|Importazione/Esportazione fisica|Migrazioni di data center o trasferimento di archivi su larga scala.|

### Casi d'Uso Chiave

- **Migrazione Una Tantum:** Spostamento iniziale di grandi archivi o database.
- **Trasferimento in Blocco Iniziale (Seeding):** Uso di Data Box per caricare la base di dati, seguito da repliche incrementali sulla rete (es. per repliche di ripristino di emergenza).
- **Ripristino di Emergenza (Esportazione):** Esportazione rapida di grandi volumi di dati da Azure per il ripristino on-premises.
- **Requisiti di Conformità/Legali:** Esportazione di dati da Azure a causa di vincoli normativi.

> **Sicurezza:** Dopo il caricamento in Azure, il contenuto dei dischi viene **cancellato in modo sicuro** secondo gli standard **NIST 800-88r1**.

## Opzioni di Spostamento e Sincronizzazione dei File

Per la movimentazione quotidiana o la sincronizzazione continua di file, Azure offre strumenti flessibili e potenti che operano a livello di file/BLOB.

### AzCopy: Il Coltello Svizzero della Riga di Comando

**AzCopy** è un'utilità da riga di comando ad alte prestazioni, ottimizzata per copiare dati in modo efficiente tra account di archiviazione, in o da un filesystem locale, o persino tra cloud diversi.

- **Funzionalità:** Caricamento, download, copia tra account e sincronizzazione.
- **Sincronizzazione Unidirezionale:** Quando si usa il comando `sync`, AzCopy esegue una **sincronizzazione unidirezionale** (source ![](data:,) destination), basandosi su timestamp o hash MD5. **Non** è una sincronizzazione bidirezionale (a differenza di Sincronizzazione file di Azure).

### Azure Storage Explorer (GUI)

**Azure Storage Explorer** è un'applicazione desktop standalone (Windows, macOS, Linux) che fornisce un'interfaccia utente grafica per la gestione di tutti gli oggetti di Archiviazione di Azure (BLOB, File, Code, Tabelle).

- **Tecnologia:** Utilizza **AzCopy** nel back-end per l'efficienza dei trasferimenti, ma offre la facilità di un'interfaccia grafica.
- **Uso:** Semplifica le attività di visualizzazione, caricamento, download e spostamento di dati tra account.

### Sincronizzazione file di Azure (Azure File Sync)

**Sincronizzazione file di Azure** è uno strumento di **storage ibrido** che centralizza le condivisioni file in **File di Azure** mantenendo l'accesso, le prestazioni e la compatibilità di un server file Windows locale.

- **Sincronizzazione Bidirezionale:** Dopo l'installazione di un agente sul server Windows, esegue una **sincronizzazione bidirezionale** tra la condivisione file locale e la condivisione file di Azure (il **Cloud Endpoint**).
- **Tiering nel Cloud (Cloud Tiering):** Funzionalità chiave che trasforma il file server locale in una **cache dinamica**.
    - I file a cui si accede più frequentemente (hot) rimangono in locale.
    - I file a cui si accede raramente (cool) vengono **divisi in livelli** in Azure, lasciando solo un puntatore (reparse point) sul server locale per risparmiare spazio. Il file viene scaricato su richiesta (on-demand).
- **Vantaggio:** Permette di ridurre la capacità di archiviazione on-premises, mantenendo al contempo un accesso veloce e tutti i protocolli (SMB, NFS, FTPS) di Windows Server.


