---
autore: Kevin Milli
---

## Introduzione e Principi di Architettura della Sicurezza

Questo modulo esplora i pilastri della sicurezza in Azure, con focus sui servizi directory, sui metodi di autenticazione e sul controllo degli accessi.

### 1.1. Zero Trust: La Filosofia Guida

Il modello **Zero Trust** (Fiducia Zero) è la strategia di sicurezza moderna adottata da Microsoft. Sostituisce la mentalità perimetrale ("fidarsi di ciò che è all'interno") con il principio: **==Mai fidarsi, verificare sempre==**.

> **Intuizione:** Non esistono reti "sicure" per impostazione predefinita. Ogni richiesta di accesso, indipendentemente dalla sua origine (interna o esterna), deve essere trattata come non attendibile.

I tre principi fondamentali del modello Zero Trust, applicati in Azure:

|Principio|Azione in Azure|
|---|---|
|**Verificare Esplicitamente**|Autenticare e autorizzare sempre l'accesso in base a tutti i punti dati disponibili: identità dell'utente, stato del dispositivo, posizione, dati sensibili.|
|**Usare l'Accesso con Privilegi Minimi (JIT/JEA)**|Limitare l'accesso ai soli diritti strettamente necessari per completare un'attività (Just-In-Time e Just-Enough-Access).|
|**Presumere una Violazione**|Ridurre al minimo il raggio d'azione di un attacco (micro-segmentazione) e usare l'analisi per migliorare le difese e la risposta automatizzata alle minacce.|

### 1.2. Difesa in Profondità (Defense in Depth)

La Difesa in Profondità è una strategia multi-livello che mira a ostacolare un potenziale aggressore a ogni passo. 
Se un livello di sicurezza viene superato, il livello successivo funge da ulteriore barriera.

In Azure, i livelli chiave di Difesa in Profondità includono:

1. **Perimetro (Identità):** **Microsoft Entra ID** come perimetro primario per l'autenticazione.
2. **Rete:** **Gruppi di Sicurezza di Rete (NSG)** e **Azure Firewall**.
3. **Calcolo (Compute):** Crittografia su VM, **Microsoft Defender for Cloud** per la protezione dei server.
4. **Dati:** Crittografia dei dati inattivi e in transito.

## Microsoft Entra ID (Ex Azure AD)

**Microsoft Entra ID** (Entra ID) è il servizio di gestione delle identità e degli accessi (IAM) basato sul cloud. 
È il servizio directory per Microsoft 365, Azure e migliaia di altre applicazioni SaaS.

### 2.1. Confronto: Entra ID vs. Active Directory (AD DS On-Prem)

| Caratteristica   | Active Directory Domain Services (AD DS)                   | Microsoft Entra ID                                                                   |
| ---------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **Architettura** | Locale (On-Premise), basato su Controller di Dominio (DC). | Basato su Cloud (Identity-as-a-Service - IDaaS), distribuito globalmente.            |
| **Protocolli**   | Kerberos, NTLM, LDAP (Legacy).                             | OAuth 2.0, OpenID Connect, SAML (Moderno).                                           |
| **Focus**        | Gestione di computer in dominio, Criteri di Gruppo (GPO).  | Gestione di utenti, gruppi, applicazioni e dispositivi mobili (MDM/MAM).             |
| **Relazione**    | Un utente può avere un solo account in una foresta.        | Gli utenti sono contenuti in un **Tenant**; un tenant può connettersi a più servizi. |

### 2.2. Chi Utilizza Entra ID?

- **Amministratori IT:** Controllano l'accesso alle risorse, applicano la Multi-Factor Authentication (MFA) e definiscono i criteri di accesso condizionale.
- **Sviluppatori di Applicazioni:** Implementano l'Single Sign-On (SSO) e l'autorizzazione nelle loro app (LOB e SaaS).
- **Utenti Finali:** Gestiscono la propria identità e la reimpostazione della password self-service (SSPR).
- **Abbonati ai Servizi Microsoft:** Utilizzato per l'autenticazione in servizi come Microsoft 365, Azure, Dynamics.

### 2.3. Funzionalità Centrali di Entra ID

| Servizio                        | Descrizione Dettagliata                                                                                                                                                                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Autenticazione**              | Verifica dell'identità. Include MFA, **Passwordless** (es. FIDO2, Microsoft Authenticator) e servizi di sicurezza come il **Blocco Intelligente** per prevenire attacchi a forza bruta.                                                           |
| **Single Sign-On (SSO)**        | Consente agli utenti di accedere a più applicazioni (sia cloud che locali) con un unico set di credenziali. Semplifica la gestione degli account utente e riduce i rischi di sicurezza.                                                           |
| **Gestione delle Applicazioni** | Gestione centralizzata dell'accesso ad app SaaS (es. Salesforce, Workday) e app Line-of-Business (LOB) locali tramite **Application Proxy**.                                                                                                      |
| **Gestione dei Dispositivi**    | Supporta la registrazione e l'adesione dei dispositivi (`Device Registration`, `Entra Join`) per l'integrazione con strumenti MDM come **Microsoft Intune**, permettendo un controllo di accesso basato sullo stato di integrità del dispositivo. |

### Insight: Accesso Condizionale (Conditional Access - CA)

Il CA è lo strumento primario per implementare la filosofia **Zero Trust** in Entra ID. 
È un motore di criteri che valuta i **segnali** in tempo reale e applica i **controlli** appropriati.

#### Elementi di un Criterio CA:

| Componente                | Descrizione                               | Esempio di Condizione                                                                                                                                  |
| ------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Utenti/Gruppi**         | Chi viene incluso o escluso dal criterio. | _Utenti inclusi:_ Tutti gli utenti; _Utenti esclusi:_ Account di emergenza (Break-Glass).                                                              |
| **Risorse Target**        | Le applicazioni protette dal criterio.    | Microsoft 365, Azure Management, App specifiche.                                                                                                       |
| **Condizioni (Segnali)**  | Il contesto in cui si verifica l'accesso. | **Rischio Utente/Accesso** (derivato da Microsoft Entra ID Protection), Piattaforma Dispositivo, Posizione di rete, App Client.                        |
| **Controlli**             | L'azione applicata (Grant o Block).       | **Concedi accesso** (richiedendo MFA, o richiedendo che il dispositivo sia segnato come conforme in Intune); **Blocca accesso**.                       |
| **Controlli di Sessione** | Azioni applicate _dopo_ l'accesso.        | Utilizzare il Controllo Applicazioni per l'Accesso Condizionale (CASB) per limitare le azioni all'interno dell'app (es. impedire il download di file). |

> **Strumenti Rilevanti:** L'accesso condizionale richiede una licenza **Microsoft Entra ID P1** o **P2**. La P2 aggiunge funzionalità avanzate come Entra ID Protection che fornisce i segnali di rischio utente e accesso.

## Integrazione Ibrida: Microsoft Entra Connect

Quando si utilizza un ambiente ibrido (AD DS locale + Entra ID), **Microsoft Entra Connect** è l'utility che sincronizza le identità tra i due ambienti.

- **Sincronizzazione Unidirezionale:** Generalmente, la sincronizzazione va dall'AD DS locale a Entra ID.
- **Metodi di Autenticazione Ibrida:**
    1. **Password Hash Synchronization (PHS):** L'hash dell'hash della password locale viene sincronizzato con Entra ID. È il metodo più semplice e resiliente.
    2. **Pass-Through Authentication (PTA):** Entra ID inoltra la richiesta di accesso all'AD DS locale per la convalida.
    3. **Federation (AD FS):** Utilizzo di Active Directory Federation Services (AD FS) per delegare l'autenticazione.

## Microsoft Entra Domain Services (DS)

**Microsoft Entra Domain Services (DS)** è un servizio che fornisce servizi di dominio _gestiti_ compatibili con Active Directory (AD DS) in Azure, senza la necessità di distribuire e gestire controller di dominio (DC) IaaS.

### 4.1. Funzionalità e Utilizzo

- **Servizi Offerti:** Aggiunta a un dominio, Criteri di Gruppo (un subset), LDAP (Lightweight Directory Access Protocol) e autenticazione Kerberos/NTLM.
- **Caso d'Uso Principale:** Ideale per il **Lift-and-Shift** di applicazioni legacy in Azure che richiedono ancora protocolli di autenticazione tradizionali (Kerberos/NTLM) o la funzionalità di Domain Join, ma che non supportano l'autenticazione moderna di Entra ID.

### 4.2. Come Funziona il Servizio Gestito

1. **Deployment:** Azure distribuisce e gestisce **due controller di dominio Windows Server** (noti come **Set di Repliche**) nell'area selezionata.
2. **Manutenzione:** Azure è responsabile di patch, aggiornamenti e backup dei DC. L'amministratore **non** ha accesso RDP a questi controller.
3. **Sincronizzazione:** Il dominio gestito è alimentato da una sincronizzazione **unidirezionale** da Entra ID $\rightarrow$ Entra Domain Services. Le modifiche apportate localmente in Entra DS **non** vengono sincronizzate con Entra ID.

### Insight: Entra DS (Managed) vs. AD DS in Azure IaaS (Self-Managed)

La scelta dipende dal livello di controllo richiesto:

|Caratteristica|Microsoft Entra Domain Services (DS)|Active Directory Domain Services (AD DS) in IaaS|
|---|---|---|
|**Gestione Infrastruttura**|**Gestita da Microsoft** (patching, HA, backup).|**Gestita dall'utente** (VM, OS, patch, HA).|
|**Obiettivo**|Abilitare applicazioni legacy con autenticazione Kerberos/NTLM nel cloud (Lift-and-Shift).|Estendere la foresta locale al cloud o creare una foresta completamente nuova e personalizzata.|
|**Schema e Trust**|**Schema non estendibile.** Non supporta trust di foresta uscenti.|**Schema completamente estendibile.** Supporta trust di foresta bidirezionali.|
|**Overhead Amministrativo**|Molto basso.|Molto alto (gestione completa dei DC).|
|**Utenti Aggiunti**|Utenti sincronizzati da Entra ID (o da AD DS locale tramite Entra Connect).|Utenti creati direttamente nei DC della VM.|

> **Conclusione:** Utilizza **Entra Domain Services** per la semplicità e il basso overhead se le tue applicazioni richiedono solo le funzionalità AD DS di base. 
> Utilizza **AD DS in IaaS** se hai bisogno del controllo completo, di estensioni dello schema o di trust di foresta.

---

# Descrivere i Metodi di Autenticazione e Accesso di Azure

L'**Autenticazione** è il processo fondamentale che verifica la **claim** di un'identità (utente, servizio o dispositivo). 
È il primo passo nella catena di sicurezza, seguito dall'**Autorizzazione**, che determina a quali risorse l'identità verificata può accedere.

## 1. L'Evoluzione dell'Autenticazione: Sicurezza vs. Usabilità

Storicamente, la sicurezza percepita (es. complessità delle password) è stata spesso in contrasto con la praticità d'uso (usabilità). **Microsoft Entra ID** mira a risolvere questo dilemma offrendo soluzioni che innalzano sia la sicurezza che l'usabilità.

|Metodo di Autenticazione|Livello di Sicurezza (Rischio di Compromissione)|Livello di Praticità (Usabilità)|
|---|---|---|
|**Password Semplici**|Basso|Alto|
|**Password Complesse + SSO**|Medio|Medio-Alto|
|**MFA**|Alto|Medio|
|**Passwordless**|**Molto Alto** (Resistenza al Phishing)|**Alto** (Azione semplice e rapida)|

## 2. Single Sign-On (SSO): Semplificazione dell'Identità Unica

L'**Single Sign-On (SSO)** è un meccanismo che permette a un utente di eseguire l'accesso una singola volta e di usare tale sessione autenticata per accedere a più applicazioni e servizi, indipendentemente dal provider.

### 2.1. Principi Tecnici e Vantaggi

- **Trust tra Applicazioni:** Per funzionare, l'SSO si basa sul fatto che i **Service Provider (SP)** (le applicazioni) si fidino del **Identity Provider (IdP)** (Entra ID). Questa fiducia è mediata da standard di federazione.
- **Standard Chiave:** Entra ID supporta i protocolli moderni di federazione, inclusi:
    - **SAML 2.0 (Security Assertion Markup Language):** Utilizzato principalmente per l'accesso a molte applicazioni SaaS di terze parti.
    - **OAuth 2.0 e OpenID Connect (OIDC):** Usati per l'accesso API, mobile e per le applicazioni native di Microsoft.
- **Riduzione del Rischio:** L'SSO riduce la superficie di attacco complessiva, poiché diminuisce il numero di password che gli utenti devono gestire (e potenzialmente riutilizzare o dimenticare). La gestione IT è semplificata, specialmente nell'offboarding.

> **Nota Critica (Zero Trust):** L'efficacia di tutto il sistema SSO dipende dalla sicurezza dell'autenticatore iniziale. Se la prima credenziale viene compromessa, l'accesso a tutte le applicazioni federate è a rischio. Per questo, l'SSO in un contesto Zero Trust **deve** essere abbinato all'MFA o al Passwordless.

## 3. Autenticazione a Più Fattori (MFA)

L'**Autenticazione a Più Fattori (MFA)** richiede all'utente di fornire due o più **fattori** di prova dell'identità. 
Questa è la difesa più efficace contro gli attacchi basati su credenziali rubate (Credential Theft).

### 3.1. I Tre Fattori di Autenticazione

L'MFA combina elementi appartenenti ad almeno due delle tre categorie seguenti per un'autenticazione completa:

1. **Conoscenza (Something you know):**
    - _Esempio:_ Password, PIN.
2. **Possesso (Something you have):**
    - _Esempio:_ Codice inviato al telefono, **Chiave di Sicurezza FIDO2**, Notifica da App Authenticator.
3. **Inerenza (Something you are):**
    - _Esempio:_ Dati Biometrici (Impronta digitale, Riconoscimento facciale).

### 3.2. Microsoft Entra MFA

**Microsoft Entra MFA** è il servizio che abilita l'MFA, spesso integrato tramite **Conditional Access** per applicare in modo selettivo le richieste di autenticazione aggiuntiva.

- **Metodi Supportati (Oltre la Password):**
    - Notifica App Microsoft Authenticator (il più sicuro dopo FIDO2 e il più consigliato).
    - Codice di verifica da App Authenticator (TOTP).
    - Telefonata o SMS (meno sicuri a causa di attacchi di scambio SIM).

## 4. L'Autenticazione Senza Password (Passwordless)

L'obiettivo dell'autenticazione senza password è eliminare il punto debole più grande della sicurezza: la password stessa. 
Rimuovendo le password, si mitigano drasticamente attacchi di phishing, brute force e riutilizzo di credenziali.

> **Intuizione:** Le passwordless combinano un fattore di **Possesso** (il dispositivo registrato) con un fattore di **Conoscenza** (PIN locale) o **Inerenza** (Biometria) per generare in modo crittografico la prova di identità.

Microsoft Azure supporta tre opzioni principali di autenticazione senza password:

### 4.1. Windows Hello for Business (WHfB)

- **Focus:** Ottimizzato per gli _Information Worker_ che utilizzano un PC Windows designato (computer come fattore di possesso).
- **Meccanismo:** Utilizza un PIN o la biometria associati direttamente al dispositivo (tramite un chip TPM) per accedere alle risorse. Le credenziali sono crittografate e non lasciano mai il dispositivo.
- **Beneficio:** Offre un'esperienza SSO fluida per risorse sia locali (tramite PKI) che cloud.

### 4.2. App Microsoft Authenticator

- **Focus:** Trasforma il telefono cellulare (iOS o Android) in una credenziale sicura.
- **Meccanismo:** Invece di una password, l'utente riceve una notifica sul telefono e deve abbinare un numero visualizzato sullo schermo del PC (anti-phishing) e confermare con PIN o biometria.
- **Vantaggio:** Offre un'ottima combinazione di sicurezza e praticità su qualsiasi piattaforma/browser.

### 4.3. Chiavi di Sicurezza FIDO2 (Fast Identity Online)

- **Focus:** Lo standard globale, aperto e più resistente al phishing.
- **Meccanismo:** Utilizza una chiave di sicurezza hardware (USB, Bluetooth, NFC) per l'autenticazione. La chiave esegue la crittografia asimmetrica per la prova di identità, garantendo che non venga esposta alcuna "credenziale segreta" condivisibile.
- **Resistenza al Phishing:** Poiché la chiave FIDO2 verifica l'URL di destinazione prima dell'autenticazione, è intrinsecamente immune agli attacchi che cercano di reindirizzare le credenziali a un sito falso.
- **Vantaggio:** Rappresenta lo **standard aureo** per l'autenticazione sicura e senza password.

---

# Descrivere le Identità Esterne e la Collaborazione

L'**Identità Esterna** (Microsoft Entra External ID) è un termine ombrello che comprende tutte le modalità con cui è possibile interagire e collaborare in modo sicuro con utenti che non fanno parte del proprio **Tenant** aziendale.

## 1. Microsoft Entra External ID: Le Tre Modalità

|Funzionalità|Scopo Principale|Tipologia di Utente|Rappresentazione Locale|
|---|---|---|---|
|**Collaborazione B2B (Business to Business)**|Collaborazione ad-hoc con partner, fornitori o altre aziende. Condivisione di documenti e app (SaaS/Custom).|Utenti Partner, Fornitori.|Utente **Guest** (BGP Type) con Identità gestita dal loro IdP originale (Entra ID, Google, MSA, ecc.).|
|**Connessione Diretta B2B**|Collaborazione fluida e continua con un partner **Entra ID** specifico, focalizzata su carichi di lavoro condivisi.|Utenti di organizzazioni partner.|L'utente **non** viene creato come guest nel tenant. Le autorizzazioni sono basate sul **Trust Bidirezionale** reciproco.|
|**Azure AD B2C (Business to Customer)**|Gestione di utenti e clienti per applicazioni _pubbliche_ rivolte ai consumatori (e-commerce, portali clienti, ecc.).|Consumatori, Clienti.|**Tenant B2C separato** (non il tenant aziendale).|

### 1.1. Collaborazione B2B: L'Utente Guest

La B2B Collaboration consente di invitare utenti esterni a utilizzare la propria identità preferita (Entra ID, account Microsoft, Google, email one-time passcode) per accedere alle risorse del tuo tenant.

- **Processo:** L'utente esterno viene rappresentato nella directory come **Utente Guest**.
- **Gestione dell'Accesso:** L'organizzazione ospite gestisce solo l'accesso e l'autorizzazione alle proprie risorse. L'autenticazione rimane responsabilità dell'**Identity Provider (IdP)** dell'utente guest.
- **Governance:** È cruciale abbinare la B2B Collaboration a funzionalità di governance come **Microsoft Entra ID Governance** per:
    - **Revisioni di Accesso (Access Reviews):** Per ricertificare periodicamente la necessità di accesso dell'ospite (parte del principio Zero Trust di JIT/JEA).
    - **Gestione Entitlement (Entitlement Management):** Per automatizzare il processo di richiesta, approvazione e provisioning/deprovisioning degli utenti guest.

### 1.2. Connessione Diretta B2B: Il Trust Senza Guest

Questa funzionalità è progettata per una collaborazione più profonda con organizzazioni specifiche che utilizzano anch'esse Entra ID.

- **Focus:** Attualmente si concentra sulla collaborazione nei **Canali Condivisi di Microsoft Teams**.
- **Differenza Chiave:** A differenza della B2B Collaboration, l'utente partner **non** viene creato come un oggetto `Guest` nella directory. Ciò mantiene la directory più pulita e semplifica la gestione del ciclo di vita dell'utente, poiché l'organizzazione partner gestisce totalmente la loro identità e le loro politiche.

# Accesso Condizionale (Conditional Access - CA) in Dettaglio

L'**Accesso Condizionale** è il motore policy-driven di **Microsoft Entra ID** che implementa i principi **Zero Trust** a livello di accesso. 
Non si limita a concedere o negare l'accesso, ma modella il modo in cui l'accesso viene concesso.

### 2.1. Anatomia di un Criterio di Accesso Condizionale

Un criterio CA è una semplice istruzione **"Se... Allora..."**:

> **SE** (Condizioni/Segnali) soddisfatte **ALLORA** (Controlli/Azioni) applicate.

#### 1. Condizioni (SE) - I Segnali d'Identità

Le condizioni sono i segnali raccolti in tempo reale da Entra ID per valutare il rischio e il contesto.

| Segnale                    | Descrizione e Applicazione                                                                                                                                                                       |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Utenti/Gruppi/Ruoli**    | Chi è l'utente. Permette di mirare il criterio a ruoli sensibili (es. Global Administrator).                                                                                                     |
| **Applicazioni Cloud**     | La risorsa a cui si sta tentando di accedere (es. Exchange Online, Azure Management Portal, App SaaS).                                                                                           |
| **Rischio Utente/Accesso** | Fornito da **Microsoft Entra ID Protection** (richiede Entra ID P2). Segnale dinamico basato su attività sospette (es. Accesso da IP anomalo, credenziali trapelate).                            |
| **Posizione**              | Consente di definire **Posizioni Denominate** (Trusted Locations) basate su indirizzi IP o Paesi/Regioni.                                                                                        |
| **Stato del Dispositivo**  | Verifica se il dispositivo è **Conforme** (gestito da Intune) o **Entra Hybrid Joined** (unito all'AD DS locale).                                                                                |
| **App Client**             | Tipo di applicazione utilizzata (Browser, App Mobile, Client Legacy). Utilizzato per bloccare i **Protocolli di Autenticazione Legacy** (es. Exchange ActiveSync) a favore di metodi più sicuri. |

#### 2. Controlli (ALLORA) - L'Imposizione della Decisione

I controlli sono le azioni che Entra ID impone in base ai segnali:

- **Consentire (Grant):** Permettere l'accesso. Può essere vincolato:
    - **Richiedi MFA:** Forza l'Autenticazione a Più Fattori.
    - **Richiedi che il dispositivo sia Entra Hybrid Joined:** Solo dispositivi aziendali gestiti possono accedere.
    - **Richiedi che il dispositivo sia segnato come conforme:** Solo dispositivi con requisiti di sicurezza e patching aggiornati possono accedere.
- **Bloccare (Block):** Nega completamente l'accesso.
- **Controlli di Sessione (Session Controls):** Permettono un'azione _durante_ la sessione:
    - **Frequenza di accesso:** Forza una ri-autenticazione dopo un certo periodo di tempo (es. ogni 4 ore).
    - **Usa restrizioni imposte dalle app:** Collabora con app Microsoft come SharePoint per limitare download o stampe da browser non gestiti.

### 2.2. Esempi Pratici e Strategie

|Obiettivo Zero Trust|Configurazione CA (SE... ALLORA...)|
|---|---|
|**Protezione Amministrativa**|**SE** l'utente è nel gruppo _Ruoli Amministrativi_ **ALLORA** _Richiedi MFA_ e _Richiedi che il dispositivo sia conforme_.|
|**Mitigazione del Rischio**|**SE** _Rischio Accesso_ è _Medio o Alto_ **ALLORA** _Blocca l'accesso_ (o _Richiedi cambio password_).|
|**Protezione Dati Ibrida**|**SE** l'app target è _SharePoint Online_ **E** _Posizione_ è _Esterna alla Rete Aziendale_ **ALLORA** _Usa Controlli di Sessione per bloccare il download_.|
|**Eliminazione Legacy**|**SE** _App Client_ include _Client di Autenticazione Legacy_ **ALLORA** _Blocca l'accesso_.|

> **Nota di Implementazione:** I criteri CA dovrebbero sempre essere implementati in modalità **Solo Report** inizialmente. Questo consente agli amministratori di monitorare l'impatto delle policy senza bloccare accidentalmente gli utenti, un passo cruciale per aderire al principio Zero Trust di "Presumere una Violazione" e testare le difese.

---

# Descrivere il Controllo degli Accessi in Base al Ruolo di Azure (RBAC)

Il **Controllo degli Accessi in Base al Ruolo (Role-Based Access Control - RBAC)** è il sistema di autorizzazione nativo di Azure che permette di gestire chi (utente, gruppo, entità servizio) può accedere a quali risorse e cosa può fare con tali risorse. È lo strumento primario per applicare il principio **Zero Trust** di **Privilegio Minimo** nel piano di controllo di Azure.

## 1. I Pilastri di Azure RBAC

RBAC si basa sulla creazione di un'**Assegnazione di Ruolo** (`Role Assignment`) che lega tre elementi fondamentali:

1. **Soggetto (Security Principal):** Chi ha bisogno dell'accesso (Utente, Gruppo di Microsoft Entra ID, Entità Servizio o Identità Gestita).
2. **Definizione del Ruolo (Role Definition):** Cosa il soggetto può fare (il set di autorizzazioni).
3. **Ambito (Scope):** Dove si applicano le autorizzazioni.

> **Definizione:** L'assegnazione di ruolo è l'atto di applicare una definizione di ruolo a un soggetto in un determinato ambito.

### 1.1. Definizione del Ruolo: Cosa si può fare?

Una definizione di ruolo è un insieme di autorizzazioni (`Actions`) e azioni negate (`NotActions`). Azure fornisce centinaia di **Ruoli Predefiniti** che coprono scenari comuni.

|Ruolo Predefinito|Ambito dell'Autorizzazione|Scopo|
|---|---|---|
|**Proprietario (Owner)**|Gestisce tutto, incluso l'accesso (RBAC) ad altre entità.|Responsabile totale della risorsa o dell'ambito.|
|**Collaboratore (Contributor)**|Può creare e gestire tutti i tipi di risorse, **escluso** l'accesso RBAC.|Ingegneri e sviluppatori che devono gestire l'infrastruttura.|
|**Lettore (Reader)**|Può visualizzare le risorse esistenti, **non può** modificarle o crearne di nuove.|Utenti di monitoraggio, Osservatori, personale di auditing.|
|**Ruoli Specifici**|Esempi: _Lettore Dati BLOB di Archiviazione_, _Amministratore Accesso Utente_.|Forniscono un accesso estremamente granulare (Principio del Privilegio Minimo).|

> **Creazione di Ruoli Personalizzati:** Se un ruolo predefinito non soddisfa i requisiti di autorizzazione, è possibile creare un **Ruolo Personalizzato Azure RBAC** definendo con precisione l'elenco delle azioni consentite.

### 1.2. Ambito: Dove si applicano le autorizzazioni?

L'**Ambito** definisce il livello di gerarchia delle risorse a cui si applica l'assegnazione di ruolo. 
In Azure, l'RBAC è **gerarchico** e le autorizzazioni vengono **ereditate** dal livello superiore a quelli inferiori.

La gerarchia degli ambiti in ordine decrescente di autorità è:

1. **Gruppo di Gestione (Management Group):** Collezioni di Sottoscrizioni (livello più alto).
2. **Sottoscrizione (Subscription):** L'unità di fatturazione e servizio.
3. **Gruppo di Risorse (Resource Group):** Contenitore logico per le risorse correlate.
4. **Risorsa Singola (Individual Resource):** Unità specifica (es. una VM, un Account di Archiviazione).

> **Esempio Pratico:** Se assegni il ruolo _Lettore_ a un gruppo nell'ambito della **Sottoscrizione**, i membri del gruppo potranno **visualizzare** automaticamente ogni Gruppo di Risorse e ogni Risorsa contenuta in quella Sottoscrizione (Ereditarietà).

## 2. Come Viene Applicato RBAC (Il Ruolo di Azure Resource Manager)

L'applicazione dell'RBAC è gestita da **Azure Resource Manager (ARM)**, il servizio di gestione che media ogni richiesta di gestione delle risorse in Azure (come creazione, modifica o eliminazione).

### 2.1. Il Modello di Autorizzazione Additivo

Azure RBAC opera su un **modello di autorizzazione additivo**:

- Quando a un soggetto vengono assegnati più ruoli, le autorizzazioni risultanti sono la **somma** dei permessi di tutti i ruoli. Se un ruolo concede l'accesso in scrittura e un altro ruolo concede l'accesso in lettura sulla stessa risorsa, l'utente avrà sia la lettura che la scrittura.
- L'unica eccezione è l'uso esplicito dell'azione `NotActions` in una definizione di ruolo o l'applicazione di **Azure Deny Assignments** (negazioni che hanno la precedenza sulle concessioni).

> **Ambito di Applicazione:** L'RBAC di Azure è limitato al **piano di controllo** (gestione delle risorse). **Non** gestisce l'accesso a livello di applicazione o di dati. 
> Ad esempio, per controllare chi accede ai dati all'interno di un BLOB, è necessario utilizzare i meccanismi di autorizzazione specifici del servizio (come ACL o ruoli specifici del piano dati di Storage Account).

# Riepilogo: Il Modello Zero Trust in Azure

Il **Modello Zero Trust** è una strategia di sicurezza che permea ogni decisione architetturale in Azure, fungendo da contesto per strumenti come Conditional Access e Azure RBAC.

| Principio Zero Trust                     | Implementazione Chiave in Azure                                                                                                        | Strumento / Servizio                                                                     |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Verificare Esplicitamente**            | L'accesso è concesso solo dopo la verifica completa dell'identità, del dispositivo, del rischio e della posizione.                     | **Microsoft Entra ID** e **Conditional Access**                                          |
| **Usare il Privilegio Minimo (JIT/JEA)** | Limitare l'autorizzazione di un soggetto a ciò che è strettamente necessario per completare l'attività, e solo per il tempo richiesto. | **Azure RBAC** e **Microsoft Entra PIM (Privileged Identity Management)**                |
| **Presumere una Violazione**             | Progettare i sistemi pensando che una violazione è inevitabile. Segmentare e crittografare tutto.                                      | **Micro-segmentazione di Rete**, **Crittografia Dati**, **Microsoft Defender for Cloud** |

Il passaggio dal vecchio modello perimetrale al **Modello Zero Trust** riflette la realtà del cloud e della forza lavoro mobile, dove l'identità (gestita da Entra ID) è diventata il nuovo perimetro di sicurezza primario.

---

# Descrivere la Difesa in Profondità (Defense in Depth - DiD)

La **Difesa in Profondità (DiD)** è una strategia di sicurezza a più livelli che utilizza una serie di meccanismi di difesa per proteggere le informazioni e mitigare i danni in caso di attacco. L'obiettivo è rallentare l'avanzamento di un aggressore e fornire molteplici punti di avviso, evitando di fare affidamento su un unico livello di protezione.

> **Principio Chiave:** Se un livello di protezione fallisce, il livello successivo è pronto a intervenire per impedire o limitare un'ulteriore esposizione.

## 1. I Sette Livelli di Difesa in Profondità di Azure layers

Il modello DiD si articola in una serie di livelli concentrici, con i **Dati** al centro, come l'asset più critico da proteggere. 
Azure fornisce strumenti specifici per ciascuno di questi livelli.

| Livello DiD                  | Scopo e Obbiettivo                                                                                                                                      | Strumenti Rilevanti in Azure                                                                                                                                                                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Sicurezza Fisica**      | Proteggere l'hardware fisico e i data center da accessi non autorizzati, furti o danni.                                                                 | **Responsabilità di Microsoft.** Controlli biometrici, recinzioni, sorveglianza 24/7.                                                                                                         |
| **2. Identità e Accesso**    | Assicurare che solo le entità verificate e autorizzate (persone, servizi) possano accedere alle risorse. **Il nuovo perimetro.**                        | **Microsoft Entra ID**, Conditional Access, MFA, SSO, Azure PIM.                                                                                                                              |
| **3. Perimetro (Perimeter)** | Proteggere l'intera rete cloud da attacchi su larga scala prima che raggiungano le risorse interne.                                                     | **Azure DDoS Protection** (per mitigare attacchi volumetrici), **Azure Front Door** e **Azure Web Application Firewall (WAF)**.                                                               |
| **4. Rete (Networking)**     | Segmentare l'ambiente cloud e limitare la comunicazione solo a ciò che è strettamente necessario (principio del "Negare per impostazione predefinita"). | **Gruppi di Sicurezza di Rete (NSG)** (filtro a livello di subnet/NIC), **Azure Firewall** (controllo centralizzato e logging).                                                               |
| **5. Calcolo (Compute)**     | Proteggere i carichi di lavoro in esecuzione, come Macchine Virtuali (VM), Contenitori o Funzioni Serverless.                                           | **Microsoft Defender for Cloud** (per la protezione degli endpoint e la gestione delle vulnerabilità), **Azure Bastion** (per l'accesso sicuro a VM senza IP pubblico), **Gestione Patch**.   |
| **6. Applicazione**          | Garantire che le applicazioni siano sviluppate in modo sicuro (Security by Design), gestendo vulnerabilità e segreti.                                   | **Azure Key Vault** (per archiviare segreti, chiavi e certificati), **Azure DevOps Security (DevSecOps)**, Controlli WAF.                                                                     |
| **7. Dati**                  | Controllare l'accesso ai dati sensibili e assicurarne la riservatezza, l'integrità e la disponibilità (CIA Triad).                                      | **Crittografia a riposo** (tramite servizi come Azure Storage Encryption, Crittografia Dischi Azure) e **Crittografia in transito** (TLS/SSL). Controlli di accesso granulari del piano dati. |

## 2. Punti Focali del Modello DiD

### Livello 2: Identità e Accesso (Il Nuovo Perimetro)

Questo livello è cruciale perché, con la scomparsa del perimetro di rete tradizionale (On-Premise), l'identità è diventata il principale punto di controllo. 
L'obiettivo non è solo l'autenticazione (`SSO`, `MFA`), ma anche l'**Autorizzazione** (`RBAC`) e l'**Audit** (registrazione degli eventi di accesso e delle modifiche).

### Livello 4: Rete (Micro-segmentazione)

La limitazione della connettività tramite la **Micro-segmentazione** è essenziale per limitare il **movimento laterale** (Lateral Movement). 
Se un aggressore compromette una risorsa, la segmentazione (es. tramite NSG) impedisce che possa raggiungere facilmente altre risorse sulla stessa rete.

### Livello 6 & 7: Applicazione e Dati (Dati Sensibili)

- **Applicazione:** La sicurezza deve essere integrata all'inizio del ciclo di vita dello sviluppo (Shift Left). La corretta gestione dei segreti (credential, connection string) in **Azure Key Vault** è un requisito critico per prevenire l'esposizione di informazioni sensibili nel codice.
- **Dati:** La **Crittografia** è il meccanismo fondamentale. I dati devono essere protetti sia quando sono _a riposo_ (storage, database, dischi VM) sia quando sono _in transito_ (comunicazioni di rete). La responsabilità di applicare i controlli sul dato spetta all'utente che lo archivia.

---

# Descrivere Microsoft Defender per il Cloud (MDC)

**Microsoft Defender per il Cloud (MDC)** è una soluzione unificata per la **gestione della postura di sicurezza** (CSPM) e la **protezione avanzata dei carichi di lavoro** (CWPP) per gli ambienti **multi-cloud** (Azure, AWS, GCP) e **ibridi**.
Defender per il Cloud è un servizio nativo di Azure che fornisce indicazioni, monitoraggio e protezione per rafforzare la sicurezza e difendersi dalle minacce informatiche.

## 1. I Tre Pilastri di Microsoft Defender per il Cloud 

MDC soddisfa tre esigenze fondamentali nella gestione della sicurezza:

|Pilastro|Descrizione e Funzione|Obbiettivi di Sicurezza|
|---|---|---|
|**Valutare Continuamente (CSPM)**|Monitoraggio continuo della postura di sicurezza del tuo ambiente, identificando configurazioni errate e vulnerabilità.|Conoscere lo stato di sicurezza, Identificare le lacune di configurazione.|
|**Proteggere (CSPM)**|Rafforzare la sicurezza delle risorse applicando configurazioni e politiche basate su standard di riferimento.|Ridurre la superficie di attacco, Rispettare i benchmark di settore.|
|**Difendere (CWPP)**|Rilevare e rispondere in tempo reale alle minacce e agli attacchi in corso sulle risorse protette.|Rilevamento e Risoluzione delle minacce, Analisi della catena di attacco.|

## 2. Valutazione e Protezione (CSPM)

Le funzionalità di **Cloud Security Posture Management (CSPM)** di MDC aiutano a identificare i rischi e a rafforzare la configurazione di sicurezza.

### 2.1. Azure Security Benchmark (ASB)

- **Standard Guida:** Le raccomandazioni di sicurezza di MDC sono basate sull'**Azure Security Benchmark (ASB)**. L'ASB è un insieme di linee guida e best practice specifiche di Azure, create da Microsoft e allineate ai framework di conformità comuni (es. CIS, NIST, PCI DSS).
- **Implementazione:** I controlli di sicurezza in MDC si basano su **Azure Policy** e possono essere applicati a livello di **Gruppo di Gestione**, **Sottoscrizione** o **Tenant**.

### 2.2. Punteggio di Sicurezza e Controlli

- **Punteggio di Sicurezza (Secure Score):** Fornisce un indicatore immediato e quantificabile dell'efficacia della postura di sicurezza. Il punteggio è calcolato in base al numero di raccomandazioni risolte e al loro impatto.
- **Controlli di Sicurezza:** Le singole raccomandazioni sono raggruppate in "Controlli" (es. "Abilita MFA", "Gestisci l'accesso"). Questo offre agli utenti un elenco di azioni prioritarie per massimizzare l'aumento del loro punteggio e, di conseguenza, della loro sicurezza.

### 2.3. Copertura Multi-Cloud e Ibrida

Le funzionalità CSPM di MDC si estendono automaticamente a:

- **Risorse di Azure:** Essendo nativo, monitora immediatamente i servizi Azure.
- **Ambienti Multi-Cloud (AWS, GCP):** Le funzionalità CSPM sono estese **senza agente** (agentless) alle risorse di altri cloud tramite l'integrazione con i rispettivi API Cloud. MDC valuta la conformità delle risorse AWS e GCP rispetto ai loro standard specifici.

## 3. Difesa e Protezione Carichi di Lavoro (CWPP)

Le funzionalità di **Cloud Workload Protection Platform (CWPP)** di MDC, fornite dai diversi piani **Microsoft Defender**, rilevano le minacce sulle risorse di calcolo e dati.

### 3.1. Protezione con Agente (Ibrido/Multi-Cloud)

- **Azure Arc:** Per estendere la protezione avanzata alle VM e ai server in esecuzione in ambienti **ibridi (locali)** o in **altri cloud**, si utilizza **Azure Arc**. Arc funge da "ponte" permettendo a MDC di gestire e monitorare le macchine non-Azure.
- **Agente Log Analytics:** L'agente viene distribuito per raccogliere dati di sicurezza dettagliati, essenziali per la protezione contro le minacce.

### 3.2. Rilevamento di Minacce Specifiche

MDC offre piani specifici per tipo di carico di lavoro che garantiscono una protezione avanzata:

| Piano Microsoft Defender       | Protezione Focalizzata                                | Insight di Sicurezza                                                                                                                                       |
| ------------------------------ | ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Defender per Server**        | VM Windows/Linux, Server On-Premise (tramite Arc).    | **Integrazione nativa con Microsoft Defender for Endpoint** (analisi vulnerabilità, EDR/XDR), **Controlli Applicazioni Adattivi** (whitelisting software). |
| **Defender per Database**      | Azure SQL, SQL Server su VM.                          | Rilevamento anomalie, **Valutazione delle vulnerabilità** e classificazione automatica dei dati.                                                           |
| **Defender per Archiviazione** | Azure Storage Accounts (BLOB, File, ecc.).            | Rilevamento malware, avvisi per attività anomale di accesso/estrazione dati.                                                                               |
| **Defender per App Service**   | Applicazioni Web e API ospitate in Azure App Service. | Rilevamento attacchi a livello di runtime.                                                                                                                 |

### 3.3. Mitigazione del Rischio di Rete e Avvisi

- **Accesso VM JIT (Just-in-Time):** Una funzionalità CWPP essenziale che riduce l'esposizione agli attacchi di forza bruta. Le porte di gestione delle VM (es. RDP/SSH) sono chiuse per impostazione predefinita e vengono aperte **solo su richiesta**, per un **periodo limitato** e solo da **indirizzi IP sorgente specifici**. Questo implementa il principio **Privilegio Minimo**.
- **Avvisi di Sicurezza:** Quando una minaccia viene rilevata, MDC genera avvisi dettagliati che includono la risorsa interessata, i passaggi di correzione e, nel caso di attacchi complessi, un'analisi unificata della **catena di attacco** per comprendere l'intera cronologia dell'incidente.

