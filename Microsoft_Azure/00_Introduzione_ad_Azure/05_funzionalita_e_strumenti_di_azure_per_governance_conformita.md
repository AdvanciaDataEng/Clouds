---
autore: Kevin Milli
---

## Obiettivi di apprendimento

- Descrivere lo scopo di Microsoft Purview
- Descrivere l'utilizzo di Criteri di Azure
- Descrivere l'utilizzo dei blocchi delle risorse
- Descrivere lo scopo del portale Service Trust

## Descrivere lo scopo di Microsoft Purview

Microsoft Purview è una famiglia di soluzioni per la governance dei dati, il rischio e la conformità che consente di ottenere una visualizzazione singola e unificata dei dati. 
Microsoft Purview riunisce le informazioni sui dati locali, multicloud e software come un servizio.
Con Microsoft Purview è possibile rimanere aggiornati sul panorama dei dati grazie a:

- Individuazione automatica dei dati
- Classificazione dei dati sensibili
- Derivazione dei dati end-to-end

Due aree principali della soluzione includono Microsoft Purview: **rischio e conformità** e **conformità e governance unificata dei dati**.

### Soluzioni di rischio e conformità di Microsoft Purview  

Microsoft 365 è un componente fondamentale delle soluzioni di rischio e conformità di Microsoft Purview. Microsoft Teams, OneDrive ed Exchange sono solo alcuni dei servizi Microsoft 365 che Microsoft Purview usa per aiutare nella gestione e monitoraggio dei dati. Microsoft Purview, gestendo e monitorando i dati, è in grado di aiutare l'organizzazione:

- Proteggere i dati sensibili su cloud, app e dispositivi.
- Identificare i rischi dei dati e gestire i requisiti di conformità alle normative.
- Introduzione alla conformità normativa.

### Governance dei dati unificata  

Microsoft Purview offre soluzioni di governance dei dati solide e unificate che aiutano a gestire i dati locali, multicloud e software come un servizio. 
Le solide funzionalità di governance dei dati di Microsoft Purview consentono di gestire i dati archiviati in Azure, nei database SQL e Hive, in locale e persino in altri cloud come Amazon S3.
La governance unificata dei dati di Microsoft Purview aiuta l’organizzazione:

- Creare una mappa aggiornata dell'intero patrimonio di dati, che includa la classificazione dei dati e la derivazione end-to-end.
- Identificare la posizione in cui sono conservati i dati sensibili nel patrimonio.
- Creare un ambiente sicuro per i consumatori di dati per trovare dati preziosi.
- Generare approfondimenti sulle modalità di archiviazione e utilizzo dei dati.
- Gestire l'accesso ai dati del patrimonio in modo sicuro e su larga scala.

## Descrivere l'utilizzo di Criteri di Azure

Criteri di Azure è un servizio di Azure che consente di creare, assegnare e gestire criteri per il controllo e la verifica delle risorse. Questi criteri applicano regole diverse a tutte le configurazioni delle risorse in modo da mantenerne la conformità agli standard aziendali.

### In che modo vengono definiti i criteri in Criteri di Azure?

Criteri di Azure consente di definire singoli criteri e gruppi di criteri correlati, noti come iniziative. 
Criteri di Azure valuta le risorse ed evidenzia quelle non conformi ai criteri creati. Criteri di Azure può anche impedire la creazione di risorse non conformi.

I criteri di Azure possono essere impostati a ogni livello, ovvero per una risorsa specifica, per un gruppo di risorse, per una sottoscrizione e così via. Inoltre, i criteri di Azure vengono ereditati, quindi i criteri impostati a un livello superiore vengono automaticamente applicati a tutti i raggruppamenti che rientrano nell'elemento padre. Ad esempio, se si imposta un criterio di Azure per un gruppo di risorse, tutte le risorse create al suo interno riceveranno automaticamente lo stesso criterio.

Criteri di Azure include definizioni di criteri e iniziative predefinite per archiviazione, rete, calcolo, Centro sicurezza e monitoraggio. Se ad esempio si definisce un criterio che consente l'uso nell'ambiente solo di determinate dimensioni di macchine virtuali (VM), tale criterio viene richiamato quando si crea una nuova VM e ogni volta che si ridimensionano quelle esistenti. Il servizio Criteri di Azure, inoltre, valuta e monitora tutte le VM correnti nell'ambiente, incluse quelle create prima della creazione dei criteri.

In alcuni casi, Criteri di Azure può correggere automaticamente risorse e configurazioni non conformi in modo da garantire l'integrità dello stato delle risorse. Se ad esempio è necessario aggiungere a tutte le risorse di un determinato gruppo di risorse il tag AppName e il valore "SpecialOrders", Criteri di Azure applicherà automaticamente il tag se è mancante. Tuttavia, si mantiene comunque il controllo completo dell'ambiente. Se non si vuole che Criteri di Azure corregga automaticamente una specifica risorsa, è possibile contrassegnarla come eccezione e non verrà inclusa nel criterio.

Criteri di Azure si integra anche con Azure DevOps tramite l'applicazione di tutti criteri di integrazione continua e di pipeline di distribuzione relativi alla pre-distribuzione e alla post-distribuzione delle applicazioni.

### Che cosa sono le iniziative di Criteri di Azure?

Un'iniziativa di Criteri di Azure è un modo di raggruppare criteri correlati. La definizione dell'iniziativa contiene tutte le definizioni di criteri per tenere traccia dello stato di conformità per un obiettivo più ampio.
Ad esempio, Criteri di Azure include un'iniziativa denominata Abilita il monitoraggio nel Centro sicurezza di Azure. L'obiettivo è monitorare tutte le raccomandazioni per la sicurezza disponibili per tutti i tipi di risorsa di Azure nel Centro sicurezza di Azure.
In base a questa iniziativa sono incluse le definizioni di criteri seguenti:

- **Monitorare il database SQL non crittografato nel Centro sicurezza** Questo criterio monitora i database e i server SQL non crittografati.
- **Monitorare le vulnerabilità del sistema operativo nel Centro sicurezza** Questo criterio monitora i server che non soddisfano la baseline di vulnerabilità del sistema operativo configurata.
- **Monitorare la mancanza di Endpoint Protection nel Centro sicurezza** Questo criterio monitora i server che non dispongono di un agente di Endpoint Protection installato.

Infatti, l'iniziativa Abilita il monitoraggio nel Centro sicurezza di Azure contiene oltre 100 definizioni di criteri separate.

## Descrivere l'utilizzo dei blocchi delle risorse

Un blocco delle risorse impedisce l'eliminazione o la modifica accidentale delle risorse.

Anche se si predispongono criteri di Controllo degli accessi in base al ruolo di Azure, esiste comunque il rischio che gli utenti con il livello di accesso corretto possano eliminare risorse cloud critiche. A seconda del tipo, i blocchi delle risorse impediscono l'eliminazione o l'aggiornamento delle risorse. I blocchi delle risorse possono essere applicati a singole risorse, a gruppi di risorse o anche a un'intera sottoscrizione. I blocchi delle risorse vengono ereditati, ovvero un blocco impostato per un gruppo di risorse verrà applicato anche a tutte le risorse al suo interno.

### Tipi di blocchi delle risorse

Esistono due tipi di blocchi delle risorse, uno che impedisce agli utenti di eliminare una risorsa e l'altro che impedisce di cambiarla o eliminarla.

- Delete significa che gli utenti autorizzati possono ancora leggere e modificare una risorsa, ma non possono eliminarla.
- ReadOnly significa che gli utenti autorizzati possono leggere una risorsa, ma non eliminarla o aggiornarla. L'applicazione di questo blocco è simile alla concessione a tutti gli utenti autorizzati solo le autorizzazioni concesse dal ruolo Lettore.

### Come si gestiscono i blocchi delle risorse?

Per gestire i blocchi delle risorse, è possibile usare il portale di Azure, PowerShell, l'interfaccia della riga di comando di Azure o un modello di Azure Resource Manager.
Per visualizzare, aggiungere o eliminare blocchi nel portale di Azure, passare alla sezione Blocchi del riquadro Impostazioni di qualsiasi risorsa nel portale di Azure.

![[image-4.png|500x227]]

### Come si elimina o modifica una risorsa bloccata?

Anche se il blocco impedisce le modifiche accidentali, è comunque possibile apportare modifiche attraverso un processo in due passaggi.

Per modificare una risorsa bloccata, è necessario prima di tutto rimuovere il blocco. Dopo aver rimosso il blocco, è possibile applicare qualsiasi azione per l'esecuzione della quale si hanno le autorizzazioni necessarie. I blocchi risorse si applicano indipendentemente dalle autorizzazioni di controllo degli accessi in base al ruolo. Anche se si è proprietari della risorsa, è comunque necessario rimuovere il blocco prima di poter eseguire l'attività bloccata.

### Configurare un blocco delle risorse

Per applicare un blocco delle risorse, è necessario creare una risorsa in Azure. La prima attività è incentrata sulla creazione di una risorsa che sarà poi possibile bloccare nelle attività successive.

1. Accedere al portale di Azure all'indirizzo [https://portal.azure.com](https://portal.azure.com/)
2. Selezionare Crea una risorsa.
3. In Categorie selezionare Archiviazione.
4. In Account di archiviazione, selezionare Crea.
5. Nella scheda Informazioni di base del pannello Crea account di archiviazione immettere le informazioni seguenti. Lasciare i valori predefiniti per tutto il resto.

|**Impostazione**|**valore**|
|---|---|
|Gruppo di risorse|Crea nuovo|
|Nome account di archiviazione|Immettere un nome di account di archiviazione univoco|
|Ufficio|impostazione predefinita|
|Prestazioni|Normale|
|Ridondanza|Archiviazione con ridondanza locale|
6. Selezionare Rivedi e crea per rivedere le impostazioni dell'account di archiviazione e consentire ad Azure di convalidare la configurazione.
7. Dopo la convalida, selezionare Crea. Attendere la notifica della creazione corretta dell'account.
8. Selezionare Vai alla risorsa.

### Applicare un blocco delle risorse di sola lettura

In questa attività si applica un blocco delle risorse di sola lettura all'account di archiviazione. Qual è l'impatto sull'account di archiviazione?

1. Scorrere verso il basso fino alla sezione Impostazioni del pannello a sinistra della schermata.
2. Selezionare Blocchi.
3. Seleziona + Aggiungi.

![[image-5.png|576x391]]

4. Immettere un Nome del blocco.
5. Verificare che Tipo di blocco sia impostato su Sola lettura.
6. Seleziona OK.

### Aggiungere un contenitore all'account di archiviazione

In questa attività si aggiunge un contenitore all'account di archiviazione. Questo contenitore consente di archiviare i BLOB.

1. Scorrere verso l'alto fino alla sezione Archiviazione dati del pannello a sinistra della schermata.
2. Selezionare Contenitori.
3. Selezionare + Contenitore.

![[image-6.png|450x308]]

4. Immettere un nome per il contenitore e selezionare Crea.
5. Verrà visualizzato il messaggio di errore Non è stato possibile creare il contenitore di archiviazione.

![[image-7.png|355x156]]

>[!Nota]
>Il messaggio di errore indica che non è stato possibile creare un contenitore di archiviazione perché è presente un blocco.
>Il blocco di sola lettura impedisce operazioni di creazione o aggiornamento nell'account di archiviazione, pertanto non è
>possibile creare un contenitore di archiviazione.


### Modificare il blocco delle risorse e creare un contenitore di archiviazione

1. Scorrere verso il basso fino alla sezione Impostazioni del pannello a sinistra della schermata.
2. Selezionare Blocchi.
3. Selezionare il blocco delle risorse di sola lettura che è stato creato.
4. Modificare Tipo di blocco impostandolo su Elimina e selezionare OK.

![[image-8.png|401x380]]

5. Scorrere verso l'alto fino alla sezione Archiviazione dati del pannello a sinistra della schermata.
6. Selezionare Contenitori.
7. Selezionare + Contenitore.
8. Immettere un nome per il contenitore e selezionare Crea.
9. Il contenitore di archiviazione verrà visualizzato nell'elenco di contenitori.

È ora possibile capire in che modo il blocco di sola lettura impedisce l'aggiunta di un contenitore all'account di archiviazione. Dopo aver modificato il tipo di blocco (si sarebbe anche potuto rimuovere), è stato possibile aggiungere un contenitore.

### Eliminare l'account di archiviazione

Quest'ultima attività verrà in realtà eseguita due volte. Tenere presente che esiste un blocco di eliminazione nell'account di archiviazione, quindi non sarà in effetti possibile eliminare ancora l'account di archiviazione.

1. Scorrere verso l'alto fino a Panoramica nella parte superiore del pannello a sinistra dello schermo.
2. Selezionare Panoramica.
3. Selezionare Elimina.
![[image-9.png|463x207]]

Verrà visualizzata una notifica che informa che non è possibile eliminare la risorsa perché esiste un blocco di eliminazione. Per eliminare l'account di archiviazione, è necessario rimuovere il blocco di eliminazione.

![[image-10.png|506x166]]

### Rimuovere il blocco di eliminazione ed eliminare l'account di archiviazione

1. Selezionare il nome dell'account di archiviazione nella barra di navigazione nella parte superiore della schermata.
2. Scorrere verso il basso fino alla sezione Impostazioni del pannello a sinistra della schermata.
3. Selezionare Blocchi.
4. Selezionare Elimina.
5. Selezionare Home page nella barra di navigazione nella parte superiore della schermata.
6. Selezionare Account di archiviazione.
7. Selezionare l'account di archiviazione usato per questo esercizio.
8. Selezionare Elimina.
9. Per impedire l'eliminazione accidentale, Azure richiede di immettere il nome dell'account di archiviazione da eliminare. Immettere il nome dell'account di archiviazione e selezionare Elimina.

![[image-11.png|328x420]]

10. Verrà visualizzato un messaggio a indicare che l'account di archiviazione è stato eliminato. Selezionando Home page > Account di archiviazione, si noterà che l'account di archiviazione creato per questo esercizio non esiste più.

Complimenti. Il processo di configurazione, aggiornamento e rimozione di un blocco delle risorse in una risorsa di Azure è stato completato.

>[!Nota]
>Assicurarsi di completare l'attività 6 rimuovendo l'account di archiviazione. L'utente è l'unico responsabile delle risorse
>nell'account Azure. Assicurarsi di pulire l'account dopo aver completato questo esercizio.

## Descrivere lo scopo del portale Service Trust

Microsoft Service Trust Portal è un portale che offre l'acceso a un'ampia gamma di contenuti, strumenti e altre risorse sulle procedure di sicurezza, privacy e conformità di Microsoft.

Service Trust Portal contiene i dettagli relativi all'implementazione di Microsoft dei controlli e dei processi che proteggono i servizi cloud e i dati dei clienti al loro interno. Per accedere ad alcune risorse di Service Trust Portal, è necessario eseguire l'accesso come utente autenticato con l'account dei servizi cloud Microsoft (account dell'organizzazione Microsoft Entra). Sarà necessario esaminare e accettare l'accordo di riservatezza Microsoft per i materiali di conformità.

### Accesso a Service Trust Portal

È possibile accedere a Service Trust Portal all'indirizzo [https://servicetrust.microsoft.com/](https://servicetrust.microsoft.com/).

![[image-12.png|504x290]]

Le funzionalità e il contenuto di Service Trust Portal sono accessibili dal menu principale. Le categorie del menu principale sono:

- **Service Trust Portal**: fornisce un collegamento ipertestuale di accesso rapido per tornare alla home page di Service Trust Portal.
- **Raccolta personale**: consente di salvare (o bloccare in alto) i documenti in modo che sia possibile accedervi rapidamente nella pagina. È anche possibile configurare la ricezione di notifiche quando i documenti di Raccolta personale vengono aggiornati.  
- L'opzione **Tutti i documenti** fornisce un singolo posto di destinazione per i documenti nel portale Service Trust. In **Tutti i documenti** è possibile aggiungere documenti per visualizzarli nella **Raccolta personale**.

