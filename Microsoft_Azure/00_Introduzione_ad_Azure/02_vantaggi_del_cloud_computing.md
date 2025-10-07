
## Obiettivi Operativi e Concettuali

Al termine, sarai in grado di sfruttare i vantaggi del cloud computing di Azure, in particolare:

1. Garantire la **Disponibilità Elevata** (uptime) e la **Scalabilità** (gestione della domanda).
2. Mantenere l'**Affidabilità** (resilienza) e la **Prevedibilità** (prestazioni e costi).
3. Implementare **Sicurezza** e **Governance** adeguate al modello di servizio.
4. Sfruttare le opzioni di **Gestibilità** (automazione e strumenti).

## 1. Disponibilità Elevata (Uptime) & Scalabilità (Domanda)

Questi sono i pilastri per un servizio cloud performante e reattivo.

### Disponibilità Elevata (High Availability - HA)

- **Concetto:** Assicurare che le risorse e i servizi siano sempre **accessibili e operativi**.
- **Misura:** Espressa tramite **SLA (Service Level Agreement)**, che definisce la **percentuale di disponibilità** (es. 99,9% è molto diverso da 90%).
- **Vantaggio Operativo:** **Massimizzazione del tempo di attività** e **continuità del servizio** per gli utenti finali, in linea con l'SLA contrattuale.

### Scalabilità (Scalability)

- **Concetto:** La capacità di **ridimensionare le risorse** per **soddisfare la domanda** (gestione dei picchi di traffico).
- **Vantaggio Economico (Modello a Consumo):** Si paga **solo per ciò che si usa**. Quando la domanda scende, si riducono le risorse e i **costi** diminuiscono automaticamente.

|Tipo di Scalabilità|Meccanismo|Descrizione Operativa|
|---|---|---|
|**Verticale (Scale Up/Down)**|**Aumentare/Ridurre la funzionalità** della risorsa esistente.|Aumentare la RAM/CPU di una macchina virtuale (più potente).|
|**Orizzontale (Scale Out/In)**|**Aggiungere/Sottrarre il numero di risorse** (istanze).|Aggiungere un'altra macchina virtuale identica al _load balancer_ (più unità).|

## 2. Affidabilità (Resilienza) & Prevedibilità (Prestazioni/Costi)

Garantiscono che il sistema funzioni come previsto e che le spese siano sotto controllo.

### Affidabilità (Reliability)

- **Concetto:** La capacità di un sistema di **recuperare da errori (Fault Tolerance)** e **continuare a funzionare (Resilienza)**.
- **Principio Azure:** È uno dei pilastri fondamentali del **Microsoft Azure Well-Architected Framework**.
- **Vantaggio Operativo:** La **progettazione decentralizzata** di Azure (distribuzione in aree globali) supporta naturalmente un'infrastruttura **affidabile e resiliente**.

### Prevedibilità (Predictability)

La prevedibilità infonde fiducia, sia nelle prestazioni che nella gestione delle spese.

|Tipo di Prevedibilità|Focus Operativo|Strumenti/Concetti Chiave|
|---|---|---|
|**Prestazioni**|**Stimare le risorse** necessarie per una _user experience_ positiva.|**Scalabilità automatica** (ridimensionamento automatico), **Bilanciamento del carico**, **Disponibilità Elevata**.|
|**Costo**|**Stimare/Prevedere la spesa cloud** (Budgeting).|**Monitoraggio in tempo reale** dell'uso, **Analisi dei dati** per modelli di spesa, **Calcolatore TCO** (Costo Totale di Proprietà) e **Calcolatore Prezzi** di Azure.|

## 3. Sicurezza & Governance

Il cloud supporta gli standard di conformità e la protezione della rete.

### Sicurezza

- **Protezione DDoS:** I Cloud Provider gestiscono in modo efficace situazioni come gli **attacchi DDoS** (Distributed Denial of Service) grazie alla loro infrastruttura estesa, rendendo la rete più sicura.
- **Controllo vs. Servizio:** La scelta del modello di servizio influisce sul livello di controllo sulla sicurezza:
    - **IaaS (Infrastructure as a Service):** **Massimo controllo**. Gestisci SO, patch e software installato.
    - **PaaS/SaaS (Platform/Software as a Service):** Le **patch e la manutenzione** di base sono **gestite automaticamente** dal provider, riducendo il carico operativo.

### Governance

- **Concetto:** Funzionalità cloud che supportano il **controllo, la conformità e la gestione delle policy** aziendali sulle risorse.

## 4. Gestibilità (Management)

Il cloud offre opzioni per automatizzare le operazioni e strumenti per la gestione delle risorse.

### Gestione _del_ Cloud (Automazione del Fornitore)

Queste sono funzionalità intrinseche di Azure che **automatizzano le operazioni di routine** sulla tua infrastruttura:

- **Scalabilità Automatica:** Ridimensionamento delle risorse _on-demand_.
- **Deployment:** Distribuzione delle risorse basata su **modelli preconfigurati** (es. ARM/Bicep), eliminando la configurazione manuale.
- **Monitoraggio Integrità:** Monitoraggio automatico dell'integrità e sostituzione automatica delle risorse con errori.
- **Alerting:** Ricezione di **avvisi automatici** basati su metriche configurate (prestazioni in tempo reale).

### Gestione _nel_ Cloud (Gestione dell'Ambiente)

Questi sono i **metodi e gli strumenti** che usi per interagire e amministrare l'ambiente Azure:

- **Portale Web:** L'interfaccia grafica (GUI) di Azure.
- **Interfaccia della Riga di Comando (CLI):** Strumento _cross-platform_ per script e automazione.
- **PowerShell:** _Shell_ e linguaggio di _scripting_ specifico per ambienti Windows e Azure.
- **API (Application Programming Interface):** Per l'integrazione programmatica con strumenti esterni.

