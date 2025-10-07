
La sicurezza in **Microsoft Fabric** è fondamentale per proteggere i dati e garantire che utenti diversi possano eseguire azioni specifiche in linea con le loro responsabilità lavorative. Fabric offre un modello di sicurezza multilivello e flessibile basato sui concetti di **privilegio minimo** e adattabilità a diversi casi d'uso.

---

## Livelli e Componenti della Sicurezza di Fabric

Il modello di sicurezza di Fabric è multilivello e valuta l'accesso in sequenza:

1. **Autenticazione di Microsoft Entra ID (precedentemente Azure AD):** Verifica se l'utente può autenticarsi al servizio di gestione delle identità e degli accessi.
2. **Accesso Fabric:** Controlla se l'utente può accedere alla piattaforma Fabric.
3. **Sicurezza dei Dati:** Determina se l'utente può eseguire l'azione richiesta su tabelle, file o altri elementi di dati.

Il terzo livello, la **Sicurezza dei Dati**, è composto da diversi elementi costitutivi che possono essere combinati per definire requisiti di accesso granulari:

|Elemento Costitutivo|Ambito di Applicazione|Livello di Granularità|
|---|---|---|
|**Ruoli dell'Area di Lavoro**|Intera Area di Lavoro|Grossolano (Accesso a tutti gli elementi)|
|**Autorizzazioni degli Elementi**|Singolo Elemento (e.g., Lakehouse, Report)|Fino al livello di elemento|
|**Autorizzazioni di Calcolo/Granulari**|Motore di Calcolo (e.g., Endpoint SQL, Modello Semantico)|Granulare (Tabella, Riga, Colonna)|
|**Ruoli di Accesso ai Dati OneLake (Anteprima)**|Cartelle e File in OneLake|Granulare (Livello di file/cartella)|

**Intuizione:** Pensa a questo come a un set di chiavi 🔑: i ruoli dell'area di lavoro ti danno la chiave per l'edificio (l'area di lavoro); le autorizzazioni degli elementi ti danno la chiave per un ufficio specifico (l'elemento); le autorizzazioni granulari e i ruoli OneLake ti danno la chiave per un cassetto o un armadietto all'interno dell'ufficio (dati specifici).

---

## 1. Sicurezza a Livello di Area di Lavoro (Ruoli) e di Elemento (Permessi)

### Ruoli dell'Area di Lavoro

Le aree di lavoro (Workspace) sono ambienti collaborativi. I ruoli dell'area di lavoro definiscono le operazioni che gli utenti possono eseguire su _tutti gli elementi_ all'interno di quell'area di lavoro.

|Ruolo|Operazioni Principali|Livello di Accesso ai Dati|
|---|---|---|
|**Amministratore**|Visualizzare, modificare, condividere e gestire tutto il contenuto e le autorizzazioni.|Massime (Controllo totale)|
|**Membro**|Visualizzare, modificare e condividere tutto il contenuto.|Alto (Modifica e lettura)|
|**Collaboratore**|Visualizzare e modificare tutto il contenuto.|Medio (Modifica e lettura)|
|**Visualizzatore**|Visualizzare tutto il contenuto.|Solo lettura|

- **Assegnazione:** I ruoli possono essere assegnati a singoli utenti, gruppi di sicurezza, gruppi di Microsoft 365 e liste di distribuzione tramite il pulsante **Gestisci accesso** dell'area di lavoro.
- **Esempio:** Un **Ingegnere dei Dati** che deve creare nuovi elementi e leggere i dati esistenti riceverebbe il ruolo di **Collaboratore**.

### Autorizzazioni degli Elementi

Le autorizzazioni degli elementi controllano l'accesso a **singoli elementi** (Lakehouse, Report, ecc.) e possono essere usate per:

1. **Modificare** le autorizzazioni ereditate dal ruolo dell'area di lavoro (restringendo l'accesso).
2. **Concedere accesso** a un utente a un elemento specifico senza aggiungerlo a un ruolo dell'area di lavoro (ad es., un utente che è solo _Visualizzatore_ nell'area di lavoro può avere un'autorizzazione specifica per _leggere i dati_ in un solo Lakehouse).

- **Configurazione:** Si configurano tramite **Gestisci autorizzazioni** per il singolo elemento.
- **Lettura Dati:** L'accesso in lettura ai _metadati_ di un Lakehouse (visualizzare il nome dell'elemento e la struttura) non implica automaticamente l'accesso ai _dati_ sottostanti. Per la lettura dei dati in un Lakehouse, è necessario selezionare esplicitamente autorizzazioni come **Leggi tutti i dati dell'endpoint SQL** o **Leggi tutto Apache Spark**.

---

## 2. Applicare Autorizzazioni Granulari (Sicurezza dei Dati)

Quando l'accesso a livello di area di lavoro o di elemento è troppo grossolano, si ricorre alle autorizzazioni granulari per proteggere specifiche tabelle, righe, colonne, cartelle o file.

### 2.1 Sicurezza tramite Endpoint di Analisi SQL (Lakehouse e Warehouse)

Ogni Lakehouse genera automaticamente un **Endpoint di Analisi SQL** che consente la transizione tra la vista _Lake_ (per l'ingegneria dei dati/Spark) e la vista _SQL_ (per l'analisi/T-SQL).

Utilizzando l'Endpoint SQL o il Warehouse, è possibile applicare la sicurezza tramite i comandi **DCL (Data Control Language)**:

- **`GRANT`:** Concede esplicitamente un'autorizzazione.
- **`DENY`:** Nega esplicitamente un'autorizzazione (ha la precedenza su `GRANT`).
- **`REVOKE`:** Rimuove un'autorizzazione precedentemente concessa o negata.

**Inoltre, è possibile implementare:**

- **Sicurezza a Livello di Riga (RLS):** Controlla l'accesso alle righe in una tabella in base all'appartenenza a gruppi o al contesto di esecuzione dell'utente (ad es., un manager vede solo i dati del suo team).
- **Sicurezza a Livello di Colonna (CLS):** Limita la visualizzazione di colonne specifiche (ad es., mascherare il codice fiscale).
- **Maschera Dati Dinamica (DDM):** Maschera i dati sensibili in una colonna, consentendo agli utenti con autorizzazioni di vederli.

**Modalità di Accesso all'Endpoint SQL (Dettaglio Avanzato):**
L'Endpoint di Analisi SQL può operare in due modalità, influenzando dove viene applicata la sicurezza:

|Modalità di Accesso|Sicurezza Applicata|Descrizione|
|---|---|---|
|**Identità Delegata (Default)**|Esclusivamente **Autorizzazioni SQL** (RLS, CLS, GRANT/DENY)|L'Endpoint SQL usa l'identità del proprietario dell'area di lavoro/elemento per accedere a OneLake. La sicurezza di OneLake viene **ignorata**.|
|**Identità Utente (SSO)**|**Ruoli di Accesso ai Dati OneLake**|L'identità dell'utente viene passata a OneLake e l'accesso in lettura è regolato dalle regole di sicurezza OneLake. Le regole OneLake vengono convertite in regole di sicurezza SQL.|

### 2.2 Ruoli di Accesso ai Dati OneLake (Sicurezza a Livello di Cartella/File)

I **Ruoli di Accesso ai Dati OneLake** offrono un modo per limitare l'accesso ai dati a livello di **cartelle** e **file** all'interno della vista _Lake_ del Lakehouse (cartelle `/Files` e `/Tables` di OneLake).

- **Meccanismo:** Si crea un ruolo personalizzato che concede l'accesso in lettura a specifiche cartelle in OneLake. La sicurezza delle cartelle è **ereditabile** nelle sottocartelle.
- **Principio:** Usa un modello di **negazione per impostazione predefinita** (Default Deny): un utente che non fa parte di un ruolo di accesso ai dati non visualizza _alcun dato_ in quel Lakehouse (a meno che non abbia un ruolo di area di lavoro più permissivo, ma questo si applica all'accesso diretto a OneLake).
- **Applicazione:** La sicurezza OneLake si applica solo agli utenti che accedono a OneLake **direttamente** (tramite Lakehouse UX, Notebook, o API).
    - Gli elementi di Fabric come l'Endpoint di Analisi SQL e i Modelli Semantici accedono a OneLake tramite una **identità delegata** e hanno i propri modelli di sicurezza.

### 2.3 Autorizzazioni del Modello Semantico

I **Modelli Semantici** (precedentemente dataset di Power BI) definiscono la sicurezza principalmente tramite **DAX**.

- È possibile applicare autorizzazioni granulari utilizzando la **Sicurezza a Livello di Riga (RLS)** definita nel Modello Semantico, che limita le righe di dati che un utente può visualizzare nei report Power BI collegati.