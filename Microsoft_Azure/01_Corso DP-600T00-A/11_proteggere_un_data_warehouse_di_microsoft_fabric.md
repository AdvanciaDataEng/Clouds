---
autore: Kevin Milli
---

Il **Data Warehouse in Microsoft Fabric** è costruito sul familiare motore SQL, offrendo agli ingegneri dei dati una suite completa di funzionalità di sicurezza native basate su T-SQL. L'obiettivo primario è garantire la protezione dei dati sensibili e la conformità normativa, implementando una strategia di **difesa in profondità (defense-in-depth)** che riduce al minimo la superficie di attacco.

---

## Livelli di Sicurezza e Controllo Accessi

La sicurezza in un Fabric Data Warehouse è stratificata, operando a livello di area di lavoro, di oggetto e granulare (riga e colonna).

### Sicurezza a Livello di Area di Lavoro (Workspace Roles)

I ruoli definiscono l'accesso e il controllo sull'intera area di lavoro di Fabric. 
Sono il primo e più ampio livello di sicurezza.

|Ruolo|Accesso e Controllo|Finalità Principale|
|---|---|---|
|**Amministratore**|Controllo totale, gestione di membri e impostazioni.|Gestione complessiva e configurazione.|
|**Membro**|Può creare, modificare ed eliminare elementi nell'area di lavoro.|Sviluppo e collaborazione attiva.|
|**Collaboratore**|Può creare, modificare ed eseguire query sugli elementi.|Contributo a progetti specifici.|
|**Visualizzatore**|Solo lettura su elementi e dati.|Consumo di dati e reportistica.|

### Sicurezza a Livello di Elemento e Granulare

Quando il Warehouse viene condiviso (**Item Permissions**), gli utenti possono accedere ai dati. 
È possibile utilizzare costrutti **T-SQL** standard per implementare una sicurezza più precisa (Granular Security).

- **Protezione a Livello di Oggetto:** Controlla l'accesso a oggetti specifici del database (tabelle, viste, procedure) tramite `GRANT`, `DENY`, `REVOKE`.
- **Protezione a Livello di Colonna:** Prevenzione della visualizzazione non autorizzata di colonne specifiche.
- **Protezione a Livello di Riga (RLS):** Limita le righe di dati che un utente può visualizzare (vedere sezione dedicata).
- **Mascheramento Dinamico dei Dati (DDM):** Offusca i dati sensibili senza alterare i dati sottostanti (vedere sezione dedicata).

> **Intuizione sul Motore SQL:** Poiché Fabric si basa sullo stesso motore SQL familiare (come in SQL Server/Azure SQL), la sicurezza granulare è gestita in modo trasparente e applicata a qualsiasi strumento di query o reportistica (inclusi i report Power BI in modalità Direct Query).

### Strumenti per la Conformità: SQL Audit Logs (Anteprima)

Per la conformità e la sicurezza avanzata, Fabric supporta i **Log di Controllo SQL (SQL Audit Logs)**. 
Questa funzionalità monitora e registra eventi chiave del database (come tentativi di autenticazione, operazioni sui dati, modifiche allo schema e alle autorizzazioni).

- **Implementazione:** Richiede l'abilitazione tramite API e l'autorizzazione `AUDIT`.
- **Accesso:** I log sono archiviati in OneLake (crittografati) e non sono direttamente visibili, ma possono essere interrogati tramite la funzione T-SQL:
```SQL
SELECT *
FROM sys.fn_get_audit_file_v2(
	-- Specifica i parametri del percorso
)
```

---

## Dynamic Data Masking (DDM)

**Dynamic Data Masking (DDM)** è una funzionalità di sicurezza che limita l'esposizione dei dati a utenti senza privilegi, nascondendo le informazioni sensibili **in tempo reale** senza modificarle in modo permanente nel database.

### Concetti Chiave del DDM

1. **Mascheramento in Tempo Reale:** Le regole di mascheramento vengono applicate al set di risultati della query.
2. **Invarianza dei Dati:** I dati effettivi nel database rimangono intatti e protetti.
3. **Semplicità d'Uso:** L'implementazione non richiede modifiche al codice applicativo o alle query esistenti.

### Regole di Mascheramento T-SQL

Le maschere sono configurate a livello di colonna e offrono diverse funzioni:

|Tipo di Maschera|Regola di Maschera|Descrizione|Output Esempio|
|---|---|---|---|
|**Default**|`default()`|Maschera completa basata sul tipo di dati.|`stringa` → `XXXX`, `int` → `0`, `float` → `0.0`|
|**E-mail**|`email()`|Espone il primo carattere e il suffisso ".com".|`nome@contoso.com` → `nxxx@xxxx.com`|
|**Testo Personalizzato (Partial)**|`partial(prefix, padding, suffix)`|Espone i primi (`prefix`) e gli ultimi (`suffix`) caratteri, riempiendo il centro con una stringa (`padding`).|`partial(0,"XXX-",4)` su `1234-5678` → `XXX-5678`|
|**Aleatorio (Random)**|`random(low, high)`|Sostituisce i valori numerici con un numero casuale nell'intervallo specificato.|`random(10, 100)` su `420` → `87`|

### Esempio Pratico di Configurazione DDM

Per applicare il mascheramento alle colonne sensibili di una tabella `Customers`:
```SQL
-- Applica la maschera E-mail
ALTER TABLE Customers
ALTER COLUMN Email ADD MASKED WITH (FUNCTION = 'email()');

-- Maschera Parziale per il Numero di Telefono (mostra solo le ultime 4 cifre)
ALTER TABLE Customers
ALTER COLUMN PhoneNumber ADD MASKED WITH (FUNCTION = 'partial(0,"XXX-XXX-",4)');

-- Maschera Parziale per il Numero di Carta di Credito (mostra solo le ultime 4 cifre)
ALTER TABLE Customers
ALTER COLUMN CreditCardNumber ADD MASKED WITH (FUNCTION = 'partial(0,"XXXX-XXXX-XXXX-",4)');

-- Per rimuovere una maschera
ALTER TABLE Customers
ALTER COLUMN ColumnName DROP MASKED;
```

> **Intuizione:** Le maschere DDM non sono pensate per l'isolamento dei tenant, ma per nascondere dati specifici (PII/PHI) in ambienti non autorizzati, come test, sviluppo o reportistica di base, fungendo da misura di protezione dati per la conformità (**GDPR/Privacy**).

---

## Row-Level Security (RLS)

La **Row-Level Security (RLS)** fornisce un controllo granulare sull'accesso alle righe di una tabella, filtrando i risultati delle query in base all'identità dell'utente o al contesto di esecuzione.

### Come Funziona la RLS

La RLS è implementata mediante due componenti T-SQL:

1. **Predicato di Sicurezza (Security Predicate):** Una funzione con valori di tabella _inline_ (`tvf`) che definisce la logica di accesso. La funzione restituisce `1` (true) se la riga è accessibile all'utente che esegue la query, o `0` (false) in caso contrario.
2. **Criterio di Sicurezza (Security Policy):** Un oggetto che lega la funzione predicato a una tabella, attivando l'applicazione della logica di filtro.

La RLS supporta due tipi di predicati:

- **Predicati di Filtro (`FILTER`):** Filtrano silenziosamente le righe per operazioni di lettura (`SELECT`, `UPDATE`, `DELETE`). L'utente semplicemente non vede le righe non autorizzate.
- **Predicati di Blocco (`BLOCK`):** Impediscono esplicitamente operazioni di scrittura (`INSERT`, `UPDATE`, `DELETE`) che violano la regola definita (sebbene l'implementazione in Fabric si concentri principalmente sul filtro).

|Operazione|Effetto del Predicato di Filtro|
|---|---|
|`SELECT`|Le righe non autorizzate non vengono visualizzate.|
|`UPDATE`|Le righe non autorizzate non possono essere aggiornate.|
|`DELETE`|Le righe non autorizzate non possono essere eliminate.|
|`INSERT`|L'operazione è permessa, ma la riga potrebbe essere filtrata nelle successive letture se l'utente non ha accesso.|

### Caso d'Uso: Isolamento Multitenant

RLS è fondamentale nelle architetture **multitenant**, dove più clienti (tenant) condividono la stessa tabella, ma ognuno deve vedere solo i propri dati.

**Obiettivo:** Consentire agli utenti normali (es. `tenant1@contoso.com`) di vedere solo le righe con il proprio `TenantName`, mentre un amministratore (`tenantAdmin@contoso.com`) vede tutto.

**Configurazione RLS con T-SQL:**

1. **Crea Schema e Funzione Predicato:**
```SQL
-- 1. Creare uno schema separato per gli oggetti RLS
CREATE SCHEMA [Sec];
GO

-- 2. Creare la funzione con valori di tabella inline (Predicato di filtro)
CREATE FUNCTION Sec.tvf_SecurityPredicatebyTenant(@TenantName AS NVARCHAR(50))
RETURNS TABLE
WITH SCHEMABINDING
AS
	RETURN  SELECT 1 AS result
			WHERE @TenantName = USER_NAME() OR USER_NAME() = 'tenantAdmin@contoso.com';
GO
```

- **Intuizione:** `USER_NAME()` restituisce il nome dell'utente che esegue la query. La funzione restituisce _true_ (risultato = 1) se il valore della colonna `TenantName` nella riga corrente corrisponde all'utente o se l'utente è l'amministratore.

2. **Crea Criterio di Sicurezza:**
```SQL
-- 3. Creare il Criterio di Sicurezza e collegare la funzione alla tabella
CREATE SECURITY POLICY Sec.SalesPolicy
ADD FILTER PREDICATE Sec.tvf_SecurityPredicatebyTenant(TenantName) ON [dbo].[Sales]
WITH (STATE = ON);
GO
```

> **Insight Cruciale per Fabric/Power BI:** Quando una query Power BI accede a un Warehouse in modalità **Direct Lake**, e quella tabella ha un criterio RLS, Power BI esegue automaticamente il **fallback alla modalità Direct Query** per garantire che i criteri RLS vengano rispettati a livello di database.

### Procedure Consigliate per RLS

Per massimizzare sicurezza e prestazioni:

- **Schema Separato:** Creare sempre uno schema separato (`Sec`) per le funzioni predicato e i criteri di sicurezza.
- **Prestazioni (Predicati):**
    - Evitare l'uso eccessivo di `JOIN` nelle funzioni predicato.
    - Evitare conversioni di tipo (`CAST`, `CONVERT`) all'interno del predicato.
    - Evitare la ricorsione (diretta o indiretta).
- **Indici:** Assicurarsi che la colonna utilizzata per l'isolamento (es. `TenantName` o `TenantID`) sia **indicizzata** per ottimizzare le prestazioni di filtro.
- **Principio del Privilegio Minimo:** L'autorizzazione `ALTER ANY SECURITY POLICY` dovrebbe essere concessa solo a utenti di alto livello (Security Policy Manager) e mai al ruolo applicativo standard.
- **Sicurezza come Default:** RLS offre **sicurezza per default**; se per errore si definisce una policy troppo restrittiva, gli utenti non vedranno dati, ma non ci sarà _data leak_.

La prima parte degli appunti ha coperto la sicurezza a livello di Area di Lavoro, RLS e DDM. Questa seconda parte completerà l'analisi concentrandosi sulla **Sicurezza a Livello di Colonna (CLS)** e sulle **Autorizzazioni Granulari SQL** (DML/DDL) tramite T-SQL, seguendo la struttura e le istruzioni fornite.

---

# Implementare la Sicurezza a Livello di Colonna (CLS)

La **Sicurezza a Livello di Colonna (CLS)** è un meccanismo di controllo degli accessi che consente di limitare la visibilità di colonne specifiche a utenti o ruoli autorizzati, fornendo un controllo granulare essenziale per la protezione dei dati sensibili all'interno del Warehouse.

## Proteggere Dati Sensibili

La CLS opera direttamente sulla struttura del database, garantendo che le restrizioni di accesso siano applicate in modo coerente da qualsiasi applicazione o strumento di reporting.

### Scenario Esempio: Sanità (Healthcare)

Consideriamo una tabella `dbo.Patients` nel settore sanitario contenente informazioni sensibili come la `MedicalHistory`. È fondamentale limitare l'accesso a questa colonna solo al personale medico autorizzato.

**Passaggi di Implementazione Concettuali:**

1. **Identificazione:** La colonna `MedicalHistory` è identificata come sensibile.
2. **Definizione Ruoli:** Si definiscono i ruoli aziendali (es. `Doctor`, `Nurse`, `Receptionist`, `Patient`).
3. **Assegnazione Utenti:** Gli utenti vengono mappati ai rispettivi ruoli.
4. **Controllo Accessi:** Si utilizza T-SQL per `DENY` l'autorizzazione `SELECT` sui dati sensibili ai ruoli non autorizzati.

## Configurazione della Sicurezza a Livello di Colonna con T-SQL

L'implementazione della CLS si basa sull'uso delle istruzioni `GRANT` e `DENY` con specifici riferimenti a livello di colonna.
```SQL
-- 1. Creare i ruoli applicativi
CREATE ROLE Doctor AUTHORIZATION dbo;
CREATE ROLE Nurse AUTHORIZATION dbo;
CREATE ROLE Receptionist AUTHORIZATION dbo;
CREATE ROLE Patient AUTHORIZATION dbo;
GO

-- 2. Concedere l'autorizzazione SELECT su TUTTA la tabella (necessario per accedere alle colonne non sensibili)
GRANT SELECT ON dbo.Patients TO Doctor;
GRANT SELECT ON dbo.Patients TO Nurse;
GRANT SELECT ON dbo.Patients TO Receptionist;
GRANT SELECT ON dbo.Patients TO Patient;
GO

-- 3. Negare SELECT in modo ESPLICITO sulla colonna sensibile ai ruoli non autorizzati
DENY SELECT ON dbo.Patients (MedicalHistory) TO Receptionist;
DENY SELECT ON dbo.Patients (MedicalHistory) TO Patient;
GO
```

>[!Nota] 
>**Principio di Prevalenza:** In T-SQL, un'istruzione **`DENY` esplicita prevale sempre su qualsiasi `GRANT`** (a livello di utente, ruolo o
>gruppo). In questo scenario, anche se `Receptionist` ha un `GRANT SELECT` sulla tabella, il `DENY SELECT` specifico sulla colonna
>`MedicalHistory` impedisce l'accesso a quei dati.

## CLS vs. Viste (Views): Analisi Comparativa

Sia la CLS che le Viste possono essere utilizzate per limitare l'accesso ai dati, ma operano a livelli diversi e offrono vantaggi distinti.

| Aspetto          | Sicurezza a Livello di Colonna (CLS)                                                                                                                                             | Viste (Views)                                                                                                                                                             |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Granularità**  | Controllo nativo e preciso per singola colonna e ruolo/utente.                                                                                                                   | Necessita di creare una vista diversa per ogni combinazione di set di colonne/autorizzazioni.                                                                             |
| **Manutenzione** | Bassa. Le autorizzazioni sono legate alla colonna; si adattano automaticamente ai cambiamenti strutturali della tabella (es. aggiunta di nuove colonne).                         | Alta. Le viste devono essere aggiornate manualmente se la struttura della tabella sottostante cambia.                                                                     |
| **Performance**  | Generalmente più efficiente, opera direttamente sui metadati di sicurezza.                                                                                                       | Può introdurre un leggero overhead, soprattutto con definizioni di viste complesse o su tabelle molto grandi.                                                             |
| **Trasparenza**  | Totale. L'utente esegue la query sulla tabella base, il motore SQL applica la restrizione in modo trasparente (la colonna appare come inesistente o come un errore di permessi). | Limitata. L'utente deve eseguire la query su un oggetto diverso (la Vista) e non sulla tabella base.                                                                      |
| **Flessibilità** | Focalizzata su colonne/oggetti.                                                                                                                                                  | Molto Flessibile. Può combinare sicurezza a livello di colonna (omettendo colonne) e **sicurezza a livello di riga** (aggiungendo una clausola `WHERE` alla definizione). |

---

# Configurare Autorizzazioni Granulari SQL con T-SQL

La gestione delle autorizzazioni granulari è il fondamento della sicurezza nel Warehouse, regolando chi può eseguire le operazioni di manipolazione dati (**DML**) e di definizione dati (**DDL**) su tabelle, viste e oggetti di programmazione.

## Autorizzazioni Fondamentali DML

Queste autorizzazioni controllano le operazioni sui dati e sono le più comuni nella gestione dei ruoli applicativi:

|Autorizzazione|Oggetto|Definizione|
|---|---|---|
|**`SELECT`**|Tabella, Vista, Colonna|Visualizzare i dati nell'oggetto.|
|**`INSERT`**|Tabella, Vista|Aggiungere nuove righe di dati.|
|**`UPDATE`**|Tabella, Vista, Colonna|Modificare i dati esistenti.|
|**`DELETE`**|Tabella, Vista|Rimuovere righe di dati.|

>[!Tips] 
>**Best Practice:** Quando si gestiscono i permessi, si lavora quasi sempre con i **Ruoli** (`CREATE ROLE`) e si assegnano gli utenti a
>quei ruoli, piuttosto che concedere o negare permessi direttamente ai singoli utenti. Questo semplifica notevolmente la
>manutenzione.

## Autorizzazioni di Controllo e Definizione (DDL/Control)

Queste autorizzazioni sono tipicamente riservate agli amministratori, agli sviluppatori e al personale di sicurezza.

|Autorizzazione|Oggetto|Definizione|
|---|---|---|
|**`EXECUTE`**|Stored Procedure, Funzione|Eseguire il codice dell'oggetto.|
|**`ALTER`**|Funzione, Stored Proc|Modificare la definizione dell'oggetto.|
|**`CONTROL`**|Qualsiasi oggetto|Concede tutti i diritti sull'oggetto, equivalente a esserne il proprietario (ma senza la capacità di eliminarlo in alcuni contesti).|

## Il Principio dei Privilegi Minimi (PoLP)

Il **Principio dei Privilegi Minimi (Principle of Least Privilege - PoLP)** è un cardine della sicurezza: **agli utenti e alle applicazioni devono essere concesse solo le autorizzazioni strettamente necessarie per completare il loro compito, e nient'altro.**

- **Esempio:** Se un'applicazione web recupera dati solo tramite una **Stored Procedure** (`sp_GetCustomerData`), l'utente del database associato all'applicazione deve avere solo il permesso **`GRANT EXECUTE ON sp_GetCustomerData`** e **nessun accesso diretto** alle tabelle sottostanti.

### Intuizione: SQL Dinamico e Sicurezza

L'SQL dinamico è una tecnica potente ma rischiosa in cui una query viene costruita come stringa di testo ed eseguita in un secondo momento.

**Esempio di Stored Procedure con SQL Dinamico:**
```SQL
CREATE PROCEDURE sp_TopTenRows @tableName NVARCHAR(128)
AS
BEGIN
    DECLARE @query NVARCHAR(MAX);
    -- QUOTENAME() è fondamentale per prevenire SQL Injection
    SET @query = N'SELECT TOP 10 * FROM ' + QUOTENAME(@tableName);
    EXEC sp_executesql @query;
END;
```

>[!Warning] 
>**Avvertimento di Sicurezza (SQL Injection):** Quando si usa SQL dinamico, è **imperativo** utilizzare funzioni di parametrizzazione
> come **`sp_executesql`** o **`QUOTENAME()`** per sanificare e delimitare in modo sicuro gli input utente. Senza queste protezioni, si
> espone il Warehouse al rischio di attacchi di **SQL Injection**.

