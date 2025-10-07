---
autore: Kevin Milli
---

## 1. Microsoft Azure: La Piattaforma

**Microsoft Azure** è una **piattaforma di cloud computing** che offre un set di servizi in continua espansione.
- **Scopo:** Consente di creare soluzioni e applicazioni per soddisfare gli obiettivi aziendali.

---

## 2. Concetti Fondamentali del Cloud Computing

Il cloud computing è la base di Azure.

### 2.1. Definizione e Servizi Chiave

Cos'è il Cloud Computing?
Cloud Computing=**Distribuzione di servizi informatici tramite Internet**.

**Servizi Inclusi:**

1. **Infrastruttura IT Comune:**
    - Macchine Virtuali (VM)
    - Archiviazione
    - Database
    - Rete
        
2. **Servizi Avanzati:**
    - Internet delle Cose (IoT)
    - Apprendimento Automatico (ML)
    - Intelligenza Artificiale (IA)

**Vantaggi Operativi (Scalabilità):**

- **Scalabilità Verticale:** Possibilità di **aumentare/diminuire la potenza computazionale** (CPU, RAM).
- **Scalabilità Orizzontale:** Possibilità di **aumentare/diminuire lo spazio di archiviazione**.

---

### 2.2. Modello Economico: Pagamento in Base al Consumo (OpEx)

Il cloud computing opera su un modello **con pagamento in base al consumo**

|Tipo di Spesa|Definizione|Esempio nel Cloud|
|---|---|---|
|**Spesa in Conto Capitale (CapEx)**|Spesa **una tantum anticipata** per l'acquisto di risorse tangibili (es. server, data center).|**Non applicabile** al cloud computing.|
|**Spesa Operativa (OpEx)**|Denaro speso per servizi o prodotti **nel tempo** (es. affitto, leasing).|**Costi del Cloud:** Si pagano solo le risorse IT utilizzate (es. VM, storage).|

- **Vantaggio Chiave:** Non si pagano l'infrastruttura fisica, l'elettricità, la sicurezza, ecc., associati alla gestione di un data center locale. Si paga solo ciò che si usa.

---

### 2.3. Modello di Responsabilità Condivisa (Shared Responsibility Model)

Questo modello definisce chiaramente **chi è responsabile di cosa** tra il **Provider di Servizi Cloud (CSP)** come Azure e l'**Utente/Consumer**.

| Responsabilità           | Data Center Tradizionale                                                                      | Cloud Computing                  |
| ------------------------ | --------------------------------------------------------------------------------------------- | -------------------------------- |
| **Piena Responsabilità** | **Utente** (responsabile di tutto)                                                            | **Condivisa** (tra CSP e Utente) |
| **A carico del CSP**     | Sicurezza fisica, alimentazione, raffreddamento, connettività di rete, manutenzione hardware. |                                  |
| **A carico dell'Utente** | **Dati e informazioni** archiviati nel cloud.                                                 |                                  |

#### Relazione con i Tipi di Servizio (IaaS, PaaS, SaaS)

Il grado di responsabilità dell'utente varia in base al modello di servizio cloud utilizzato:

|Modello di Servizio|Descrizione|Livello di Responsabilità dell'Utente|
|---|---|---|
|**IaaS** (Infrastructure as a Service)|Fornisce solo l'infrastruttura di base (VM, rete).|**Massima Responsabilità** (Gestione SO, applicazioni, dati).|
|**PaaS** (Platform as a Service)|Fornisce piattaforma e strumenti (middleware, SO).|**Responsabilità Media** (Gestione dati e applicazioni).|
|**SaaS** (Software as a Service)|Fornisce software pronto all'uso (es. Office 365).|**Minima Responsabilità** (Gestione solo di dati e accesso).|

---

### 2.4. Modelli di Distribuzione Cloud (Cloud Models)

I modelli cloud definiscono il tipo di implementazione delle risorse. I tre principali sono: **Pubblico, Privato e Ibrido**.

#### Cloud Pubblico

- **Gestione:** Creato, controllato e gestito da un **Provider di Servizi Cloud di terze parti** (es. Azure, AWS).
- **Accesso:** Accessibile e utilizzabile da **chiunque** voglia acquistare i servizi.
- **Vantaggi:**
    - Nessuna spesa in conto capitale (CapEx).
    - Provisioning rapido di applicazioni.
    - Si paga solo l'uso.
- **Svantaggi:** Controllo limitato su risorse e sicurezza.
 
#### Cloud Privato

- **Gestione:** Utilizzato da una **singola entità** (evoluzione del data center aziendale). Offre servizi IT su Internet (o rete privata).
- **Accesso:** Uso esclusivo dell'entità proprietaria.
- **Vantaggi:**
    - **Controllo completo** su risorse e sicurezza.
    - I dati non sono collocati con altri dati di organizzazioni esterne.
- **Svantaggi:**
    - Costo maggiore e meno vantaggi di scalabilità del cloud pubblico.
    - Necessario acquistare hardware e gestire manutenzione e aggiornamenti.

#### Cloud Ibrido

- **Definizione:** Ambiente di calcolo che utilizza **cloud pubblici e privati in un ambiente interconnesso**.
- **Scenari d'Uso:**
    - **Flessibilità/Overflow:** Consente al cloud privato di far fronte a un aumento temporaneo della domanda (burst) distribuendo risorse pubbliche.
    - **Sicurezza/Conformità:** Permette di scegliere quali servizi mantenere nel cloud pubblico e quali nell'infrastruttura privata per requisiti di sicurezza/legali.
- **Vantaggi:**
    - **Massima flessibilità.**
    - Le organizzazioni decidono dove eseguire le proprie applicazioni.
    - Controllo sui requisiti di sicurezza, conformità o legali.

|Cloud Pubblico|Cloud Privato|Cloud Ibrido|
|---|---|---|
|Nessuna spesa in CapEx.|Controllo completo su risorse e sicurezza.|Offre la massima flessibilità.|
|Provisioning/deprovisioning rapido.|Dati isolati da altre organizzazioni.|Si decide dove eseguire le applicazioni.|
|Si paga solo ciò che si usa.|Necessario acquistare e mantenere l'hardware.|Si controllano i requisiti di sicurezza/legali.|
|Controllo limitato.|Manutenzione e aggiornamenti a carico dell'organizzazione.|Ambiente interconnesso Pubblico + Privato.|

---

### 2.5. Scenari Avanzati e Strumenti Azure

#### Multi-cloud

- **Definizione:** Utilizzo di **due (o più) provider di servizi cloud pubblici** contemporaneamente (es. Azure + AWS).
- **Motivazione:** Utilizzare funzionalità diverse di diversi provider, o in fase di migrazione tra provider.

#### Azure Arc

- **Guida Operativa:** **Semplifica la gestione** dell'ambiente cloud, indipendentemente dalla distribuzione.
- **Ambienti Supportati:** Cloud Pubblico (solo Azure), Cloud Privato (data center), Ibrido o **Multi-cloud**.

#### Soluzione Azure VMware (AVS)

- **Guida Operativa:** Consente di **eseguire carichi di lavoro VMware in Azure**.
- **Scenario:** Ideale per chi usa già VMware in un ambiente privato e vuole migrare a un cloud pubblico o ibrido con integrazione e scalabilità semplificate.

---

## Obiettivi di Apprendimento - Riepilogo

I concetti fondamentali che hai coperto sono:

1. **Definire il cloud computing.**
2. **Descrivere il modello di responsabilità condivisa.**
3. **Definire i modelli cloud:** pubblico, privato e ibrido.
4. **Identificare i casi d'uso appropriati** per ogni modello cloud (es. ibrido per il burst).
5. **Descrivere il modello di pagamento in base al consumo** (OpEx).
6. **Confrontare i modelli dei prezzi del cloud** (CapEx vs. OpEx).