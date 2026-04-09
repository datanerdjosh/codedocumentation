# Schema Documentation: BatchHouseholdMastery  

## Job Overview  
The **BatchHouseholdMastery** job reads household data from three sources, applies business rules to determine the current master record, and flags rows for insert or update (CDC). The primary inputs are:  

* **STG.HOUSEHOLD_IN_PROCESS** – staging view of households that are being processed.  
* **dbo.HOUSEHOLD** – production master table that stores existing household records.  
* **REF.PERSONA** – reference table that defines persona identifiers and their priority.  

A hash‑based input (`tHashInput`) supplies additional fields that are used in the CDC logic. The job produces a unified result set (the UNION of staging rows and selected master rows) that downstream components can load into a target system.  

---

## Source Schemas  

### 1. STG.HOUSEHOLD_IN_PROCESS (tMSSqlInput)  

| Column | Talend Type / Length | Nullable | Key | Business Description |
|--------|----------------------|----------|-----|-----------------------|
| Household_ID | id_String(50) | No | – | Unique identifier for the household in the staging area. |
| SalesForce_Account_ID | id_String(18) | Yes | – | Identifier of the related Salesforce account, if any. |
| Is_Master | id_String(3) | No | – | Flag indicating whether the row is a master record. For staging rows the job forces this value to **‘No’**. |
| Status | id_String(20) | Yes | – | Current lifecycle status of the household (e.g., Active, Inactive). |
| Branch_Code | id_String(6) | Yes | – | Code of the branch that owns the household. |
| Department_ID | id_String(20) | Yes | – | Identifier of the department responsible for the household. |
| Officer_ID | id_String(6) | Yes | – | Identifier of the relationship officer assigned to the household. |
| Credit_Information_Sharable | id_String(3) | Yes | – | Indicates if credit information may be shared (`Yes`/`No`). |
| Client_Information_Sharable | id_String(3) | Yes | – | Indicates if client‑level information may be shared. |
| Email_Solicitation_Allowed | id_String(3) | Yes | – | Permission flag for email marketing. |
| Mail_Solicitation_Allowed | id_String(3) | Yes | – | Permission flag for postal mail marketing. |
| Phone_Solicitation_Allowed | id_String(3) | Yes | – | Permission flag for phone contact. |
| Text_Solicitation_Allowed | id_String(3) | Yes | – | Permission flag for SMS/text contact. |
| Source_Application_ID | id_String(5) | Yes | – | Identifier of the source application that created the record. |
| Last_Name | id_String(100) | Yes | – | Household primary contact’s last name. If a matching Salesforce record exists, the value from Salesforce overrides the staging value (see SQL). |
| Income | id_BigDecimal(20) | Yes | – | Reported annual income for the household. |
| Income_Date | id_Date(10) | Yes | – | Date when the income figure was recorded. |
| Income_Source | id_String(5) | Yes | – | Code indicating the source of the income information. |
| Income_Level_ID | id_String(20) | Yes | – | Classification of income level (e.g., Low, Medium, High). |
| Income_Level_Date | id_Date(10) | Yes | – | Date the income level classification was applied. |
| Income_Level_Source | id_String(5) | Yes | – | Source system for the income level classification. |
| Investable_Assets | id_BigDecimal(20) | Yes | – | Total investable assets reported for the household. |
| Investable_Assets_Date | id_Date(10) | Yes | – | Date the investable assets value was captured. |
| Investable_Assets_Source | id_String(5) | Yes | – | Source system for the investable assets figure. |
| Sub_Persona_ID | id_String(10) | Yes | – | Identifier linking the household to a sub‑persona definition. |
| Net_Worth | id_BigDecimal(20) | Yes | – | Calculated net worth of the household. |
| Added_Date | id_Date(10) | Yes | – | Date the household record was first added to the system. |
| First_Account_Date | id_Date(10) | Yes | – | Date of the first account opened by the household. |
| Maintenance_Date | id_Date(10) | Yes | – | Most recent maintenance (update) timestamp. |
| Mastery_Key | id_String(580) | Yes | – | Composite key used to identify duplicate households. May be overridden by logic in the UNION query (see Derived Columns). |
| CDC_FLAG | id_String(19) | No | – | Set by the job to indicate **INSERT** or **UPDATE** for Change Data Capture. For staging rows the value is derived from the presence of a matching master group (`IIF(H_M.Match_Group_ID IS NOT NULL, 'UPDATE', 'INSERT')`). |
| Record_Deleted | id_String(3) | Yes | – | Soft‑delete flag (`Yes`/`No`). |
| Is_Associate | id_String(10) | Yes | – | Flag indicating whether the household is an associate of another household. |
| Phase | id_String(20) | Yes | – | Business phase (e.g., Prospect, Active, Closed). |
| Address_ID | id_String(50) | Yes | – | Identifier of the address record linked to the household. |
| Acquisition_Date | id_Date(23) | Yes | – | Date the household was acquired (e.g., onboarding date). |
| Attrition_Date | id_Date(23) | Yes | – | Date the household left the program, if applicable. |
| Record_Type | id_String(100) | Yes | – | Categorisation of the record type (e.g., Individual, Joint). |
| Record_Created_Date | *not present in this source* | – | – | – |
| Record_Modified_Date | *not present in this source* | – | – | – |
| Master_Household_ID | *not present in this source* | – | – | – |
| Household_Name | *not present in this source* | – | – | – |
| Match_Group_ID | *not present in this source* | – | – | – |
| Match_Group_Quality | *not present in this source* | – | – | – |
| Match_Group_Size | *not present in this source* | – | – | – |
| Matching_Distance | *not present in this source* | – | – | – |
| Match_Process | *not present in this source* | – | – | – |
| Match_Group_Type | *not present in this source* | – | – | – |
| Batch_ID | *not present in this source* | – | – | – |

> **Note:** Columns listed after `Record_Type` belong to the **tHashInput** component (see section 3) and are not part of the staging table itself.

---

### 2. dbo.HOUSEHOLD (tMSSqlInput)  

The production master table shares the same column list as the staging table, with identical data types and meanings. The key differences are:  

* **Is_Master** reflects the actual master status (`Yes` or `No`).  
* The **Mastery_Key**, **Match_Group_ID**, **Match_Group_Quality**, **Match_Group_Size**, **Matching_Distance**, **Match_Process**, **Match_Group_Type**, **Batch_ID**, and **CDC_FLAG** columns are populated based on existing master‑matching logic.  

All columns are documented in the table above; they retain the same business definitions.

---

### 3. REF.PERSONA (tMSSqlInput)  

| Column | Talend Type | Nullable | Key | Business Description |
|--------|-------------|----------|-----|-----------------------|
| Sub_Persona_ID | id_String(-1) | No | Primary Key | Identifier of a sub‑persona used to segment households (e.g., “Young Professional”). |
| Priority | id_Integer(-1) | Yes | – | Numeric priority that determines the ordering of personas when multiple matches exist (lower number = higher priority). |

---

### 4. Hash Input (tHashInput)  

| Column | Talend Type / Length | Nullable | Key | Business Description |
|--------|----------------------|----------|-----|-----------------------|
| Is_Master | id_String(3) | No | – | Mirrors the master flag from the source record. |
| Household_ID | id_String(50) | No | – | Unique household identifier (same as in staging/master tables). |
| SalesForce_Account_ID | id_String(18) | Yes | – | Salesforce account reference. |
| Record_Created_Date | id_Date(23) | Yes | – | Timestamp when the record was originally created. |
| Record_Modified_Date | id_Date(23) | Yes | – | Timestamp of the most recent modification. |
| Record_Deleted | id_String(3) | No | – | Soft‑delete indicator. |
| Master_Household_ID | id_String(50) | Yes | – | Identifier of the master household when this row is an associate. |
| Household_Name | id_String(100) | Yes | – | Descriptive name of the household. |
| Status | id_String(20) | Yes | – | Lifecycle status. |
| Phase | id_String(20) | Yes | – | Business phase. |
| Acquisition_Date | id_Date(23) | Yes | – | Acquisition date. |
| Attrition_Date | id_Date(23) | Yes | – | Attrition date. |
| Branch_Code | id_String(6) | Yes | – | Branch ownership code. |
| Department_ID | id_String(20) | Yes | – | Department identifier. |
| Officer_ID | id_String(6) | Yes | – | Relationship officer identifier. |
| Credit_Information_Sharable | id_String(3) | Yes | – | Credit sharing permission. |
| Client_Information_Sharable | id_String(3) | Yes | – | Client information sharing permission. |
| Email_Solicitation_Allowed | id_String(3) | Yes | – | Email solicitation consent. |
| Mail_Solicitation_Allowed | id_String(3) | Yes | – | Postal solicitation consent. |
| Phone_Solicitation_Allowed | id_String(3) | Yes | – | Phone solicitation consent. |
| Text_Solicitation_Allowed | id_String(3) | Yes | – | SMS solicitation consent. |
| Source_Application_ID | id_String(5) | Yes | – | Originating application code. |
| Last_Name | id_String(100) | Yes | – | Primary contact’s last name. |
| Income | id_BigDecimal(20) | Yes | – | Annual income. |
| Income_Date | id_Date(10) | Yes | – | Date of income capture. |
| Income_Source | id_String(5) | Yes | – | Source of income data. |
| Income_Level_ID | id_String(20) | Yes | – | Income level classification. |
| Income_Level_Date | id_Date(10) | Yes | – | Date of income level assignment. |
| Income_Level_Source | id_String(5) | Yes | – | Source of income level data. |
| Investable_Assets | id_BigDecimal(20) | Yes | – | Investable assets amount. |
| Investable_Assets_Date | id_Date(10) | Yes | – | Date assets were measured. |
| Investable_Assets_Source | id_String(5) | Yes | – | Source of assets data. |
| Sub_Persona_ID | id_String(10) | Yes | – | Link to persona definition. |
| Net_Worth | id_BigDecimal(20) | Yes | – | Net worth figure. |
| Added_Date | id_Date(23) | Yes | – | Record creation date. |
| First_Account_Date | id_Date(23) | Yes | – | First account opening date. |
| Maintenance_Date | id_Date(23) | Yes | – | Last maintenance timestamp. |
| Address_ID | id_String(50) | Yes | – | Address reference. |
| Match_Group_ID | id_String(60) | Yes | – | Identifier of the duplicate‑match group the household belongs to. |
| Match_Group_Quality | id_Double(20) | Yes | – | Quality score of the match group (higher = better confidence). |
| Match_Group_Size | id_Integer(10) | Yes | – | Number of households in the match group. |
| Matching_Distance | id_String(2000) | Yes | – | Serialized distance metrics used by the matching algorithm. |
| Match_Process | id_String(20) | Yes | – | Name of the process that generated the match (e.g., “DailyDedup”). |
| Match_Group_Type | id_String(30) | Yes | – | Classification of the match group (e.g., “Exact”, “Fuzzy”). |
| Batch_ID | id_String(50) | Yes | – | Identifier of the batch run that produced the record. |
| Mastery_Key | id_String(580) | Yes | – | Composite key used for deduplication and master determination. |
| Is_Associate | id_String(10) | Yes | – | Flag indicating associate status. |
| Record_Type | id_String(100) | Yes | – | Type of household record. |

---

## Derived Columns (Logic Implemented in the UNION Query)

| Derived Column | Expression (SQL) | Business Meaning |
|----------------|------------------|------------------|
| **Is_Master** (for staging rows) | `'No' AS Is_Master` | Staging rows are always treated as non‑master; master status is resolved later via lookup. |
| **Last_Name** (staging side) | `ISNULL(H_SF.[Last_Name], H_IP.[Last_Name])` | Prefer the last name from the linked Salesforce master record; fall back to the staging value if none exists. |
| **Mastery_Key** | `CASE WHEN H_SF.Household_ID IS NOT NULL THEN H_SF.Mastery_Key WHEN H_M.Household_ID IS NOT NULL AND H_M.Mastery_Key NOT LIKE 'Client%' AND H_IP.Mastery_Key NOT LIKE 'Client%' AND H_IP.Address_ID = H_M.Address_ID THEN H_M.Mastery_Key ELSE H_IP.[Mastery_Key] END` | Determines the definitive mastery key: use Salesforce master if available; otherwise, if a non‑client master exists with the same address, adopt its key; else keep the staging key. |
| **CDC_FLAG** (staging side) | `IIF(H_M.Match_Group_ID IS NOT NULL, 'UPDATE', 'INSERT')` | Marks the row for **UPDATE** when the household already belongs to a match group; otherwise marks it for **INSERT**. |
| **CDC_FLAG** (master side) | `'MATCH_NEW_HOUSEHOLD' AS CDC_FLAG` | For master rows selected from `dbo.HOUSEHOLD`, indicates that a new household match has been identified. |
| **Record_Deleted**, **Is_Associate**, **Phase**, **Address_ID**, **Acquisition_Date**, **Attrition_Date**, **Record_Type** | Direct pass‑through from source tables (either staging or master) | Preserve original attribute values for downstream processing. |

---

## Data Type Mapping (Talend ↔ SQL Server)  

| Talend Type | SQL Server Equivalent (as seen in source) | Typical Oracle Equivalent |
|-------------|-------------------------------------------|----------------------------|
| id_String(n) | `VARCHAR(n)` | `VARCHAR2(n)` |
| id_BigDecimal(p)
