---
autore: Kevin Milli
---

## Architettura di Azure: I Componenti Chiave

I componenti principali dell'architettura di Azure sono suddivisi in due raggruppamenti principali:

1. **Infrastruttura Fisica** (Data center, Aree Geografiche, Zone di Disponibilità)
2. **Infrastruttura di Gestione** (Risorse, Gruppi di Risorse, Sottoscrizioni, Gruppi di Gestione)

### Infrastruttura Fisica (Resilienza e Affidabilità)

L'infrastruttura fisica è progettata per offrire **resilienza e affidabilità** per carichi di lavoro critici.

| Componente                        | Descrizione                                                                                                                              | Funzione Chiave / Nota Operativa                                                                                                                         |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Data Center**                   | Strutture fisiche con risorse (server, alimentazione, raffreddamento, rete). Sono i mattoni di base.                                     | Non sono accessibili direttamente; sono raggruppati in Zone/Aree.                                                                                        |
| **Aree (Regions)**                | Area geografica che contiene **almeno uno, ma potenzialmente più data center** vicini, collegati da una rete a bassa latenza.            | **Operativo:** Quando si distribuisce una risorsa, è spesso necessario scegliere l'Area. Alcuni servizi sono disponibili solo in determinate Aree.       |
| **Zone di Disponibilità (AZ)**    | **Data center separati fisicamente** all'interno di un'Area di Azure, con alimentazione, raffreddamento e rete indipendenti.             | Forniscono un **limite di isolamento**: se una Zona si guasta, le altre continuano a funzionare. Devono essercene **almeno tre** in ogni Area abilitata. |
| **Coppie di Aree (Region Pairs)** | Due Aree (es. Stati Uniti occidentali e Stati Uniti orientali) all'interno della stessa collocazione geografica, distanti almeno 480 km. | Offrono **maggiore resilienza** in caso di disastri che colpiscono un'intera Area. Priorità all'aggiornamento e al ripristino di un'Area alla volta.     |
| **Aree Sovrane**                  | Istanze di Azure isolate dall'istanza principale, utilizzate per scopi legali o di conformità (es. US Gov, Cina).                        | Gestite in modo speciale per conformità normativa.                                                                                                       |

#### Utilizzo Operativo delle Zone di Disponibilità

Per le app a disponibilità elevata, le AZ garantiscono ridondanza:

- **Servizi di Zona:** Risorse (VM, IP) associate a una Zona specifica.
- **Servizi con Ridondanza della Zona:** La piattaforma replica automaticamente applicazioni e dati tra le Zone (es. Archiviazione con ridondanza della zona).
- **Servizi Non a Livello di Area:** Sempre disponibili e resilienti (es. Microsoft Entra ID).

---

## Infrastruttura di Gestione (Organizzazione Gerarchica)

Comprendere la gerarchia di gestione è fondamentale per pianificare l'accesso, la fatturazione e l'applicazione di policy.

### 1. Risorse e Gruppi di Risorse (RG)

- **Risorsa:** È il **blocco predefinito di base** di Azure. Tutto ciò che si crea (VM, reti, database, servizi cognitivi) è una Risorsa.
- **Gruppo di Risorse (RG):** Un raggruppamento logico di Risorse.

|Caratteristiche Operative|Dettagli Importanti|
|---|---|
|**Associazione**|Una Risorsa appartiene a **un solo** RG alla volta, ma può essere spostata.|
|**Annidamento**|I RG **non possono essere annidati** (non puoi mettere un RG dentro un altro).|
|**Gestione**|L'azione applicata a un RG (es. **Eliminazione**) viene applicata a **tutte le Risorse** al suo interno.|
|**Accesso**|Concedere o negare l'accesso a un RG si applica a tutte le Risorse al suo interno.|

**Guida Operativa:** Raggruppa le risorse in base al ciclo di vita, allo schema di accesso o alla funzione (es. un RG per un ambiente di sviluppo temporaneo).

### 2. Sottoscrizioni (Subscriptions)

Le Sottoscrizioni sono un'unità di **gestione, fatturazione e scalabilità** in Azure. Raggruppano logicamente i Gruppi di Risorse.

- **Obbligo:** L'uso di Azure richiede una Sottoscrizione, collegata a un **Account Azure** (un'identità in Microsoft Entra ID). Un Account può avere più Sottoscrizioni.
- **Limiti:** Le sottoscrizioni definiscono due tipi di limiti:
    - **Limite di Fatturazione:** Determina come l'Account Azure viene fatturato. Utile per organizzare e gestire i costi per dipartimento o progetto.
    - **Limite di Controllo Accessi:** I criteri di gestione degli accessi (Governance) vengono applicati a livello di Sottoscrizione.

Guida Operativa: Creare Sottoscrizioni Aggiuntive

È utile creare più Sottoscrizioni per separare:

- **Ambienti:** Dev/Test, Produzione, Sicurezza, Conformità.
- **Strutture Organizzative:** Limitare un team a risorse a basso costo.
- **Fatturazione:** Tracciare i costi separatamente (es. Produzione vs. Dev/Test).

### 3. Gruppi di Gestione (Management Groups - MG)

I MG forniscono un **livello di ambito superiore** alle Sottoscrizioni.

- **Gerarchia:** Organizzano le Sottoscrizioni in contenitori per applicare la governance su larga scala.
- **Ereditarietà:** Tutte le Sottoscrizioni all'interno di un MG **ereditano automaticamente** le condizioni (policy, accesso) applicate al MG.
- **Annidamento:** I MG **possono essere annidati** (fino a sei livelli, escludendo radice e sottoscrizione).

**Guida Operativa: Usare i Gruppi di Gestione**

1. **Applicare Policy su larga scala:** Limitare le posizioni delle VM o imporre policy di sicurezza a tutte le sottoscrizioni in un MG (es. "Produzione"). Queste policy **non possono essere modificate** a livello di risorsa/sottoscrizione.
2. **Fornire Accesso Utente a più Sottoscrizioni:** Un'assegnazione di controllo degli accessi in base al ruolo (RBAC) a livello di MG si propaga (eredita) a tutti i livelli sottostanti.

### Riepilogo Gerarchico

Gruppi di Gestione→Sottoscrizioni→Gruppi di Risorse→Risorse

---

## Guida Operativa: Interagire con Azure

È possibile interagire con Azure tramite il **Portale di Azure** (interfaccia grafica) e l'**Interfaccia della Riga di Comando (CLI)** tramite **Cloud Shell**.

### 1. Azure Cloud Shell

Cloud Shell è un terminale basato sul browser accessibile dal Portale di Azure (icona ☁️).

|Modalità|Inizio Prompt|Comandi di Base|Comandi Azure (Entrambe le modalità)|
|---|---|---|---|
|**PowerShell**|`PS`|`Get-date`|Iniziano con **`az`** (es. `az version`)|
|**BASH**|`name@azure`|`date`|Iniziano con **`az`** (es. `az upgrade`)|

**Passare tra le modalità:** Usa `BASH` o `PWSH` nel prompt dei comandi.

### 2. Modalità Interattiva della CLI

- **Attivazione:** Immetti `az interactive`.
- **Vantaggi:** Rende la CLI più simile a un ambiente di sviluppo integrato (IDE): fornisce completamento automatico, descrizioni dei comandi ed esempi.
- **Nota Operativa:** In modalità interattiva, **non è necessario** digitare `az` per avviare un comando Azure (es. digita `version` invece di `az version`).
- **Uscita:** Usa il comando `exit`.

---

## Esercizio Pratico: Creare e Gestire una Macchina Virtuale (VM)

Questo esercizio illustra la creazione di una **Risorsa** (la VM) all'interno di un **Gruppo di Risorse**.

### 1. Creazione della Macchina Virtuale (VM)

1. Accedi al **Portale di Azure**.
2. Seleziona **Crea una risorsa** > **Compute** > **Macchina Virtuale** (o cerca "Virtual Machine").
3. **Scheda Informazioni di base:** Inserisci i seguenti valori:

| Impostazione                             | Valore                                                  | Note Operative                                                         |
| ---------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Sottoscrizione**                       | Seleziona la tua Sottoscrizione.                        | Determina l'addebito.                                                  |
| **Gruppo di risorse**                    | Seleziona **Crea nuovo** e immetti `IntroAzureRG`.      | **Importante:** Questo RG conterrà tutte le risorse associate alla VM. |
| **Nome della macchina virtuale**         | `my-VM`                                                 | Nome identificativo della Risorsa principale.                          |
| **Area**                                 | Lascia l'impostazione predefinita o scegli la tua Area. | Determina dove risiederà fisicamente la VM.                            |
| **Opzioni zona / Zona di disponibilità** | Lascia l'impostazione predefinita.                      | Opzioni per la resilienza fisica (vedi AZ sopra).                      |
| **Tipo di autenticazione**               | Password                                                |                                                                        |
| **Nome utente**                          | `azureuser`                                             |                                                                        |
| **Password**                             | Immetti e conferma una password personalizzata.         |                                                                        |
| **Porte in ingresso pubbliche**          | Nessuna                                                 | Per maggiore sicurezza, si consiglia "Nessuna".                        |
4. Seleziona **Rivedi e crea**.
5. Seleziona **Crea** e attendi il completamento della distribuzione.

### 2. Verifica delle Risorse Create

La creazione di una singola VM comporta la creazione automatica di più **Risorse** correlate (es. disco, rete virtuale, IP) e il loro raggruppamento.

1. Seleziona **Home** nel Portale.
2. In **Servizi di Azure**, seleziona **Gruppi di risorse**.
3. Seleziona il Gruppo di Risorse **`IntroAzureRG`**.
4. Vedrai l'elenco di tutte le Risorse create per la VM, raggruppate logicamente sotto il nome del RG.

### 3. Eseguire la Pulizia (Fondamentale per i Costi)

Per evitare costi non necessari, specialmente se non sei in un ambiente di "sandbox" a consumo zero, devi eliminare il Gruppo di Risorse, che elimina tutte le Risorse al suo interno.

1. Nel Portale, vai al Gruppo di Risorse **`IntroAzureRG`**.
2. Seleziona **Elimina gruppo di risorse**.
3. Immetti **`IntroAzureRG`** per confermare l'eliminazione.
4. Seleziona **Elimina**.


---
---

## Introduzione agli account Azure

Per creare e usare i servizi di Azure, è necessaria una sottoscrizione di Azure. Quando tuttavia si usano le proprie applicazioni ed esistono specifiche esigenze aziendali da soddisfare, è necessario creare un account Azure. La società, ad esempio, potrebbe usare un singolo account Azure per l'azienda e sottoscrizioni separate per i reparti di sviluppo, marketing e vendite. Dopo aver creato una sottoscrizione di Azure, è possibile iniziare a creare le risorse di Azure all'interno di ogni sottoscrizione.

![[image-17.png|385x304]]

## Usare l'interfaccia della riga di comando

È possibile usare l'interfaccia della riga di comando dal portale di Azure. Dopo aver eseguito l'accesso ad Azure, accedere all'interfaccia della riga di comando selezionando l'icona di Cloud Shell. L'avvio di Cloud Shell consente di visualizzare una finestra dell'interfaccia della riga di comando in modalità PowerShell o BASH. Se si ha familiarità con PowerShell, è possibile gestire l'ambiente di Azure usando i comandi di PowerShell.

Per accedere a CloudShell dal portale di Azure, selezionare l'icona cloudShell.

![[image-18.png|549x44]]

È possibile passare rapidamente da PowerShell a BASH nell'interfaccia della riga di comando selezionando il pulsante **Passa a ...** o immettendo `BASH` o `PWSH`.

![[image-19.png|563x151]]

>[!Nota]
>Quando si usa la modalità PowerShell, la riga di comando inizia con PS. 
>Quando si usa la modalità BASH, la riga di comando inizia con l'utente name@azure.

### Usare PowerShell nell'interfaccia della riga di comando

Usare il comando di PowerShell `Get-date` per ottenere la data e l'ora correnti.

Il comando Get-date è un comando specifico di PowerShell. La maggior parte dei comandi specifici di Azure inizia con le lettere **az**. Provare ora un comando di Azure per verificare la versione dell'interfaccia della riga di comando in uso.

```powershell
az version
```

### Usare BASH nell'interfaccia della riga di comando

Se si ha familiarità con BASH, è possibile usare invece i comandi BASH passando all'interfaccia della riga di comando di BASH.
Immettere `bash` per passare all'interfaccia della riga di comando di BASH.

![[image-20.png|544x55]]

In `Bash` è  `date` il comando per ottenere la data e l'ora correnti.

Proprio come nella modalità PowerShell dell'interfaccia della riga di comando, usare le lettere az per avviare un comando di Azure in modalità BASH. 
Provare a eseguire un aggiornamento dell'interfaccia della riga di comando con `az upgrade`.
È possibile tornare alla modalità PowerShell immettendo il comando pwsh nella riga di comando di BASH.

## Usare la modalità interattiva dell'interfaccia della riga di comando di Azure

Un altro modo per interagire con la sandbox consiste nell'usare la modalità interattiva dell'interfaccia della riga di comando di Azure. Questa modalità rende il comportamento dell'interfaccia della riga di comando più simile a un ambiente di sviluppo integrato (IDE). La modalità interattiva fornisce il completamento automatico, le descrizioni dei comandi e anche alcuni esempi. Se non si ha familiarità con BASH e PowerShell, ma si vuole usare la riga di comando, la modalità interattiva può essere la soluzione.

Immettere `az interactive` per attivare la modalità interattiva.

Decidere se inviare i dati di telemetria e immettere `YES` o `NO`.

Potrebbe essere necessario attendere un paio di minuti per consentire l'inizializzazione completa della modalità interattiva. Immettere quindi la lettera "a" per attivare il completamento automatico. Se il completamento automatico non funziona, attendere un po' più a lungo e riprovare.

![[image-21.png|506x306]]

Dopo l'inizializzazione, è possibile usare i tasti di direzione o il tasto TAB per completare i comandi. La modalità interattiva è configurata in modo specifico per Azure, quindi non è necessario immettere az per avviare un comando. Provare di nuovo i comandi `upgrade` o `version` , ma questa volta senza az in primo piano.

I comandi dovrebbero funzionare come prima e restituire gli stessi risultati. Usare il comando `exit` per lasciare la modalità interattiva.

### Zone di disponibilità

Le zone di disponibilità sono data center separati fisicamente all'interno di un'area di Azure. Ogni zona di disponibilità è costituita da uno o più data center dotati di impianti indipendenti per l'energia, il raffreddamento e la rete. Una zona di disponibilità è configurata per essere un limite di isolamento. Se una zona diventa inattiva, l'altra continua a funzionare. Le zone di disponibilità sono connesse tramite reti in fibra ottica private ad alta velocità.

![[image-22.png|319x318]]

>[!Nota]
>Per assicurare la resilienza, sono presenti almeno tre zone di disponibilità separate in tutte le aree abilitate per le zone di disponibilità. 
>Tuttavia, non tutte le aree di Azure attualmente supportano le zone di disponibilità.

### Coppie di aree

La maggior parte delle aree di Azure è associata a un'altra area con la stessa collocazione geografica (ad esempio Stati Uniti, Europa o Asia) ad almeno 480 km di distanza. Questo approccio consente la replica delle risorse in due diverse aree geografiche in modo da ridurre la probabilità di interruzioni dovute a eventi come calamità naturali, agitazioni, interruzioni di corrente o interruzioni della rete fisica che interessano un'intera area. Ad esempio, se un'area dovesse essere interessata da una calamità naturale, verrebbe eseguito automaticamente il failover dei servizi nell'altra area della coppia.

>[!Nota]
>Non tutti i servizi di Azure replicano automaticamente i dati o eseguono automaticamente il fallback da un'area problematica per eseguire la replica incrociata in un'altra area abilitata. 
>In questi scenari il ripristino e la replica devono essere configurati dal cliente.

Esempi di coppie di aree in Azure sono Stati Uniti occidentali e Stati Uniti orientali o Asia sud-orientale e Asia orientale. Poiché la coppia di aree è direttamente connessa e le aree sono sufficientemente distanti da essere isolate dalle situazioni di emergenza a livello di area, queste possono essere usate per garantire la ridondanza dei servizi e dei dati.

![[image-23.png|549x304]]

## Descrivere l'infrastruttura di gestione di Azure

L'infrastruttura di gestione include risorse e gruppi di risorse, sottoscrizioni e account di Azure. Se si comprende l'organizzazione gerarchica diventa più facile pianificare i progetti e i prodotti in Azure.

## Risorse e gruppi di risorse di Azure

Una risorsa è il blocco predefinito di base di Azure. Tutto ciò che si crea, di cui si effettua il provisioning, che si distribuisce e così via è una risorsa. Macchine virtuali (VM), reti virtuali, database, servizi cognitivi e così via sono tutti considerati risorse all'interno di Azure.

![[image-24.png|371x164]]

I gruppi di risorse sono semplicemente raggruppamenti di risorse. Quando si crea una risorsa, è necessario inserirla in un gruppo di risorse. Anche se un gruppo di risorse può contenere molte risorse, una risorsa può appartenere a un solo gruppo di risorse alla volta. È possibile spostare le risorse tra gruppi di risorse, ma la risorsa che viene spostata in un nuovo gruppo non sarà più associata al gruppo precedente. Inoltre i gruppi di risorse non possono essere annidati. Questo significa che non è possibile inserire il gruppo di risorse B all'interno del gruppo di risorse A.

I gruppi di risorse offrono un modo pratico per raggruppare le risorse. Quando si applica un'azione a un gruppo di risorse, tale azione verrà applicata a tutte le risorse all'interno del gruppo. Se si elimina un gruppo di risorse, verranno eliminate anche tutte le risorse al suo interno. Se si concede o si nega l'accesso a un gruppo di risorse, si concede o si nega l'accesso a tutte le risorse all'interno del gruppo.

Quando si effettua il provisioning delle risorse, è consigliabile considerare la struttura del gruppo di risorse più adatta alle proprie esigenze.

Se ad esempio si configura un ambiente di sviluppo temporaneo, raggruppando tutte le risorse è possibile eseguire il deprovisioning contemporaneamente di tutte le risorse associate eliminando il gruppo di risorse. Se si effettua il provisioning di risorse di calcolo che richiederanno tre schemi di accesso diversi, potrebbe essere più utile raggruppare le risorse in base allo schema di accesso e quindi assegnare l'accesso a livello di gruppo di risorse.

Non esistono regole rigide su come usare i gruppi di risorse, quindi è bene valutare come configurare i gruppi di risorse per massimizzarne l'utilità per le proprie esigenze.

## Sottoscrizioni Azure

Le sottoscrizioni sono un'unità di gestione, fatturazione e scalabilità in Azure. Così come i gruppi di risorse sono un modo per organizzare in modo logico le risorse, le sottoscrizioni consentono di organizzare logicamente i gruppi di risorse e di facilitare la fatturazione.

![[image-25.png|457x188]]

L'uso di Azure richiede una sottoscrizione di Azure. Con una sottoscrizione si ottiene l'accesso autenticato e autorizzato ai servizi e ai prodotti di Azure, oltre alla possibilità di effettuare il provisioning delle risorse. Una sottoscrizione di Azure è collegata a un account Azure, ovvero un'identità in Microsoft Entra ID o in una directory considerata attendibile da Microsoft Entra ID.

Un account può avere più sottoscrizioni, ma deve necessariamente averne almeno una. In un account con più sottoscrizioni è possibile usare le sottoscrizioni per configurare modelli di fatturazione diversi e applicare criteri di gestione degli accessi diversi. È possibile usare le sottoscrizioni di Azure per definire i limiti relativi a prodotti, servizi e risorse di Azure. È possibile usare due tipi di limiti delle sottoscrizioni:

- **Limite di fatturazione**: questo tipo di sottoscrizione determina la modalità di fatturazione di un account Azure per l'uso di Azure. È possibile creare più sottoscrizioni per diversi tipi di requisiti di fatturazione. Azure genera report di fatturazione e fatture distinti per ogni sottoscrizione, in modo da poter organizzare e gestire i costi.
- **Limite di controllo** di accesso: Azure applica i criteri di gestione degli accessi a livello di sottoscrizione ed è possibile creare sottoscrizioni separate per riflettere diverse strutture organizzative. Ad esempio, all'interno di un'azienda è possibile applicare criteri di sottoscrizione di Azure distinti ai diversi reparti. Questo modello di fatturazione consente di gestire e controllare l'accesso alle risorse di cui gli utenti effettuano il provisioning con sottoscrizioni specifiche.

## Gerarchia di gruppi di gestione, sottoscrizioni e gruppi di risorse

È possibile creare una struttura flessibile di gruppi di gestione e sottoscrizioni, in modo da organizzare le risorse in una gerarchia per la gestione unificata di accesso e criteri. Il diagramma seguente mostra un esempio di creazione di una gerarchia per la governance tramite gruppi di gestione.

![[image-26.png|449x281]]

Alcuni esempi di come usare i gruppi di gestione possono essere:

- **Creare una gerarchia che applica un criterio**. È ad esempio possibile limitare le posizioni delle macchine virtuali all'area Stati Uniti occidentali in un gruppo chiamato Produzione. Questo criterio erediterà da tutte le sottoscrizioni discendenti del gruppo di gestione e verrà applicato a tutte le macchine virtuali all'interno delle sottoscrizioni. Questi criteri di sicurezza, inoltre, non possono essere modificati dal proprietario della risorsa o della sottoscrizione e ciò consente una governance migliore.
- **Fornire l'accesso utente a più sottoscrizioni**. Spostando più sottoscrizioni all'interno di un gruppo di gestione, è possibile creare un'assegnazione di controllo degli accessi in base al ruolo di Azure nel gruppo di gestione. L'assegnazione del controllo degli accessi in base al ruolo di Azure a livello di gruppo di gestione significa che anche tutti i gruppi di sottogestione, le sottoscrizioni, i gruppi di risorse e le risorse sottostanti erediteranno tali autorizzazioni. Una sola assegnazione nel gruppo di gestione può consentire agli utenti di accedere a tutte le risorse necessarie invece di eseguire script di controllo degli accessi in base al ruolo di Azure per diverse sottoscrizioni.

Informazioni importanti sui gruppi di gestione:

- Una singola directory può supportare 10.000 gruppi di gestione.
- L'albero di un gruppo di gestione può supportare fino a sei livelli di profondità. Questo limite non include il livello radice o il livello di sottoscrizione.
- Ogni gruppo di gestione e sottoscrizione può supportare un solo elemento padre.

## Creare una macchina virtuale

In questa attività si creerà una macchina virtuale usando il portale di Azure.

1. Accedere al [portale di Azure](https://portal.azure.com/).
2. Selezionare Create a resource > Compute > Virtual Machine Create (Crea macchina > virtuale di calcolo risorse).
3. Verrà visualizzato il riquadro Crea macchina virtuale nella scheda Informazioni di base.    
4. Verificare o immettere i valori seguenti per ogni impostazione. Se un'impostazione non è specificata, lasciare il valore predefinito.


**Scheda Informazioni di base**

|**Impostazione**|**Valore**|
|---|---|
|Sottoscrizione|Selezionare la sottoscrizione da usare per l'esercizio.|
|Gruppo di risorse|Selezionare Crea nuovo e immettere `IntroAzureRG` e selezionare OK|
|Nome della macchina virtuale|`my-VM`|
|Area|Lasciare l'impostazione predefinita|
|Opzioni di disponibilità|Lasciare l'impostazione predefinita|
|Opzioni zona|Zona auto-selezionata|
|Zona di disponibilità|Lasciare l'impostazione predefinita|
|Tipo di sicurezza|Lasciare l'impostazione predefinita|
|Immagine|Lasciare l'impostazione predefinita|
|Architettura della macchina virtuale|Lasciare l'impostazione predefinita|
|Eseguire con sconto di Spot Azure|Deselezionata|
|Dimensione|Lasciare l'impostazione predefinita|
|Tipo di autenticazione|Password|
|Nome utente|`azureuser`|
|Password|Immettere una password personalizzata|
|Conferma password|Immettere di nuovo la password personalizzata|
|Porte in ingresso pubbliche|Nessuna|
5. Selezionare Rivedi e crea.

>[!Nota]
>I dettagli del prodotto includeranno un costo associato alla creazione della macchina virtuale. 
>Si tratta di una funzione di sistema. Se si sta creando la macchina virtuale nella sandbox di Learn, non verranno addebitati costi.

6. Selezionare Crea

Attendere il provisioning della macchina virtuale. Quando la macchina virtuale è pronta, il messaggio La distribuzione è in corso cambierà in La distribuzione è stata completata.

## Verificare le risorse create

Una volta creata la distribuzione, è possibile verificare che Azure abbia creato non solo una macchina virtuale, ma anche tutte le risorse associate necessarie per la macchina virtuale.

1. Selezionare Home.
2. In Servizi di Azure selezionare Gruppi di risorse.
3. Selezionare il gruppo di **risorse IntroAzureRG** .

Verrà visualizzato un elenco delle risorse nel gruppo di risorse. Le risorse sono state create al momento della creazione della macchina virtuale. Per impostazione predefinita, Azure ha assegnato a tutte le risorse un nome simile per facilitare l'associazione e le ha raggruppate nello stesso gruppo di risorse.

Congratulazioni. È stata creata una risorsa in Azure ed è stato possibile osservare in che modo le risorse vengono raggruppate al momento della creazione.

## Eseguire la pulizia

Per pulire gli asset creati in questo esercizio ed evitare costi non necessari, eliminare il gruppo di risorse e tutte le risorse associate.

1. Nella home page di Azure, in Servces di Azure selezionare **Gruppi di risorse**.
2. Selezionare il gruppo di **risorse IntroAzureRG** .
3. Selezionare **Elimina gruppo di risorse**.
4. Immettere `IntroAzureRG` per confermare l'eliminazione del gruppo di risorse e selezionare Elimina.

