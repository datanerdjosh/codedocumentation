# Data Flow: PushClientToCMAWithSurvivorship  

---  

## Visual Flow Diagram  

```
+---------------------------+        +---------------------------+
|   tMSSqlInput (CLIENT)    |        |   tMSSqlInput (Complex   |
|   SELECT TOP 1000 ...     |        |   Client Query)          |
|   FROM CMA.dbo.CLIENT)    |        |   SELECT ... FROM dbo.CLIENT|
+------------+--------------+        +------------+--------------+
             |                                   |
             |                                   |
             v                                   v
+---------------------------------------------------------------+
|                         tMap_1                               |
|   (pass‑through mapping – no expressions)                     |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                tMSSqlOutput (Client Load)                    |
|   Writes rows returned by first two tMSSqlInput components    |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                 tDBInput_3 (Third Client Query)              |
|   SELECT ... FROM dbo.CLIENT C_NM JOIN dbo.CLIENT C_M ...   |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                  tPartitioner_1 (hash partition)            |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                  tCollector_1 (collect partitions)          |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                     tLogRow_1 (debug logging)               |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                         tMap_3                               |
|   (pass‑through mapping – no expressions)                     |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                tRuleSurvivorship_1 (survivorship)            |
|   Applies survivorship rules to determine the surviving      |
|   client record within each match group.                     |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                         tMap_2                               |
|   (pass‑through mapping – no expressions)                     |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                 tDepartitioner_1 (partition)                 |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                 tRecollector_1 (re‑assemble)                |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                 tHashOutput (store hashed rows)             |
|   Persists intermediate result set for later use.            |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                 tHashInput (read hashed rows)                |
|   Reads the rows written by tHashOutput.                     |
+----------------------+----------------------------------------+
                       |
                       v
+---------------------------------------------------------------+
|                tDBOutput_2 (Final Load)                      |
|   Writes survivorship‑processed rows to target table(s).     |
+---------------------------------------------------------------+
```

---  

## Source Extraction  

### 1️⃣ tMSSqlInput – “Client Sample”  

* **SQL Query**  

```sql
SELECT TOP (1000) 
      [Client_ID],[Record_Created_Date],[Record_Modified_Date],
      [Record_Deleted],[Tax_ID_Number],[SalesForce_Account_ID],
      [SalesForce_Contact_ID],[Client_Name],[Status],[Acquisition_Date],
      [Attrition_Date],[Branch_Code],[Department_ID],[Officer_ID],
      [Credit_Information_Sharable],[Client_Information_Sharable],
      [Email_Solicitation_Allowed],[Mail_Solicitation_Allowed],
      [Phone_Solicitation_Allowed],[Text_Solicitation_Allowed],
      [Profit_Tier],[Penetration_Tier],[Client_Type],[Source_Application_ID],
      [Relationship_ID],[Address_ID],[Is_Internal],[First_Name],[Last_Name],
      [Middle_Name],[Suffix_Name],[Preferred_Name],[Is_Military],
      [Date_of_Birth],[Passport_ID],[Drivers_License_ID],[Income],
      [Investable_Assets],[Net_Worth],[Employer],[Is_Associate],
      [Deceased_Date],[Is_Wealth],[Salutation],[NAICS_ID],
      [Employee_Headcount],[Sales_Volume],[Incorporation_Year],
      [Incorporation_Date],[Tax_ID_Type],[Website],[Stage],
      [Added_Date],[First_Account_Date],[Credit_Information_Sharable_Date],
      [Inactive_Date],[Maintenance_Date],[Has_OLB_Profile],
      [Email_ID],[Phone_ID]
FROM [CMA].[dbo].[CLIENT]
```

* **Extracted Columns** – All columns listed in the `SELECT` clause (approximately 60 fields).  
* **Filter Conditions** – `TOP (1000)` limits the row count to a maximum of 1 000 rows; no `WHERE` clause.  
* **Business Meaning** – Provides a quick‑sample of client master data for diagnostic or downstream validation purposes.

---

### 2️⃣ tMSSqlInput – “Non‑Master Clients (Unique Match Group)”  

* **SQL Query**  

```sql
SELECT CL.[Is_Master], CL.[Client_ID], CL.[Record_Created_Date],
       CL.[Record_Modified_Date], CL.[Record_Deleted], CL.[Master_Client_ID],
       CL.[Tax_ID_Number], CL.[SalesForce_Account_ID],
       CL.[Salesforce_Contact_ID], CL.[Client_Name], CL.[Status],
       SP.[Branch_Code] AS Client_Branch_Code,
       ACCT_BRANCH.Branch_Code AS Account_Branch_Code,
       ISNULL(SP.[Department_ID], D2.Department_ID) AS Client_Department_ID,
       ACCT_BRANCH.Department_ID AS Account_Department_ID,
       O.[Officer_ID] AS Client_Officer_ID,
       ACCT_OFF.Officer_ID AS Account_Officer_ID,
       CL.[Credit_Information_Sharable], CL.[Client_Information_Sharable],
       CL.[Email_Solicitation_Allowed], CL.[Mail_Solicitation_Allowed],
       CL.[Phone_Solicitation_Allowed], CL.[Text_Solicitation_Allowed],
       CL.[Client_Type], CL.[Source_Application_ID], CL.[Is_Internal],
       CL.[First_Name], CL.[Last_Name], CL.[Middle_Name], CL.[Suffix_Name],
       CL.[Preferred_Name], CL.[Is_Military], CL.[Date_of_Birth],
       CL.[Passport_ID], CL.[Drivers_License_ID], CL.[Income],
       CL.[Investable_Assets], CL.[Net_Worth], CL.[Employer],
       CL.[Is_Associate], CL.[Deceased_Date], CL.[Is_Wealth],
       CL.[Salutation], CL.[NAICS_ID], CL.[Employee_Headcount],
       CL.[Sales_Volume], CL.[Incorporation_Year], CL.[Tax_ID_Type],
       CL.[Website], CL.[Added_Date], CL.[First_Account_Date],
       CL.[Inactive_Date], CL.[Maintenance_Date], CL.[Match_Group_ID],
       CL.[Match_Group_Quality], CL.[Match_Group_Size],
       CL.[Matching_Distance], CL.[Match_Process], CL.[Match_Group_Type],
       CL.[Nbr_Of_Accounts], CL.[All_Accounts_Closed], CL.[Batch_ID],
       CL.[Has_OLB_Profile], CL.Client_Information_Sharable_Date,
       CL.Mastery_Key,
       CASE WHEN CL.Client_Type = 'Business' THEN CL.Salesforce_Parent_Account_ID ELSE NULL END AS [Relationship_ID],
       ACCT_ANY.Status AS Account_Status,
       ACCT_ANY.Origination_Date, ACCT_ANY.Closed_Date
FROM [dbo].[CLIENT] CL
LEFT JOIN REF.OFFICER O ON CL.Officer_ID = O.Officer_ID
LEFT JOIN REF.SALES_POINT SP ON CL.Branch_Code = SP.Branch_Code
LEFT JOIN REF.DEPARTMENT D ON SP.Department_ID = D.Department_ID
LEFT JOIN REF.DEPARTMENT D2 ON CL.Department_ID = D2.Department_ID
OUTER APPLY ( … ) ACCT_OFF
OUTER APPLY ( … ) ACCT_BRANCH
OUTER APPLY ( … ) ACCT_ANY
WHERE CL.Is_Master = 'No'
  AND CL.Match_Group_Size = 1
  AND Batch_ID = '" + (String)globalMap.get("BatchId") + "'
```

* **Extracted Columns** – All fields listed in the `SELECT` clause, plus derived columns (`Client_Branch_Code`, `Account_Branch_Code`, `Client_Department_ID`, `Account_Department_ID`, `Client_Officer_ID`, `Account_Officer_ID`, `Relationship_ID`, `Account_Status`, `Origination_Date`, `Closed_Date`).  
* **Filter Conditions**  
  * `CL.Is_Master = 'No'` – selects only **non‑master** client records.  
  * `CL.Match_Group_Size = 1` – keeps records whose match group contains a single client (i.e., no duplicates).  
  * `Batch_ID = <runtime batch id>` – restricts processing to the current CDM batch.  
* **Business Meaning** – Retrieves non‑master client rows that are uniquely matched within the current batch, enriching them with related officer, branch, department, and account information for downstream survivorship evaluation.

---

### 3️⃣ tMSSqlInput – “Survivorship Candidate Set (Master + Non‑Master)”  

* **SQL Query**  

```sql
SELECT CASE WHEN C_NM.[Client_ID] = C_M.[Client_ID] THEN 'Yes' ELSE 'No' END AS [Is_Master],
       C_NM.[Client_ID], C_NM.[Record_Created_Date], C_NM.[Record_Modified_Date],
       C_NM.[Record_Deleted], C_NM.[Master_Client_ID], C_NM.[Tax_ID_Number],
       C_M.[SalesForce_Account_ID], C_M.[Salesforce_Contact_ID],
       C_NM.[Client_Name], C_NM.[Status],
       SP.[Branch_Code] AS Client_Branch_Code,
       ACCT_BRANCH.Branch_Code AS Account_Branch_Code,
       ISNULL(SP.[Department_ID], D2.Department_ID) AS Client_Department_ID,
       ACCT_BRANCH.Department_ID AS Account_Department_ID,
       O.[Officer_ID] AS Client_Officer_ID,
       ACCT_OFF.Officer_ID AS Account_Officer_ID,
       ISNULL(C_NM.[Credit_Information_Sharable], '') AS Credit_Information_Sharable,
       ISNULL(C_NM.[Client_Information_Sharable], '') AS Client_Information_Sharable,
       ISNULL(C_NM.[Email_Solicitation_Allowed], '') AS Email_Solicitation_Allowed,
       ISNULL(C_NM.[Mail_Solicitation_Allowed], '') AS Mail_Solicitation_Allowed,
       ISNULL(C_NM.[Phone_Solicitation_Allowed], '') AS Phone_Solicitation_Allowed,
       ISNULL(C_NM.[Text_Solicitation_Allowed], '') AS Text_Solicitation_Allowed,
       C_NM.[Client_Type], C_NM.[Source_Application_ID],
       ISNULL(C_NM.[Is_Internal], '') AS Is_Internal,
       C_NM.[First_Name], C_NM.[Last_Name], C_NM.[Middle_Name],
       C_NM.[Suffix_Name], C_NM.[Preferred_Name],
       ISNULL(C_NM.[Is_Military], '') AS Is_Military,
       C_NM.[Date_of_Birth], C_NM.[Passport_ID], C_NM.[Drivers_License_ID],
       C_NM.[Income], C_NM.[Investable_Assets], C_NM.[Net_Worth],
       C_NM.[Employer] AS Employer,
       ISNULL(C_NM.[Is_Associate], '') AS Is_Associate,
       C_NM.[Deceased_Date], C_NM.[Is_Wealth], C_NM.[Salutation],
       C_NM.[NAICS_ID], C_NM.[Employee_Headcount], C_NM.[Sales_Volume],
       C_NM.[Incorporation_Year], C_NM.[Tax_ID_Type], C_NM.[Website],
       C_NM.[Added_Date], C_NM.[First_Account_Date], C_NM.[Inactive_Date],
       C_NM.[Maintenance_Date],
       C_M.[Match_Group_ID], C_M.[Match_Group_Quality], C_M.[Match_Group_Size],
       C_M.[Matching_Distance], C_M.[Match_Process], C_M.[Match_Group_Type],
       C_NM.[Nbr_Of_Accounts], C_NM.[All_Accounts_Closed],
       C_NM.[Batch_ID], C_NM.[Has_OLB_Profile],
       C_NM.Client_Information_Sharable_Date, C_NM.Mastery_Key,
       C_NM.Salesforce_Parent_Account_ID,
       CASE WHEN 'SF' + C_M.SalesForce_Account_ID = C_NM.Client_ID THEN 1 ELSE 0 END AS Survived_SF_Account,
       ACCT_ANY.Status AS Account_Status,
       ACCT_ANY.Origination_Date, ACCT_ANY.Closed_Date
FROM [dbo].[CLIENT] C_NM
INNER JOIN dbo.CLIENT C_M ON C_NM.Match_Group_ID = C_M.Match_Group_ID
LEFT JOIN REF.OFFICER O ON C_NM.Officer_ID = O.Officer_ID
LEFT JOIN REF.SALES_POINT SP ON C_NM.Branch_Code = SP.Branch_Code
LEFT JOIN REF.DEPARTMENT D ON SP.Department_ID = D.Department_ID
LEFT JOIN REF.DEPARTMENT D2 ON C_NM.Department_ID = D2.Department_ID
OUTER APPLY ( … ) ACCT_OFF
OUTER APPLY ( … ) ACCT_BRANCH
OUTER APPLY ( … ) ACCT_ANY
WHERE C_NM.Is_Master = 'No'
  AND C_M.Is_Master = 'Yes'
  AND C_M.Match_Group_Size > 1
  AND C_NM.Batch_ID = '" + (String)globalMap.get("BatchId") + "'
ORDER BY Match_Group_ID
```

* **Extracted Columns** – Full list of fields in the
