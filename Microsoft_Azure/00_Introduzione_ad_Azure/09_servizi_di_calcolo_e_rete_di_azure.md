
## Introduzione

Questo modulo presenta i servizi di calcolo e di rete di Azure. Verranno fornite informazioni su tre opzioni di calcolo (macchine virtuali, contenitori e Funzioni di Azure). Si apprenderanno anche alcune delle funzionalità di rete, ad esempio reti virtuali di Azure, DNS di Azure e Azure ExpressRoute.

## Obiettivi di apprendimento

Al termine di questo modulo si sarà in grado di:

- Confrontare i tipi di risorse di calcolo, tra cui istanze di contenitore, macchine virtuali e funzioni.
- Descrivere le opzioni delle macchine virtuali, tra cui macchine virtuali, set di scalabilità di macchine virtuali, set di disponibilità di macchine virtuali e Desktop virtuale Azure.
- Descrivere le risorse necessarie per le macchine virtuali.
- Descrivere le opzioni di hosting per le applicazioni, tra cui app Web di Azure, contenitori e macchine virtuali.
- Descrivere la rete virtuale, tra cui lo scopo di reti virtuali di Azure, subnet virtuali di Azure, peering, DNS di Azure, Gateway VPN ed ExpressRoute.
- Definire endpoint pubblici e privati.

## macchine virtuali di Azure

Con Macchine virtuali di Azure è possibile creare e usare macchine virtuali (VM) nel cloud. Le macchine virtuali offrono l'infrastruttura distribuita come servizio (IaaS) sotto forma di server virtualizzato e possono essere usate in molti modi. Analogamente a un computer fisico, consentono infatti di personalizzare tutto il software in esecuzione nella VM. Le macchine virtuali sono una scelta ideale quando è necessario:

- Il controllo totale del sistema operativo.
- La possibilità di eseguire software personalizzato.
- Usare configurazioni di hosting personalizzate.

Una macchina virtuale di Azure offre la flessibilità della virtualizzazione senza dover acquistare e gestire l'hardware fisico su cui è in esecuzione la macchina virtuale. Tuttavia, come per un'offerta IaaS, è necessario configurare, aggiornare e gestire il software in esecuzione nella VM.
È anche possibile creare o usare un'immagine già creata per effettuare rapidamente il provisioning delle VM. Se si seleziona un'immagine di macchina virtuale pre-configurata, è possibile creare ed effettuare il provisioning di una macchina virtuale in pochi minuti. Un'immagine è un modello usato per creare una VM e può già includere un sistema operativo e un altro software, ad esempio strumenti di sviluppo o ambienti di hosting Web.

## Dimensionare le macchine virtuali in Azure

È possibile eseguire singole macchine virtuali per attività di test, di sviluppo o secondarie oppure è possibile raggruppare le macchine virtuali per offrire disponibilità elevata, scalabilità e ridondanza. Azure può anche gestire il raggruppamento delle macchine virtuali con funzionalità come i set di scalabilità e i set di disponibilità.

### Set di scalabilità di macchine virtuali

I set di scalabilità di macchine virtuali consentono di creare e gestire un gruppo di macchine virtuali identiche con bilanciamento del carico. Se sono state semplicemente create più VM con lo stesso scopo, è necessario assicurarsi che tutte siano configurate in modo identico e quindi configurare i parametri di routing della rete per garantire l'efficienza. È anche necessario monitorare l'utilizzo per determinare se è necessario aumentare o diminuire il numero di macchine virtuali.

Con i set di scalabilità di macchine virtuali, invece, Azure automatizza la maggior parte di queste operazioni. I set di scalabilità consentono di gestire, configurare e aggiornare in modo centralizzato e veloce un numero elevato di VM. Il numero di istanze di VM può aumentare o diminuire automaticamente in risposta alla domanda o in base a una pianificazione definita. I set di scalabilità di macchine virtuali distribuiscono automaticamente anche un servizio di bilanciamento del carico per assicurarsi che le risorse vengano usate in modo efficiente. Con i set di scalabilità di macchine virtuali è possibile creare servizi su larga scala per aree quali calcolo, Big Data e carichi di lavoro contenitore.

### Set di disponibilità di macchine virtuali

I set di disponibilità delle macchine virtuali sono un altro strumento che consente di creare un ambiente più resiliente e a disponibilità elevata. I set di disponibilità sono progettati per garantire lo sfalsamento degli aggiornamenti e che le VM abbiano una connettività di rete e di alimentazione diversificata, per evitare di perdere tutte le VM in caso di guasto della rete o dell'alimentazione.

I set di disponibilità consentono di raggiungere questi obiettivi raggruppando le macchine virtuali in due modi: dominio di aggiornamento e dominio di errore.

- **Dominio di aggiornamento**: le macchine virtuali dei gruppi di dominio di aggiornamento che possono essere riavviate contemporaneamente. Questa configurazione consente di applicare gli aggiornamenti pur sapendo che un solo raggruppamento di domini di aggiornamento alla volta è offline. Tutti i computer in un dominio di aggiornamento vengono aggiornati. A un gruppo di aggiornamento sottoposto al processo di aggiornamento viene assegnato un tempo di 30 minuti per il ripristino prima dell'avvio della manutenzione per il dominio di aggiornamento successivo.
- **Dominio di errore**: il dominio di errore raggruppa le macchine virtuali in base all'alimentazione comune e al commutatore di rete. Per impostazione predefinita, un set di disponibilità suddivide le VM tra un massimo di tre domini di errore. In questo modo è possibile proteggersi da un guasto fisico dell'alimentazione o della rete, poiché le VM sono in domini di errore diversi e si possono connettere ad altre risorse di alimentazione e di rete.

In più, non sono previsti costi aggiuntivi per la configurazione di un set di disponibilità. Si paga solo per le istanze di macchina virtuale create.

## Esempi di quando usare le macchine virtuali

Alcuni esempi o casi d'uso comuni per le macchine virtuali includono:

- **Durante i test e lo sviluppo**. Le macchine virtuali consentono di creare in modo rapido e semplice diverse configurazioni del sistema operativo e dell'applicazione. Il personale dedicato a test e sviluppo può quindi eliminare facilmente le macchine virtuali quando non sono più necessarie.
- **Quando si eseguono applicazioni nel cloud**. La possibilità di eseguire determinate applicazioni nel cloud pubblico invece di creare un'infrastruttura tradizionale per eseguirle può offrire vantaggi economici considerevoli. Un'applicazione, ad esempio, potrebbe dover gestire le fluttuazioni nella domanda. La possibilità di arrestare le macchine virtuali quando non sono necessarie o di avviarle rapidamente per soddisfare un improvviso aumento della domanda consente di pagare solo per le risorse usate.
- **Quando si estende il data center al cloud**: un'organizzazione può estendere le funzionalità della propria rete locale creando una rete virtuale in Azure e aggiungendo macchine virtuali a tale rete virtuale. Le applicazioni come SharePoint possono quindi essere eseguite in una macchina virtuale di Azure invece che in locale. Questa configurazione consente di eseguire la distribuzione in modo più semplice o meno costoso che in un ambiente locale.
- **Durante il ripristino** di emergenza: come per l'esecuzione di determinati tipi di applicazioni nel cloud e l'estensione di una rete locale al cloud, è possibile ottenere risparmi significativi sui costi usando un approccio basato su IaaS per il ripristino di emergenza. Se si verifica un errore in un data center primario, è possibile creare macchine virtuali in esecuzione in Azure per eseguire le applicazioni critiche e quindi arrestarle quando il data center primario torna operativo.

## Effettuare il passaggio al cloud con le macchine virtuali

Le macchine virtuali rappresentano anche una scelta eccellente per il passaggio da un server fisico al cloud (noto anche come trasferimento in modalità lift-and-shift). È possibile creare un'immagine del server fisico e ospitarla in una macchina virtuale con poche modifiche o nessuna. La VM deve essere gestita proprio come un server fisico locale: si è responsabili della gestione del sistema operativo e del software installati.

## Risorse delle VM

Quando si effettua il provisioning di una VM, si avrà anche la possibilità di selezionare le risorse associate a tale VM, tra cui:
- Dimensioni (scopo, numero di core del processore e quantità di RAM)
- Dischi di archiviazione (unità disco rigido, unità SSD e così via)
- Rete (rete virtuale, indirizzo IP pubblico e configurazione della porta)

## il desktop virtuale di Azure

Un altro tipo di macchina virtuale è Desktop virtuale Azure. Desktop virtuale Azure è un servizio di virtualizzazione di applicazioni e desktop eseguito nel cloud. Consente di usare una versione ospitata nel cloud di Windows da qualsiasi posizione. Desktop virtuale Azure funziona tra dispositivi e sistemi operativi e funziona con le app che è possibile usare per accedere a desktop remoti o browser più moderni.

## Migliorare la sicurezza

Desktop virtuale Azure offre una gestione centralizzata della sicurezza per i desktop degli utenti con Microsoft Entra ID. È possibile abilitare l'autenticazione a più fattori per proteggere gli accessi utente. È anche possibile proteggere l'accesso ai dati assegnando controlli granulari degli accessi in base al ruolo (RBAC) agli utenti.

Con Desktop virtuale Azure, i dati e le app sono separati dall'hardware locale. Il desktop effettivo e le app sono in esecuzione nel cloud, il che significa che il rischio di lasciare dati riservati in un dispositivo personale è ridotto. Inoltre, le sessioni utente sono isolate in ambienti singoli e multisessione.

## Distribuzione di Windows 10 o Windows 11 a più sessioni

Desktop virtuale Azure consente di usare Windows 10 o Windows 11 Enterprise multisessione, l'unico sistema operativo basato su client Windows che consente più utenti simultanei in una singola macchina virtuale. Desktop virtuale Azure offre anche un'esperienza più coerente con un supporto delle applicazioni più ampio rispetto ai sistemi operativi basati su Windows Server.

## Descrivere i contenitori di Azure

Anche se le macchine virtuali sono un ottimo modo per ridurre i costi rispetto agli investimenti necessari per l'hardware fisico, sono ancora limitate a un unico sistema operativo per macchina virtuale. I contenitori sono la scelta ideale per eseguire più istanze di un'applicazione in un singolo computer host.

## Che cosa sono i contenitori?

Contenitori sono un ambiente di virtualizzazione. Così come è possibile eseguire più macchine virtuali in un singolo host fisico, si possono eseguire più contenitori in un singolo host fisico o virtuale. A differenza delle macchine virtuali, il sistema operativo di un contenitore non viene gestito dall'utente. Le macchine virtuali sono un'istanza di un sistema operativo a cui è possibile connettersi e che è possibile gestire. I contenitori sono leggeri e progettati per essere creati, aumentati e arrestati dinamicamente. È possibile creare e distribuire macchine virtuali man mano che aumenta la domanda delle applicazioni, ma i contenitori sono un metodo più leggero e più agile. I contenitori sono progettati per consentire di rispondere ai cambiamenti nella domanda. Con i contenitori è possibile eseguire un riavvio rapido in caso di arresto anomalo del sistema o di interruzione dell'hardware. Uno dei motori di contenitori più diffusi è Docker, che è supportato da Azure.

### Istanze di Azure Container

Il servizio Istanze di Azure Container rappresenta il modo più semplice e rapido per eseguire un contenitore in Azure senza dover gestire le macchine virtuali o adottare servizi aggiuntivi. Azure Container Instances sono un'offerta PaaS (Platform as a Service, piattaforma distribuita come servizio). Azure Container Instances consentono di caricare i container, che verranno quindi eseguiti dal servizio.

### App contenitore di Azure

Le app contenitore di Azure presentano molte somiglianze con un'istanza di contenitore. Essi consentono di diventare subito operativi, rimuovono la gestione dei contenitori e costituiscono un'offerta PaaS. Le app contenitore offrono ulteriori vantaggi, ad esempio la possibilità di incorporare il bilanciamento del carico e la scalabilità. Queste altre funzioni garantiscono maggiore flessibilità nella progettazione.

### Servizio Azure Kubernetes

Il servizio Azure Kubernetes è un servizio di orchestrazione dei contenitori. Un servizio di orchestrazione gestisce il ciclo di vita dei contenitori. Quando distribuisci una flotta di contenitori, AKS può rendere la gestione della flotta più semplice e in modo più efficiente.

### Usare i contenitori nelle soluzioni

I contenitori vengono spesso usati per creare soluzioni con un'architettura di microservizi. Questa architettura prevede la suddivisione delle soluzioni in parti più piccole e indipendenti. È ad esempio possibile dividere un sito Web in tre contenitori: uno che ospita il front-end, un altro per il back-end e un terzo per la risorsa di archiviazione. Ciò consente di separare i componenti dell'app in sezioni logiche che è possibile gestire, ridimensionare o aggiornare in modo indipendente.

Si supponga che il back-end del sito Web abbia raggiunto i limiti di capacità, mentre il front-end e l'archiviazione non risultano ancora sovraccaricati. Con i contenitori è possibile ridimensionare il back-end separatamente per migliorare le prestazioni. Se è necessaria una modifica di questo tipo, è anche possibile scegliere di modificare il servizio di archiviazione o il front-end senza alcun impatto sugli altri componenti.

## le funzioni di Azure

Funzioni di Azure è un servizio di calcolo serverless guidato dagli eventi che non richiede la gestione di macchine virtuali o contenitori. Se si crea un'app usando macchine virtuali o contenitori, tali risorse devono essere "in esecuzione" per consentire all'app di funzionare. Con Funzioni di Azure, un evento riattiva la funzione eliminando la necessità di mantenere il provisioning delle risorse quando non sono presenti eventi.

## Vantaggi di Funzioni di Azure

Funzioni di Azure è una soluzione ideale quando si è interessati solo al codice che esegue il servizio e non alla piattaforma o all'infrastruttura sottostante. La soluzione Funzioni viene usata comunemente quando occorre eseguire operazioni in risposta a un evento (spesso tramite una richiesta REST), un timer o un messaggio proveniente da un altro servizio di Azure e quando l'operazione può essere completata in pochi secondi al massimo.

Il servizio Funzioni offre scalabilità automatica in base alla domanda, quindi è una buona opzione quando la domanda è variabile.

Funzioni di Azure esegue il codice quando viene attivato e dealloca automaticamente le risorse al termine della funzione. In questo modello, Azure addebita solo il tempo CPU usato durante l'esecuzione della funzione.

Le funzioni possono essere con stato o senza stato. Quando sono senza stato (impostazione predefinita), si comportano come se venissero riavviate ogni volta che rispondono a un evento. Quando sono con stato (Durable Functions), viene passato un contesto tramite la funzione per tenere traccia dell'attività precedente.

Le funzioni sono un componente chiave dell'elaborazione serverless, oltre a essere una piattaforma di calcolo generale per l'esecuzione di qualsiasi tipo di codice. Se le esigenze dell'app dello sviluppatore cambiano, è possibile distribuire il progetto in un ambiente serverless. Questa flessibilità consente di gestire il ridimensionamento, di operare su reti virtuali e persino di isolare completamente le funzioni.

---
## Descrivere le opzioni di hosting per le applicazioni

Se è necessario ospitare l'applicazione in Azure, inizialmente si potrebbe scegliere di usare una macchina virtuale o contenitori. Sia le macchine virtuali che i contenitori offrono soluzioni di hosting eccellenti. Le macchine virtuali offrono il massimo controllo sull'ambiente di hosting e consentono di configurarlo esattamente come si vuole. Le macchine virtuali sono inoltre il metodo di hosting più conosciuto se non si ha familiarità con il cloud. I contenitori, con la possibilità di isolare e gestire singolarmente diversi aspetti della soluzione di hosting, possono essere un'opzione affidabile e accattivante.

Ci sono altre opzioni di hosting che è possibile usare con Azure, tra cui il servizio app di Azure.

## Servizio app di Azure

Servizio app consente di creare e ospitare app Web, processi in background, back-end per dispositivi mobili e API RESTful nel linguaggio di programmazione preferito senza gestire l'infrastruttura. Offre scalabilità automatica e disponibilità elevata. Il servizio app supporta Windows e Linux. Consente le distribuzioni automatiche da GitHub, Azure DevOps o qualsiasi repository Git per supportare un modello di distribuzione continua.

Il servizio app di Azure è un'opzione di hosting affidabile che è possibile usare per ospitare le app in Azure. Il servizio app di Azure consente di concentrarsi sulla creazione e sulla gestione dell'app, mentre Azure si occupa di mantenere l'ambiente in esecuzione.

Il Servizio app di Azure è un servizio per l'hosting di applicazioni Web, API REST e back-end mobili, basato su HTTP. Il servizio app di Azure supporta più tecnologie, tra cui linguaggi di programmazione come Java, PHP, Python e JavaScript (tramite Node.js), nonché framework come .NET e .NET Core. Servizio app di Azure supporta ambienti Windows e Linux.

### Tipi di servizi app

Con Servizio app è possibile ospitare gli stili di servizi app più comuni, ad esempio:

- App Web
- App per le API
- Processi Web
- App per dispositivi mobili

Servizio app gestisce la maggior parte delle decisioni relative all'infrastruttura che devono essere affrontate per ospitare le app accessibili dal Web:

- La distribuzione e la gestione sono integrate nella piattaforma.
- Gli endpoint possono essere protetti.
- I siti possono essere dimensionati rapidamente per gestire i carichi di traffico elevati.
- I servizi predefiniti di bilanciamento del carico predefinito e gestione del traffico garantiscono la disponibilità elevata.

Tutti questi stili di app sono ospitati nella stessa infrastruttura e condividono questi vantaggi. La flessibilità fa del servizio app la scelta ideale per ospitare applicazioni orientate al Web.

### App Web

Il servizio app include il supporto completo per l'hosting di app Web con ASP.NET, ASP.NET Core, Java, Ruby, Node.js, PHP o Python. È possibile scegliere Windows o Linux come sistema operativo host.

### App per le API

Analogamente all'hosting di un sito Web, è possibile creare API Web basate su REST usando il linguaggio e il framework preferiti. Si ottengono il supporto di Swagger completo e la possibilità di creare un pacchetto dell'API e pubblicarlo in Azure Marketplace. Le app generate possono essere utilizzate da qualsiasi client basato su HTTP o HTTP.

### Processi Web

È possibile usare la funzionalità Processi Web per eseguire un programma (EXE, Java, PHP, Python o Node.js) o uno script (CMD, BAT, PowerShell o Bash) nello stesso contesto di un'app Web, un'app per le API o un'app per dispositivi mobili. Questi processi possono essere pianificati o eseguiti da un trigger. Il servizio Processi Web viene spesso usato per eseguire attività in background come parte della logica dell'applicazione.

### App per dispositivi mobili

Usare la funzionalità App per dispositivi mobili di Servizio app per creare rapidamente un back-end per app iOS e Android. Con pochi passaggi nel portale di Azure è possibile:

- Archiviare i dati delle app per dispositivi mobili in un database SQL basato sul cloud.
- Autenticare i clienti per i provider di social networking più comuni, ad esempio account del servizio gestito, Google, X e Facebook.
- Inviare notifiche push.
- Eseguire la logica di back-end personalizzata in C# o Node.js.

Sul lato dell'app per dispositivi mobili è disponibile il supporto SDK per app iOS, Android, Xamarin e React native.

---

## Descrivere le reti virtuali di Azure

Le subnet virtuali e le reti virtuali di Azure consentono alle risorse di Azure, come macchine virtuali, app Web e database, di comunicare tra loro, con gli utenti in Internet e con i computer client locali. È possibile paragonare una rete di Azure a un'estensione della rete locale, con risorse che collegano altre risorse di Azure.

Le reti virtuali di Azure forniscono le funzionalità di rete essenziali seguenti:

- Isolamento e segmentazione
- Comunicazioni Internet
- Comunicazione tra risorse di Azure
- Comunicazione con le risorse locali
- Indirizzare il traffico di rete
- Filtri per il traffico di rete
- Connettere reti virtuali

La rete virtuale di Azure supporta endpoint sia pubblici che privati per consentire la comunicazione tra risorse esterne o interne con altre risorse interne.

- Gli endpoint pubblici hanno un indirizzo IP pubblico e sono accessibili da qualsiasi parte del mondo.
- Gli endpoint privati si trovano in una rete virtuale e hanno un indirizzo IP privato all'interno dello spazio indirizzi di tale rete virtuale.

## Isolamento e segmentazione

La rete virtuale di Azure consente di creare più reti virtuali isolate. Quando si configura una rete virtuale, si definisce uno spazio indirizzi IP privato usando intervalli di indirizzi IP pubblici o privati. L'intervallo IP esiste solo all'interno della rete virtuale e non è instradabile in Internet. È possibile dividere tale spazio indirizzi IP in subnet e allocare una parte dello spazio indirizzi definito a ogni subnet denominata.

Per la risoluzione dei nomi è possibile usare il servizio di risoluzione dei nomi integrato in Azure. È anche possibile configurare la rete virtuale per l'uso di un server DNS interno o esterno.

## Comunicazioni Internet

È possibile abilitare le connessioni in ingresso da Internet assegnando un indirizzo IP pubblico a una risorsa di Azure o usando un servizio di bilanciamento del carico pubblico.

## Comunicazione tra risorse di Azure

Si desidera consentire alle risorse di Azure di comunicare tra loro in modo sicuro. È possibile ottenere questo risultato in due modi:

- Le reti virtuali possono connettere non solo le VM ma anche altre risorse di Azure, come l'ambiente del servizio app per Power Apps, il servizio Azure Kubernetes e i set di scalabilità di macchine virtuali di Azure.
- Gli endpoint di servizio consentono la connessione ad altri tipi di risorse di Azure, come account di archiviazione e database SQL di Azure. Questo approccio consente di collegare più risorse di Azure alle reti virtuali, per migliorare la sicurezza e garantire un routing ottimale tra le risorse.

## Comunicazione con le risorse locali

Le reti virtuali di Azure consentono di collegare risorse nell'ambiente locale e nella sottoscrizione di Azure. È possibile creare una rete che si estende sia negli ambienti locali che nell'ambiente cloud. Per ottenere tale connettività sono disponibili tre meccanismi:

- Le connessioni di rete privata virtuale da punto a sito vengono stabilite tra un computer esterno all'organizzazione e la rete aziendale. In questo caso, il computer client avvia una connessione VPN crittografata per connettersi alla rete virtuale di Azure.
- Le reti private virtuali da sito a sito collegano il gateway o il dispositivo VPN locale al gateway VPN di Azure in una rete virtuale. I dispositivi in Azure possono di fatto risultare come se si trovassero nella rete locale. La connessione è crittografata e viene effettuata tramite Internet.
- Azure ExpressRoute fornisce connettività privata dedicata ad Azure, non basata su Internet. ExpressRoute è utile per gli ambienti che richiedono maggiore larghezza di banda e livelli di sicurezza ancora più elevati.

## Indirizzare il traffico di rete

Per impostazione predefinita, Azure indirizza il traffico tra subnet su qualsiasi rete virtuale connessa, sulle reti locali e su Internet. È anche possibile controllare il routing ed eseguire l'override di queste impostazioni, come indicato di seguito:

- Una tabella di route consente di definire regole per l'indirizzamento del traffico. È possibile creare tabelle di route personalizzate che controllano il modo in cui i pacchetti vengono indirizzati tra subnet.
- Border Gateway Protocol (BGP) interagisce con i gateway VPN di Azure, Server di route Azure o Azure ExpressRoute per propagare le route BGP locali nelle reti virtuali di Azure.

## Filtri per il traffico di rete

Le reti virtuali di Azure consentono di filtrare il traffico tra le subnet mediante gli approcci seguenti:

- I gruppi di sicurezza di rete sono risorse di Azure che possono contenere più regole di sicurezza in ingresso e in uscita. È possibile definire queste regole per consentire o bloccare il traffico in base a fattori come l'indirizzo IP di origine e di destinazione, la porta e il protocollo.
- Le appliance virtuali di rete sono macchine virtuali specializzate paragonabili a un'appliance di rete con protezione avanzata. Un'appliance virtuale di rete svolge una funzione di rete specifica, ad esempio l'esecuzione di un firewall o l'ottimizzazione di una rete WAN (Wide Area Network).

## Connettere reti virtuali

È possibile collegare le reti virtuali tramite il peering di reti virtuali. Il peering consente a due reti virtuali di connettersi direttamente tra loro. Il traffico di rete tra reti con peering è privato e si sposta nella rete backbone Microsoft, senza entrare mai nella rete Internet pubblica. Il peering consente alle risorse in ogni rete virtuale di comunicare tra loro. Queste reti virtuali possono trovarsi in aree separate. Questa funzionalità consente di creare una rete interconnessa globale tramite Azure.

Le route definite dall'utente consentono di controllare le tabelle di routing tra subnet all'interno di una rete virtuale o tra reti virtuali. Ciò offre un maggiore controllo sul flusso del traffico di rete.

---

## Descrivere le reti private virtuali di Azure

Una rete privata virtuale (VPN) usa un tunnel crittografato all'interno di un'altra rete. Le reti VPN vengono in genere distribuite per connettere due o più reti private attendibili attraverso una rete non attendibile, in genere la rete Internet pubblica. Il traffico sulla rete non attendibile viene crittografato per evitare intercettazioni o altri attacchi. Le VPN possono consentire alle reti di condividere in modo sicuro e protetto le informazioni sensibili.

## Gateway VPN

Un gateway VPN è un tipo di gateway di rete virtuale, Le istanze di Gateway VPN di Azure vengono distribuite in una subnet dedicata della rete virtuale e consentono la connettività seguente:

- Connettere i data center locali a Rete virtuale di Azure attraverso una connessione da sito a sito.
- Connettere i singoli dispositivi alle reti virtuali attraverso una connessione da punto a sito.
- Connettere le reti virtuali ad altre reti virtuali attraverso una connessione da rete a rete.

Tutti i dati trasferiti vengono crittografati in un tunnel privato mentre attraversano Internet. È possibile distribuire un solo gateway VPN in ogni rete virtuale. È tuttavia possibile usare un gateway per connettersi a più posizioni, tra cui altre reti virtuali o i data center locali.

Quando si configura un gateway VPN, è necessario specificare il tipo di VPN, basata su criteri o su route. La distinzione principale tra questi due tipi è il modo in cui determinano il traffico che richiede la crittografia. In Azure, indipendentemente dal tipo di VPN, il metodo di autenticazione usato è una chiave precondivisa.

- I gateway VPN basati su criteri specificano staticamente l'indirizzo IP dei pacchetti che devono essere crittografati in ogni tunnel. Questo tipo di dispositivo valuta ogni pacchetto dati in base ai set di indirizzi IP per scegliere il tunnel in cui inviare il pacchetto.
- Con i gateway basati su route, i tunnel IPSec sono modellati come interfaccia di rete o interfaccia di tunnel virtuale. Il routing IP (route statiche o protocolli di routing dinamico) stabilisce quale interfaccia di tunnel usare per l'invio di ogni pacchetto. Le VPN basate su route sono il metodo di connessione preferito per i dispositivi locali. Sono più resilienti alle modifiche della topologia, ad esempio la creazione di nuove subnet.

Usare un gateway VPN basato su route se è necessario uno dei tipi seguenti di connettività:

- Connessioni tra reti virtuali
- Connessioni da punto a sito
- Connessioni multisito
- Coesistenza con un gateway di Azure ExpressRoute

## Scenari di disponibilità elevata

Se si sta configurando una rete VPN per proteggere le informazioni, è anche necessario assicurarsi che si tratti di una configurazione VPN a disponibilità elevata e a tolleranza di errore. Ci sono alcuni modi per ottimizzare la resilienza del gateway VPN.

### Attivo/Standby

Per impostazione predefinita, i gateway VPN vengono distribuiti come due istanze in una configurazione attiva/in standby, anche se in Azure è visibile una sola risorsa gateway VPN. Se la manutenzione pianificata o un'interruzione imprevista influisce sull'istanza attiva, l'istanza in standby assume automaticamente la responsabilità delle connessioni senza alcun intervento da parte dell'utente. Le connessioni vengono interrotte durante questo failover, ma generalmente vengono ripristinate entro pochi secondi per la manutenzione pianificata ed entro 90 secondi per interruzioni impreviste.

### Attivo/Attivo

Con l'introduzione del supporto per il protocollo di routing BGP è ora possibile distribuire i gateway VPN in una configurazione di tipo Attivo/attivo. In questa configurazione si assegna un indirizzo IP pubblico univoco a ogni istanza. Si creano tunnel separati dal dispositivo locale a ogni indirizzo IP. È possibile estendere la disponibilità elevata distribuendo un dispositivo VPN aggiuntivo in locale.

### Failover di ExpressRoute

Un'altra opzione di disponibilità elevata consiste nel configurare un gateway VPN come percorso di failover protetto per le connessioni ExpressRoute. I circuiti ExpressRoute hanno la resilienza incorporata. Non sono però immuni da potenziali problemi fisici dei cavi di connettività o da interruzioni dell'intero sito ExpressRoute. Negli scenari a disponibilità elevata con rischio associato a un'interruzione di un circuito ExpressRoute, è anche possibile eseguire il provisioning di un gateway VPN che usa Internet come metodo di connettività alternativo. In questo modo è possibile garantire sempre la disponibilità di una connessione alle reti virtuali.

### Gateway con ridondanza della zona

Nelle aree che supportano le zone di disponibilità, i gateway VPN e i gateway ExpressRoute possono essere distribuiti in una configurazione con ridondanza della zona. Con questa configurazione i gateway di rete virtuale ottengono maggiore disponibilità, scalabilità e resilienza. La distribuzione di gateway in zone di disponibilità di Azure separa fisicamente e logicamente i gateway all'interno di un'area e nel contempo permette di proteggere la connettività di rete locale ad Azure da errori a livello di zona. Questi gateway richiedono Stock Keeping Unit (SKU) del gateway diversi e usano indirizzi IP pubblici Standard anziché indirizzi IP pubblici Basic.

---

# Descrivere Azure ExpressRoute

Azure ExpressRoute consente di estendere le reti locali nel cloud Microsoft tramite una connessione privata con l'ausilio di un provider di connettività. Questa connessione è detta circuito ExpressRoute. Con ExpressRoute è possibile stabilire connessioni ai servizi cloud Microsoft, come Microsoft Azure e Microsoft 365. Questa funzionalità consente di connettere uffici, data center o altre strutture al cloud Microsoft. Ogni posizione ha un circuito ExpressRoute.

La connettività può essere stabilita da una rete Any-To-Any (IP VPN), da una rete Ethernet da punto a punto o con una Cross Connection virtuale tramite un provider di connettività in una struttura di coubicazione. Le connessioni ExpressRoute non passano attraverso la rete Internet pubblica. Questa configurazione consente alle connessioni ExpressRoute di offrire maggiore affidabilità, velocità più elevate, latenze coerenti e un livello di sicurezza superiore rispetto alle tipiche connessioni Internet.

## Funzionalità e vantaggi di ExpressRoute

L'uso di ExpressRoute come servizio di connessione tra Azure e le reti locali offre diversi vantaggi.

- Connettività ai servizi cloud Microsoft in tutte le aree all'interno dell'area geopolitica.
- Connettività globale ai servizi Microsoft in tutte le aree con Copertura globale ExpressRoute.
- Routing dinamico tra la rete e Microsoft con Border Gateway Protocol (BGP).
- Ridondanza incorporata in ogni località di peering per una maggiore affidabilità.

### Connettività a servizi cloud Microsoft

ExpressRoute permette l'accesso diretto ai servizi seguenti in tutte le aree:

- Microsoft Office 365
- Microsoft Dynamics 365
- Servizi di calcolo di Azure, ad esempio Macchine virtuali di Azure
- Servizi cloud di Azure, ad esempio Azure Cosmos DB e Archiviazione di Azure

### Connettività globale

È possibile abilitare la funzionalità Copertura globale di ExpressRoute per lo scambio di dati tra i siti locali tramite la connessione dei circuiti ExpressRoute. Si supponga ad esempio di avere un ufficio in Asia e un data center in Europa, entrambi con circuiti ExpressRoute che li connettono alla rete Microsoft. È possibile usare Copertura globale ExpressRoute per connettere queste due strutture, consentendo loro di comunicare senza trasferire i dati tramite la rete Internet pubblica.

### Routing dinamico

ExpressRoute usa il protocollo BGP. BGP viene usato per scambiare route tra reti locali e risorse in esecuzione in Azure. Questo protocollo consente il routing dinamico tra la rete locale e i servizi in esecuzione nel cloud Microsoft.

### Ridondanza predefinita

Ogni provider di connettività usa dispositivi ridondanti per garantire la disponibilità elevata delle connessioni stabilite con Microsoft. È possibile configurare più circuiti per completare questa funzionalità.

## Modelli di connettività di ExpressRoute

ExpressRoute supporta quattro modelli che è possibile usare per connettere la rete locale al cloud Microsoft:

- Condivisione del percorso di CloudExchange
- Connessione Ethernet da punto a punto
- Connessione any-to-any
- Direttamente dai siti ExpressRoute

### Coubicazione in CloudExchange

La coubicazione indica che il data center, l'ufficio e altre strutture siano collocate fisicamente nella stessa struttura Cloud Exchange, ad esempio un ISP. Se una struttura è coubicata in Cloud Exchange, è possibile richiedere una connessione incrociata virtuale al cloud Microsoft.

### Connessione Ethernet da punto a punto

La connessione Ethernet da punto a punto si riferisce all'uso di una connessione da punto a punto per connettere la struttura al cloud Microsoft.

### Reti Any-to-Any

Con la connettività Any-to-Any è possibile integrare la rete WAN con Azure abilitando connessioni agli uffici e ai data center. Azure si integra con la connessione WAN per offrire una connessione come quella disponibile tra il data center e qualsiasi succursale.

### Direttamente dai siti ExpressRoute

È possibile connettersi direttamente alla rete globale di Microsoft in una località di peering strategicamente distribuita in tutto il mondo. ExpressRoute Direct fornisce doppia connettività a 100 Gbps o a 10 Gbps, che supporta la connettività attiva-attiva su larga scala.

## Considerazioni sulla sicurezza

Con ExpressRoute i dati non passano attraverso la rete Internet pubblica, riducendo i rischi associati alle comunicazioni Internet. ExpressRoute è una connessione privata dall'infrastruttura locale all'infrastruttura di Azure. Anche se si usa una connessione ExpressRoute, le query DNS, il controllo dell'elenco di revoche dei certificati e le richieste della rete per la distribuzione di contenuti di Azure vengono comunque inviati attraverso la rete Internet pubblica.

---

# Descrivere il DNS di Azure

DNS di Azure è un servizio di hosting per i domini DNS che offre la risoluzione dei nomi usando l'infrastruttura di Microsoft Azure. Ospitando i domini in Azure, è possibile gestire i record DNS usando le stesse credenziali, API, strumenti e fatturazione come per gli altri servizi Azure.

## Vantaggi di DNS di Azure

DNS di Azure sfrutta l'ambito e la scalabilità di Microsoft Azure per offrire numerosi vantaggi, tra cui:

- Affidabilità e prestazioni
- Sicurezza
- Facilità d'uso
- Reti virtuali personalizzabili
- Record alias

### Affidabilità e prestazioni

I domini DNS nel servizio DNS di Azure sono ospitati nella rete globale di Azure dei server dei nomi DNS e ciò offre resilienza e disponibilità elevata. DNS di Azure usa reti Anycast, in modo che ogni query DNS riceva una risposta dal server DNS disponibile più vicino per offrire prestazioni migliori e disponibilità elevata per il dominio.

### Sicurezza

DNS di Azure si basa su Azure Resource Manager, che offre funzionalità quali:

- Controllo degli accessi in base al ruolo di Azure per controllare gli utenti autorizzati ad accedere ad azioni specifiche per l'organizzazione.
- Log attività per monitorare il modo in cui un utente dell'organizzazione ha modificato una risorsa o per trovare un errore durante la risoluzione dei problemi.
- Blocco delle risorse per bloccare una sottoscrizione, un gruppo di risorse o una risorsa. I blocchi impediscono ad altri utenti dell'organizzazione di modificare o eliminare accidentalmente risorse di importanza fondamentale.

### Semplicità di utilizzo

DNS di Azure consente di gestire i record DNS per i servizi di Azure, nonché di garantire il servizio DNS alle risorse esterne. DNS di Azure è integrato nel portale di Azure e usa le stesse credenziali, lo stesso contratto di supporto e gli stessi metodi di fatturazione di altri servizi di Azure.
Poiché DNS di Azure è in esecuzione in Azure, è possibile gestire i domini e i record con il portale di Azure, i cmdlet di Azure PowerShell e l'interfaccia della riga di comando di Azure multipiattaforma. Le applicazioni che richiedono la gestione DNS automatica possono integrarsi con il servizio usando l'API REST e gli SDK.

### Reti virtuali personalizzabili con domini privati

DNS di Azure supporta anche i domini DNS privati. Questa funzionalità consente di usare nomi di dominio personalizzati nelle reti virtuali private, anziché dover usare i nomi forniti da Azure.

### Record alias

DNS di Azure supporta anche i set di record alias. È possibile usare un set di record alias per fare riferimento a una risorsa di Azure, come ad esempio un indirizzo IP pubblico di Azure, un profilo di Gestione traffico di Azure o un endpoint della rete per la distribuzione di contenuti (rete CDN) di Azure. Se l'indirizzo IP della risorsa sottostante viene modificato, il set di record alias si aggiorna automaticamente durante la risoluzione DNS. Il set di record alias fa riferimento all'istanza del servizio e l'istanza del servizio è associata a un indirizzo IP.

>[!Nota]
>Non è possibile usare DNS di Azure per acquistare un nome di dominio. 
>Per una tariffa annuale, è possibile acquistare un nome di dominio usando Domini del servizio app o un
>registrar di nomi di dominio di terze parti. 
>Una volta acquistati, i domini possono essere ospitati in DNS di Azure per la gestione dei record.

