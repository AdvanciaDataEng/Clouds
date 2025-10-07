
Microsoft Fabric è una piattaforma di analisi e dati end-to-end che offre una **soluzione unificata** per l'intero ciclo di vita dei dati (archiviazione, spostamento, trasformazione, elaborazione e reportistica). Data la crescente sensibilità dei dati e l'elevato volume gestito, la **Governance dei Dati** (insieme di tecniche e strumenti per garantire accuratezza, affidabilità e sicurezza) è fondamentale.

Microsoft Purview è la soluzione completa di Microsoft per la Governance dei Dati, progettata per scoprire, catalogare e gestire gli asset di dati nell'intera organizzazione, integrandosi perfettamente con Fabric.

---

## 1. Governance dei Dati in Microsoft Fabric (Funzionalità Integrate)

Microsoft Fabric include funzionalità di governance e conformità predefinite che possono soddisfare le esigenze di base, spesso sufficienti con la sola licenza Fabric.

### Concetti Chiave di Fabric

- **Microsoft Fabric:** Piattaforma unificata che gestisce l'intero ciclo di vita dei dati, indipendentemente da dimensioni o formato, permettendo analisi avanzate e Business Intelligence (BI).
- **OneLake:** L'implementazione di Fabric del Data Lake, basata su **Azure Data Lake Storage Gen2 (ADLS Gen2)**. Archivia **una singola copia** di dati strutturati e non strutturati, garantendo l'applicazione universale di criteri e controlli di sicurezza.

### Pilastri della Governance dei Dati

La governance dei dati è la pratica di gestione dei dati per garantirne **qualità, coerenza, sicurezza e usabilità** attraverso un framework di ruoli, responsabilità, processi, politiche e standard.

|Elemento|Descrizione|
|---|---|
|**Controllo**|Registrazione dell'origine e delle modifiche ai dati (**Derivazione dei Dati**).|
|**Valutazione**|Verifica dell'utilità e accuratezza dei dati.|
|**Documentazione**|Descrizioni chiare per supportare l'uso informato (**Catalogo**).|
|**Gestione**|Correzione dei dati, gestione delle richieste di accesso, conformità normativa.|
|**Protezione**|Sicurezza contro accessi non autorizzati e attacchi dannosi.|

**Intuizione:** Una governance dei dati efficace porta a una **Singola Fonte di Verità**, migliorando la qualità dei dati, riducendo i costi e facilitando la conformità (es. **HIPAA** in sanità).

### Funzionalità Integrate di Governance di Fabric

|Area di Governance|Funzionalità di Fabric|Dettagli|
|---|---|---|
|**Gestione Asset**|**Portale di Amministrazione**|Controlli a livello di tenant, capacità, domini e aree di lavoro (riservato agli amministratori).|
||**Tenant, Domini, Aree di Lavoro**|Contenitori logici per il controllo di accesso e funzionalità. **Domini** raggruppano dati per aree aziendali; **Aree di Lavoro** per team specifici.|
||**Analisi dei Metadati**|Estrazione di valori (nomi, sensibilità, approvazioni) per l'analisi e l'impostazione di criteri.|
|**Sicurezza e Protezione**|**Tag Dati**|Identificazione della riservatezza dei dati e applicazione di criteri di conservazione/protezione.|
||**Ruoli dell'Area di Lavoro**|Definizione degli utenti autorizzati ad accedere ai dati in un'area di lavoro.|
||**Controlli a Livello di Dati**|Restrizioni granulari a livello di elementi (tabelle, righe, colonne) di Fabric.|
||**Certificazioni**|Conformità a standard come **HIPAA BAA**, ISO/IEC 27001, ecc.|
|**Individuazione e Uso**|**Hub Dati OneLake**|Strumento che facilita l'individuazione e l'esplorazione dei dati nell'ambiente Fabric.|
||**Approvazione (Certificazione/Promozione)**|L'utente approva un elemento per identificarlo come di **alta qualità** o **promosso**, aumentandone l'affidabilità.|
||**Derivazione dei Dati**|Visualizzazione del flusso di dati tra gli elementi, essenziale per l'analisi dell'impatto delle modifiche.|
|**Monitoraggio**|**Hub di Monitoraggio**|Visualizzazione centralizzata delle attività di Fabric per gli elementi a cui l'utente è autorizzato.|
||**App Metriche di Capacità**|Dettagli sull'uso di Fabric e sul consumo di risorse per una gestione efficiente della capacità.|

---

## 2. Perché Utilizzare Microsoft Purview con Microsoft Fabric

Quando le esigenze di governance e conformità superano le funzionalità integrate di Fabric (es. requisiti normativi più rigorosi come la **HIPAA** per i dati sanitari), l'integrazione con Microsoft Purview è essenziale.

### Che cos'è Microsoft Purview?

Microsoft Purview è una suite di soluzioni unificate per **gestire, proteggere e governare i dati in ambienti ibridi e multicloud**. Integra i pilastri di:

1. **Governance dei Dati** (Discovery, Catalogazione, Classificazione).
2. **Sicurezza dei Dati** (Information Protection, DLP).
3. **Rischio e Conformità** (Audit, Compliance Manager).

Fornisce un **repository centralizzato di metadati** e automatizza l'identificazione delle informazioni sensibili.

### Strumenti Aggiuntivi di Microsoft Purview per Fabric

Registrando il tenant di Fabric in Purview, si sbloccano funzionalità avanzate che migliorano la conformità e la protezione.

|Strumento Purview|Descrizione e Vantaggi per Fabric|Esempio Pratico (Sanità - HIPAA)|
|---|---|---|
|**Mappa dei Dati**|Analizza gli asset di dati per acquisire metadati e identificare dati sensibili in ambienti ibridi/multicloud. Mantiene il catalogo aggiornato con analisi e classificazione automatizzate.|Identifica automaticamente dove sono archiviati i Dati Sanitari Protetti (PHI) nei diversi lakehouse e data warehouse di Fabric.|
|**Catalogo Unificato**|Catalogo ricercabile e centralizzato per la gestione, l'accesso e il miglioramento dell'integrità dei dati. Permette la ricerca per parole chiave e la visualizzazione di metadati, derivazione, classificazione ed etichette.|Permette agli utenti di cercare "record paziente" e visualizzare immediatamente le etichette di riservatezza, la derivazione e richiedere l'accesso self-service gestito.|
|**Protezione delle Informazioni (MIP)**|Classifica, etichetta e protegge i dati sensibili in tutta l'organizzazione con **Etichette di Riservatezza** personalizzabili. Le etichette applicano controlli di accesso e crittografia che **seguono i dati** ovunque.|**Crittografa** automaticamente gli elementi di Fabric etichettati come "Estremamente Riservato - PHI" e impedisce la copia a utenti non autorizzati.|
|**Prevenzione della Perdita dei Dati (DLP)**|Criteri per rilevare, monitorare e controllare automaticamente la condivisione o lo spostamento non autorizzato di dati sensibili.|Impedisce che un utente tenti di condividere un report di Fabric contenente dati PHI con un destinatario esterno all'organizzazione.|
|**Audit**|Soluzioni di controllo che registrano automaticamente tutte le attività di utenti e amministratori (es. accesso a elementi di Fabric, modifiche ai criteri).|Traccia la modifica di un criterio DLP da parte di un amministratore e registra gli accessi non autorizzati a un record paziente per scopi forensi e di conformità.|

---

## 3. Gestire i Dati tramite l'Hub di Microsoft Purview in Fabric

L'Hub di Microsoft Purview in Fabric è il punto di accesso per visualizzare report e informazioni dettagliate (insights) sugli elementi di Fabric, agendo da gateway verso le funzionalità avanzate di Purview.

### Connessione Purview a Fabric

Per stabilire la connessione è necessario seguire passaggi specifici:

1. **Registrazione Origine Dati:** Nel portale di Microsoft Purview, registrare il tenant di Fabric come nuova origine dati (tramite l'icona **Mappa dati**).
2. **Autenticazione:** Configurare l'autenticazione in Microsoft Entra ID (precedentemente Azure AD), tipicamente usando l'**Identità Gestita** (Managed Identity) dell'account Purview.
3. **Creazione Analisi:** Creare un'analisi (scan) in Purview che utilizzi l'origine dati Fabric. Questo consente alla **Mappa dei Dati** di Purview di estrarre i metadati.

### Funzionalità dell'Hub Purview in Fabric

Dopo la connessione, l'Hub Purview diventa disponibile in Fabric tramite le **Impostazioni Infrastruttura**.

- **Gateway Centralizzato:** Offre una posizione centralizzata per avviare la governance dei dati e accedere alle funzionalità più avanzate nel portale di Microsoft Purview.
- **Report e Dashboard:** I riquadri e i report forniscono **dashboard pratici** e informazioni dettagliate che riflettono la posizione di sicurezza e governance del patrimonio di dati del tenant di Fabric (es. panoramica sui dati sensibili, verifica dell'autenticità degli elementi, domini).
- **Insight Pratici:** Le visualizzazioni guidano gli utenti verso azioni concrete per migliorare la postura di governance e sicurezza.

