
## Obiettivi di Apprendimento (Cosa Saprai Fare)

Dopo aver consultato questi appunti, sarai in grado di:

- **Interagire con Azure** utilizzando il **Portale di Azure** (GUI) e **Azure Cloud Shell** (CLI/PowerShell).
- **Gestire risorse non-Azure** (on-premises, altri cloud) utilizzando **Azure Arc**.
- Comprendere la funzione di **Azure Resource Manager (ARM)** come servizio centrale di gestione e i **Modelli ARM/Bicep** per l'**Infrastructure as Code (IaC)**.

## Strumenti Essenziali per Interagire con Azure

Azure offre diverse interfacce per la gestione:

|Strumento|Tipo|Uso Principale|Piattaforme|Vantaggi Operativi|
|---|---|---|---|---|
|**Portale di Azure**|Interfaccia Grafica (Web)|Gestione **visiva** e **monitoraggio** interattivo di risorse complesse.|Browser (Cloud-based)|Ideale per la **gestione occasionale**, monitoraggio (Dashboard) e utenti meno inclini alla riga di comando. **Alta disponibilità** (presente in ogni data center).|
|**Azure Cloud Shell**|Shell **Basata su Browser**|Esecuzione di comandi PowerShell/Bash per creare, configurare e gestire risorse.|Browser (Integrato nel Portale)|**Nessuna installazione richiesta**, **autenticato** con credenziali Azure, supporta **PowerShell** e **CLI di Azure** (Bash).|
|**Azure PowerShell**|Shell (Cmdlet)|Esecuzione di **cmdlet** per chiamare l'API REST di Azure, scripting e automazione.|Cloud Shell, Windows, Linux, Mac|Ottimo per **professionisti IT/DevOps** abituati all'ambiente PowerShell; automazione di **azioni complesse** (es. distribuzione di infrastrutture intere).|
|**Interfaccia della riga di comando di Azure (CLI)**|Shell (Bash/Comandi)|Esecuzione di **comandi Bash** per la gestione e l'orchestrazione di attività.|Cloud Shell, Windows, Linux, Mac|Ottimo per chi ha familiarità con **Bash/Linux**; gli stessi vantaggi di scripting e automazione di Azure PowerShell.|

### Che cos'è Azure Cloud Shell?

Cloud Shell è una **shell basata su browser** (Bash o PowerShell) accessibile direttamente dal Portale di Azure (icona Cloud Shell). 
È lo strumento ideale per iniziare subito a usare la riga di comando **senza configurazioni locali**.

## Azure Resource Manager (ARM)

**Azure Resource Manager (ARM)** è il **servizio di distribuzione e gestione centrale** per Azure. 
Qualsiasi interazione con le risorse di Azure (tramite portale, CLI, SDK, ecc.) passa attraverso ARM.

### Funzione Operativa di ARM

ARM funge da livello di gestione unificato:

1. **Riceve** le richieste da qualsiasi strumento Azure.
2. **Autentica** e **autorizza** la richiesta (grazie a **RBAC** integrato).
3. **Invia** la richiesta al servizio Azure appropriato.

Questo garantisce **risultati e funzionalità coerenti** indipendentemente dallo strumento utilizzato.

### Vantaggi Operativi di ARM

- **Gestione Collettiva:** Distribuisci, gestisci e monitora **tutte le risorse per una soluzione come un gruppo** (Gruppo di Risorse).
- **Controllo Accessi:** **RBAC (Role-Based Access Control)** integrato nativamente per applicare il controllo accessi in modo coerente.
- **Organizzazione:** Applica **tag** alle risorse per organizzare logicamente e semplificare la **fatturazione** (visualizzando i costi per tag/gruppo di risorse).

## Infrastructure as Code (IaC)

IaC è la pratica di gestire e configurare l'infrastruttura come righe di codice. È un concetto fondamentale per **automazione, ripetibilità** e **coerenza** delle distribuzioni.
I principali strumenti IaC in Azure sono i **Modelli ARM** e **Bicep**.

### Modelli ARM (Azure Resource Manager Templates)

- **Cos'è:** File **JSON dichiarativi** che definiscono le risorse da distribuire in Azure.
- **Vantaggi Operativi Chiave:**
    - **Sintassi Dichiarativa:** Si dichiara **cosa** si vuole (lo stato desiderato) senza scrivere la sequenza di comandi (imperativi) per ottenerlo.
    - **Orchestrazione:** ARM orchestra automaticamente l'ordine di distribuzione, gestendo le dipendenze e creando le risorse **in parallelo** per una distribuzione più rapida.
    - **Risultati Ripetibili:** Garantisce che l'infrastruttura venga distribuita in uno **stato coerente**, ideale per creare più ambienti (dev/test/prod) identici.
    - **Modularità:** Possibilità di suddividere e **annidare** modelli per riutilizzo e semplificazione.

### Bicep

- **Cos'è:** Un linguaggio che usa una sintassi dichiarativa **più semplice e concisa** rispetto al JSON dei Modelli ARM.
- **Vantaggi Aggiuntivi rispetto ad ARM JSON:**
    - **Sintassi Semplice:** Più facile da leggere e scrivere; non richiede conoscenze pregresse di linguaggi di programmazione.
    - **Supporto Immediato:** Supporta immediatamente **tutti i tipi di risorse e le versioni API** di Azure, inclusi anteprime e GA (General Availability).
    - **Modularità Avanzata:** Ottimo supporto per i **moduli**, che consentono di riutilizzare il codice in modo efficiente.
    - **Idempotenza:** La distribuzione dello stesso file produce gli stessi tipi di risorse nello stesso stato, permettendo di concentrarsi sullo stato desiderato.

## Azure Arc

Azure Arc estende le funzionalità di **governance** e **gestione** di Azure ad **ambienti non-Azure**, semplificando la gestione ibrida e multi-cloud.

### Scopo Operativo di Azure Arc

Consente di usare **Azure Resource Manager (ARM)** per gestire risorse che risiedono **all'esterno** di Azure (es. un datacenter locale o un altro cloud provider).

- **Piattaforma Unificata:** Offre una piattaforma di gestione **coerente** per ambienti locali e multi-cloud.
- **Estensione di Azure:** Proietta le risorse non-Azure in ARM, permettendo di gestirle e monitorarle **come se fossero in esecuzione in Azure**.
- **Uso di Funzionalità Azure:** Permette di usare familiari servizi di gestione Azure (monitoraggio, sicurezza, configurazione) indipendentemente dalla loro posizione.

### Tipi di Risorse Gestibili all'Esterno di Azure (Tramite Arc)

- **Server**
- **Cluster Kubernetes**
- **Servizi Dati di Azure** (es. PostgreSQL, MySQL)
- **SQL Server**
- **Macchine virtuali** (in anteprima)
