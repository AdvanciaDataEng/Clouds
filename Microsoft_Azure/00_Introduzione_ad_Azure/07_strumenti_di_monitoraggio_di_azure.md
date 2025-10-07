---
autore: Kevin Milli
---

## Introduzione

In questo modulo verranno presentati gli strumenti che consentono di monitorare l'ambiente e le applicazioni, sia in Azure che in ambienti locali o multicloud.

## Obiettivi di apprendimento

Dopo aver completato questo modulo, sarà possibile:

- Descrivere lo scopo di Azure Advisor.
- Descrivere l'integrità dei servizi di Azure.
- Descrivere Monitoraggio di Azure, tra cui Azure Log Analytics, Avvisi di Monitoraggio di Azure e Application Insights.

## Descrivere lo scopo di Azure Advisor

Azure Advisor valuta le risorse di Azure e fornisce raccomandazioni per migliorare l'affidabilità, la sicurezza e le prestazioni, ottenere l'eccellenza operativa e ridurre i costi.

Quando ci si trova nel portale di Azure, il dashboard di Advisor visualizza raccomandazioni personalizzate per tutte le sottoscrizioni. È possibile usare i filtri per selezionare le raccomandazioni per sottoscrizioni, gruppi di risorse o servizi specifici. Le raccomandazioni si dividono in cinque categorie:

- **L'affidabilità** viene usata per garantire e migliorare la continuità delle applicazioni aziendali critiche.
- **La sicurezza** viene usata per rilevare minacce e vulnerabilità che potrebbero causare violazioni della sicurezza.
- **Le prestazioni** vengono usate per migliorare la velocità delle applicazioni.
- **L'eccellenza operativa** viene usata per ottenere efficienza del processo e del flusso di lavoro, gestibilità delle risorse e procedure consigliate per la distribuzione.
- **Il costo** viene usato per ottimizzare e ridurre la spesa complessiva di Azure.

L'immagine seguente mostra il dashboard di Azure Advisor.
![[image-14.png|572x355]]

## Integrità dei servizi di Azure

Azure consente di monitorare lo stato dell’infrastruttura globale e delle singole risorse tramite **Integrità dei servizi di Azure**, che unisce tre strumenti:

- **Stato di Azure**: panoramica globale dello stato dei servizi in tutte le aree, utile per identificare interruzioni diffuse.
- **Integrità dei servizi**: visione mirata ai servizi e alle aree in uso, con notifiche su interruzioni, manutenzioni pianificate e raccomandazioni. Permette di configurare avvisi personalizzati.
- **Integrità risorse**: monitoraggio puntuale delle singole risorse (es. VM). Con Azure Monitor si possono impostare avvisi sulle variazioni di disponibilità.

Questi strumenti offrono una visione completa, dal livello globale fino alle risorse specifiche. Gli avvisi vengono archiviati per analisi successive, utili a rilevare tendenze. In caso di eventi che impattano i carichi di lavoro, sono disponibili collegamenti diretti al supporto Azure.

## Descrivere Monitoraggio di Azure (Azure Monitor)

Azure Monitor è una piattaforma per raccogliere dati sulle risorse, analizzare tali dati, visualizzare le informazioni e persino agire sui risultati. Monitoraggio di Azure può monitorare le risorse di Azure, le risorse locali e persino le risorse multi-cloud, come le macchine virtuali ospitate con un provider di servizi cloud diverso.

Il diagramma seguente illustra quanto sia completo Monitoraggio di Azure:

![[image-15.png|497x333]]

A sinistra è riportato un elenco delle origini dei dati di registrazione e metrica che possono essere raccolti a ogni livello dell'architettura dell'applicazione, dall'applicazione al sistema operativo e alla rete.

Nel centro i dati di registrazione e metrica vengono archiviati nei repository centrali.

A destra, i dati vengono usati in diversi modi. È possibile visualizzare le prestazioni in tempo reale e cronologico in ogni livello dell'architettura o informazioni aggregate e dettagliate. I dati vengono visualizzati a livelli diversi per gruppi di destinatari diversi. È possibile visualizzare report di alto livello nel dashboard di Monitoraggio di Azure o creare visualizzazioni personalizzate usando power BI e query Kusto.

Inoltre, è possibile usare i dati per reagire agli eventi critici in tempo reale, tramite avvisi recapitati ai team tramite SMS, posta elettronica e così via. In alternativa, è possibile usare le soglie per attivare la funzionalità di scalabilità automatica per soddisfare la domanda.

## Analisi dei log di Azure

**Log Analytics** è lo strumento di Azure Monitor per scrivere ed eseguire query sui dati raccolti. Supporta query semplici (per ordinare, filtrare e analizzare record) e query avanzate (per analisi statistiche e visualizzazioni grafiche). È utile sia per analisi interattive sia in combinazione con altre funzionalità, come avvisi basati su query o cartelle di lavoro.

## Avvisi di Monitoraggio di Azure

Gli **avvisi di Monitoraggio di Azure** notificano automaticamente quando viene superata una soglia definita. È possibile:

- Configurarli su **metriche** (es. CPU > 80%) con notifiche quasi in tempo reale.
- Basarli su **log**, con logiche complesse che combinano più origini dati.

Gli avvisi possono solo notificare o tentare azioni correttive. Le notifiche e le azioni sono gestite tramite **gruppi di azioni**, collezioni di preferenze condivise da Monitoraggio di Azure, Integrità dei servizi e Azure Advisor.

![[image-16.png|526x233]]

## Application Insights

**Application Insights**, parte di Azure Monitor, analizza le prestazioni delle applicazioni web, indipendentemente dal fatto che siano ospitate su Azure, on-premises o altri cloud.

Si può configurare tramite:
- **SDK** integrato nell’applicazione.
- **Agente di Application Insights** (supporta C#.NET, VB.NET, Java, JavaScript, Node.js, Python).

Monitora:

- Frequenza richieste, tempi di risposta, errori.
- Prestazioni e affidabilità delle dipendenze esterne.
- Visualizzazioni pagina e tempi di caricamento lato browser.
- Chiamate AJAX (tassi, latenze, errori).
- Utenti, sessioni e contatori di prestazioni (CPU, memoria, rete).

In più, può inviare **richieste sintetiche periodiche** per verificare lo stato dell’app anche in assenza di traffico reale.

