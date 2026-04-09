# Schema Documentation  
**Job:** `PushClientToCMAWithSurvivorship`  

---

## 1. Overview  

The **PushClientToCMAWithSurvivorship** job extracts client‑level data from the core banking database, enriches it with reference information (officer, sales point, department), evaluates survivorship rules, and prepares the consolidated view for downstream loading into the Customer Master Application (CMA).  

Four Talend components feed data into the job:

| Component | Source Table / View | Purpose |
|-----------|--------------------|---------|
| `tMSSqlInput` #1 | `CMA.dbo.CLIENT` (raw client record) | Baseline client attributes. |
| `tMSSqlInput` #2 | `dbo.CLIENT` (master‑client view with joins) | Master‑client details plus derived relationship fields. |
| `tMSSqlInput` #3 | `dbo.CLIENT` (non‑master vs. master comparison) | Survivorship flag (`Survived_SF_Account`) and additional account status info. |
| `tHashInput` | In‑memory hash named **Client** | Cached client rows used later in the flow (primary key = `Client_ID`). |

No explicit target table definition is supplied in the metadata; the job’s output is therefore assumed to be written to a downstream CMA staging area that matches the combined schema described below.

---

## 2. Source Schemas  

### 2.1 `CLIENT_RAW` – Result set of **tMSSqlInput #1**  

| Column | Talend Type | Nullable | Key | Business Description |
|--------|-------------|----------|-----|-----------------------|
| `Client_ID` | id_String(-1) | No | – | Unique identifier assigned to the client by the source system. |
| `Record_Created_Date` | id_Date(-1) | Yes | – | Timestamp when the client record was first created. |
| `Record_Modified_Date` | id_Date(-1) | Yes | – | Timestamp of the most recent update to the client record. |
| `Record_Deleted` | id_String(-1) | Yes | – | Flag (`Yes`/`No`) indicating whether the record has been logically deleted. |
| `Tax_ID_Number` | id_String(-1) | Yes | – | Government‑issued tax identification number (e.g., SSN, EIN). |
| `SalesForce_Account_ID` | id_String(-1) | Yes | – | Identifier of the corresponding Salesforce account, if linked. |
| `SalesForce_Contact_ID` | id_String(-1) | Yes | – | Identifier of the corresponding Salesforce contact record. |
| `Client_Name` | id_String(-1) | Yes | – | Full legal name of the client (individual or organization). |
| `Status` | id_String(-1) | Yes | – | Current lifecycle status (e.g., Active, Inactive, Prospect). |
| `Acquisition_Date` | id_Date(-1) | Yes | – | Date the client was first acquired by the institution. |
| `Attrition_Date` | id_Date(-1) | Yes | – | Date the client left or was closed out. |
| `Branch_Code` | id_String(-1) | Yes | – | Code of the branch that originally onboarded the client. |
| `Department_ID` | id_String(-1) | Yes | – | Internal department responsible for the client relationship. |
| `Officer_ID` | id_String(-1) | Yes | – | Identifier of the relationship officer assigned to the client. |
| `Credit_Information_Sharable` | id_String(-1) | Yes | – | Permission flag (`Yes`/`No`) to share credit information with third parties. |
| `Client_Information_Sharable` | id_String(-1) | Yes | – | Permission flag to share general client information. |
| `Email_Solicitation_Allowed` | id_String(-1) | Yes | – | Indicates if email marketing solicitations are permitted. |
| `Mail_Solicitation_Allowed` | id_String(-1) | Yes | – | Indicates if postal mail solicitations are permitted. |
| `Phone_Solicitation_Allowed` | id_String(-1) | Yes | – | Indicates if phone solicitations are permitted. |
| `Text_Solicitation_Allowed` | id_String(-1) | Yes | – | Indicates if SMS/text solicitations are permitted. |
| `Profit_Tier` | id_String(-1) | Yes | – | Internal profitability tier classification (e.g., A, B, C). |
| `Penetration_Tier` | id_String(-1) | Yes | – | Tier reflecting product penetration depth. |
| `Client_Type` | id_String(-1) | Yes | – | Classification of client (`Individual`, `Business`, etc.). |
| `Source_Application_ID` | id_String(-1) | Yes | – | Identifier of the originating application/system. |
| `Relationship_ID` | id_String(-1) | Yes | – | Identifier linking related client records (used for survivorship). |
| `Address_ID` | id_String(-1) | Yes | – | Foreign key to the client’s primary address record. |
| `Is_Internal` | id_String(-1) | Yes | – | Flag indicating an internal (employee) client. |
| `First_Name` | id_String(-1) | Yes | – | Given name (for individuals). |
| `Last_Name` | id_String(-1) | Yes | – | Family name (for individuals). |
| `Middle_Name` | id_String(-1) | Yes | – | Middle name or initial. |
| `Suffix_Name` | id_String(-1) | Yes | – | Name suffix (Jr., Sr., III, etc.). |
| `Preferred_Name` | id_String(-1) | Yes | – | Nickname or preferred display name. |
| `Is_Military` | id_String(-1) | Yes | – | Flag indicating military affiliation. |
| `Date_of_Birth` | id_Date(-1) | Yes | – | Birthdate of an individual client. |
| `Passport_ID` | id_String(-1) | Yes | – | Passport number (if applicable). |
| `Drivers_License_ID` | id_String(-1) | Yes | – | Driver’s licence number (if applicable). |
| `Income` | id_BigDecimal(-1) | Yes | – | Reported annual income (currency). |
| `Investable_Assets` | id_BigDecimal(-1) | Yes | – | Total assets available for investment. |
| `Net_Worth` | id_BigDecimal(-1) | Yes | – | Overall net worth of the client. |
| `Employer` | id_String(-1) | Yes | – | Current employer name (for individuals). |
| `Is_Associate` | id_String(-1) | Yes | – | Flag indicating the client is an associate of the bank. |
| `Deceased_Date` | id_Date(-1) | Yes | – | Date of death (if applicable). |
| `Is_Wealth` | id_String(-1) | Yes | – | Flag marking the client as a wealth management prospect. |
| `Salutation` | id_String(-1) | Yes | – | Formal salutation (Mr., Ms., Dr., etc.). |
| `NAICS_ID` | id_String(-1) | Yes | – | Industry classification code (for businesses). |
| `Employee_Headcount` | id_Integer(-1) | Yes | – | Number of employees (business clients). |
| `Sales_Volume` | id_BigDecimal(-1) | Yes | – | Annual sales volume (business clients). |
| `Incorporation_Year` | id_Integer(-1) | Yes | – | Year the business entity was incorporated. |
| `Incorporation_Date` | id_Date(-1) | Yes | – | Exact incorporation date. |
| `Tax_ID_Type` | id_String(-1) | Yes | – | Type of tax identifier (SSN, EIN, etc.). |
| `Website` | id_String(-1) | Yes | – | Public website URL of the client (business). |
| `Stage` | id_String(-1) | Yes | – | Marketing or sales stage (e.g., Lead, Qualified). |
| `Added_Date` | id_Date(-1) | Yes | – | Date the client was added to the system (duplicate of acquisition). |
| `First_Account_Date` | id_Date(-1) | Yes | – | Date of the client’s first opened account. |
| `Credit_Information_Sharable_Date` | id_Date(-1) | Yes | – | Effective date of the credit‑share permission. |
| `Inactive_Date` | id_Date(-1) | Yes | – | Date the client became inactive. |
| `Maintenance_Date` | id_Date(-1) | Yes | – | Last maintenance activity date. |
| `Has_OLB_Profile` | id_String(-1) | Yes | – | Flag indicating an Online Banking profile exists. |
| `Email_ID` | id_String(-1) | Yes | – | Primary email address identifier. |
| `Phone_ID` | id_String(-1) | Yes | – | Primary phone number identifier. |

---

### 2.2 `CLIENT_MASTER` – Result set of **tMSSqlInput #2**  

This query returns a *master* view of client records (including joined reference data). Most columns mirror those in `CLIENT_RAW` but with additional derived fields.

| Column | Talend Type | Nullable | Key | Business Description |
|--------|-------------|----------|-----|-----------------------|
| `Is_Master` | id_String(3) | No | – | Indicates whether the row represents the master client in a survivorship group (`Yes`/`No`). |
| `Client_ID` | id_String(50) | No | – | Unique client identifier (same as in raw view). |
| `Record_Created_Date` | id_Date(23) | No | – | Creation timestamp. |
| `Record_Modified_Date` | id_Date(23) | No | – | Last modification timestamp. |
| `Record_Deleted` | id_String(3) | No | – | Logical delete flag. |
| `Master_Client_ID` | id_String(50) | Yes | – | Identifier of the master client for this record (null for the master itself). |
| `Tax_ID_Number` … *(all columns up to `Maintenance_Date`)* | same as raw view | Yes | – | Direct pass‑through of the raw client attributes. |
| `Match_Group_ID` | id_String(200) | Yes | – | Grouping key that ties together duplicate client records identified by the matching engine. |
| `Match_Group_Quality` | id_Double(15) | Yes | – | Numeric score representing confidence of the match (higher = better). |
| `Match_Group_Size` | id_Integer(10) | Yes | – | Number of records in the match group. |
| `Matching_Distance` | id_String(2000) | Yes | – | Serialized representation of the distance metrics used during matching. |
| `Match_Process` | id_String(20) | Yes | – | Name of the matching process that produced the group. |
| `Match_Group_Type` | id_String(30) | Yes | – | Classification of the match group (e.g., “Exact”, “Fuzzy”). |
| `Nbr_Of_Accounts` | id_Integer(10) | Yes | – | Count of financial accounts linked to the client. |
| `All_Accounts_Closed` | id_String(3) | Yes | – | Flag (`Yes`/`No`) indicating whether every linked account is closed. |
| `Batch_ID` | id_String(50) | Yes | – | Identifier of the ETL batch run that produced the row. |
| `Has_OLB_Profile` | id_String(3) | Yes | – | Same as raw view. |
| `Client_Information_Sharable_Date` | id_Date(23) | Yes | – | Effective date for client‑information sharing permission. |
| `Mastery_Key` | id_String(50) | Yes | – | Composite key used internally for survivorship logic. |
| `Relationship_ID` | id_String(-1) | Yes | – | Populated only for business clients; holds the parent Salesforce account ID. |
| `Account_Status` | id_String(-1) | Yes | – | Consolidated status of all accounts belonging to the client (derived from `ACCOUNT_ANY`). |
| `Origination_Date` | id_Date(10) | Yes | – | Earliest account origination date for the client. |
| `Closed_Date` | id_Date(10) | Yes | – | Latest account closure date for the client. |

*Derived columns (see Section 4):* `Relationship_ID`, `Account_Status`, `Origination_Date`, `Closed_Date`.

---

### 2.3 `CLIENT_SURVIVOR` – Result set of **tMSSqlInput #3**  

This query compares a *non‑master* client (`C_NM`) with its *master* counterpart (`C_M`) to decide which record survives.

| Column | Talend Type | Nullable | Key | Business Description |
|--------|-------------|----------|-----|-----------------------|
| `Is_Master` | id_String(3) | No | – | Always `'No'` for the left side (non‑master). |
| `Client_ID` | id_String(30) | No | – | Identifier of the non‑master client. |
| `Record_Created_Date` … *(through `Maintenance_Date`)* | same as raw view | No/Yes | – | Standard audit fields for the non‑master record. |
| `Match_Group_ID` … *(through `Matching_Distance`)* | same as master view | Yes | – | Matching group identifiers (identical to master). |
| `Mastery_Key` | id_String(50) | Yes | – | Internal key used for linking master/non‑master rows. |
| `Salesforce_Parent_Account_ID` | id_String(18) | Yes | – | Parent account identifier from Salesforce (used only for business clients). |
| `Survived_SF_Account` | id_Integer(10) | Yes | – | **Derived flag** – `1` if the non‑master client’s ID equals the concatenated string `'SF' + SalesForce_Account_ID`; otherwise `0`. Indicates whether the Salesforce account survived the deduplication. |
| `Account_Status` | id_String(-1) | Yes | – | Consolidated account status (derived via `ACCT_ANY`). |
| `Origination_Date` | id_Date(10) | Yes | – | Earliest account origination date for the non‑master client. |
| `Closed_Date` | id_Date(10) | Yes | – | Latest account closure date for the non‑master client. |
| *(All other client attribute columns are present, mirroring the raw view, often with `ISNULL(...,'')` defaults.)* |

---

### 2.4 `HASH_CLIENT` – Result set of **tHashInput**  

A cached copy of client rows used for fast look‑ups downstream.

| Column | Talend Type | Nullable | Key | Business Description |
|--------|-------------|----------|-----|-----------------------|
| `Client_ID` | id_String(30) | No | **PK** | Primary key – unique client identifier. |
| `Source_Client_ID` | id_String(50) | No | – | Original source identifier before any transformation. |
| `Record_Created_Date` … `Phone_ID
