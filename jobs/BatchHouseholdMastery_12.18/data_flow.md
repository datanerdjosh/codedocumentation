# Data Flow: BatchHouseholdMastery  

---  

## Visual Flow Diagram  

```
+---------------------------+          +----------------------+          +--------------------+
|  STG.HOUSEHOLD_IN_PROCESS |          |   REF.PERSONA       |          |   dbo.HOUSEHOLD   |
|  (tMSSqlInput – DBInput_1) |          | (tMSSqlInput – DBInput_2) |      | (joined inside   |
|                           |          |                      |          |  first query)    |
+------------+--------------+          +----------+-----------+          +--------+-----------+
             |                                 |                               |
             |                                 |                               |
             +---------------+-----------------+-------------------------------+
                             |                                         |
                             v                                         v
                        +-------------------+                     +-------------------+
                        |   tMap_2          |<--------------------|   tMap_2          |
                        | (merge inputs)    |                     | (merge inputs)    |
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        |   tMatchGroup_1   |                     |   tMatchGroup_1   |
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        |   tPartitioner_1  |                     |   tPartitioner_1  |
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        |   tCollector_1    |                     |   tCollector_1    |
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        |   tMap_3          |                     |   tMap_3          |
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        | tRuleSurvivorship_|                     | tRuleSurvivorship_|
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        |   tMap_4          |                     |   tMap_4          |
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        | tDepartitioner_1  |                     | tDepartitioner_1  |
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        |  tRecollector_1   |                     |  tRecollector_1   |
                        +---------+---------+                     +---------+---------+
                                  |                                       |
                                  v                                       v
                        +-------------------+                     +-------------------+
                        |   tHashOutput_1   |<--------------------|   tHashOutput_1   |
                        |   (target)        |                     |   (target)        |
                        +-------------------+                     +-------------------+

Additional flow:
tHashInput → tDBOutput / tDBRow components → tMSSqlOutput (final target)
```  

*Note:* The diagram reflects the components and their connections as described in the job metadata. No Oracle TRANSACTIONS/ACCOUNTS tables are referenced in the source metadata; therefore they are omitted.  

---  

## Source Extraction  

| Source Component | SQL Query (as defined) | Extracted Columns | Filter / Join Conditions | Business Meaning |
|------------------|-----------------------|-------------------|--------------------------|------------------|
| **tMSSqlInput – DBInput_1** | ```sql SELECT  H_IP.[Household_ID] , H_IP.[SalesForce_Account_ID] ,  'No' AS Is_Master ,H_IP.[Status] ,H_IP.[Branch_Code] ,H_IP.[Department_ID] ,H_IP.[Officer_ID] ,H_IP.[Credit_Information_Sharable] ,H_IP.[Client_Information_Sharable] ,H_IP.[Email_Solicitation_Allowed] ,H_IP.[Mail_Solicitation_Allowed] ,H_IP.[Phone_Solicitation_Allowed] ,H_IP.[Text_Solicitation_Allowed] ,H_IP.[Source_Application_ID] ,ISNULL(H_SF.[Last_Name], H_IP.[Last_Name]) AS [Last_Name] ,H_IP.[Income] ,H_IP.[Income_Date] ,H_IP.[Income_Source] ,H_IP.[Income_Level_ID] ,H_IP.[Income_Level_Date] ,H_IP.[Income_Level_Source] ,H_IP.[Investable_Assets] ,H_IP.[Investable_Assets_Date] ,H_IP.[Investable_Assets_Source] ,H_IP.[Sub_Persona_ID] ,H_IP.[Net_Worth] ,H_IP.[Added_Date] ,H_IP.[First_Account_Date] ,H_IP.[Maintenance_Date] ,CASE WHEN H_SF.Household_ID IS NOT NULL THEN H_SF.Mastery_Key WHEN H_M.Household_ID IS NOT NULL AND H_M.Mastery_Key NOT LIKE 'Client%' AND H_IP.Mastery_Key NOT LIKE 'Client%' AND H_IP.Address_ID = H_M.Address_ID THEN H_M.Mastery_Key ELSE H_IP.[Mastery_Key] END AS Mastery_Key ,IIF(H_M.Match_Group_ID IS NOT NULL, 'UPDATE', 'INSERT') AS CDC_FLAG , H_IP.Record_Deleted , H_IP.Is_Associate , H_IP.Phase ,ISNULL(H_SF.[Address_ID], H_IP.[Address_ID]) AS [Address_ID] , H_IP.Acquisition_Date , H_IP.Attrition_Date , H_IP.Record_Type FROM [STG].[HOUSEHOLD_IN_PROCESS] H_IP LEFT OUTER JOIN dbo.HOUSEHOLD H_M ON H_IP.Household_ID = H_M.Household_ID AND H_M.Is_Master = 'No' LEFT OUTER JOIN dbo.HOUSEHOLD H_SF ON H_IP.Master_Household_ID = H_SF.Household_ID AND H_SF.Is_Master = 'Yes' UNION ALL SELECT H_M.[Household_ID] , H_M.[SalesForce_Account_ID] , H_M.Is_Master ,H_M.[Status] ,H_M.[Branch_Code] ,H_M.[Department_ID] ,H_M.[Officer_ID] ,H_M.[Credit_Information_Sharable] ,H_M.[Client_Information_Sharable] ,H_M.[Email_Solicitation_Allowed] ,H_M.[Mail_Solicitation_Allowed] ,H_M.[Phone_Solicitation_Allowed] ,H_M.[Text_Solicitation_Allowed] ,H_M.[Source_Application_ID] ,H_M.[Last_Name] ,H_M.[Income] ,H_M.[Income_Date] ,H_M.[Income_Source] ,H_M.[Income_Level_ID] ,H_M.[Income_Level_Date] ,H_M.[Income_Level_Source] ,H_M.[Investable_Assets] ,H_M.[Investable_Assets_Date] ,H_M.[Investable_Assets_Source] ,H_M.[Sub_Persona_ID] ,H_M.[Net_Worth] ,H_M.[Added_Date] ,H_M.[First_Account_Date] ,H_M.[Maintenance_Date] ,H_M.[Mastery_Key] ,'MATCH_NEW_HOUSEHOLD' AS CDC_FLAG , H_M.Record_Deleted , H_M.Is_Associate , H_M.Phase ,H_M.[Address_ID] , H_M.Acquisition_Date , H_M.Attrition_Date , H_M.Record_Type FROM [dbo].[HOUSEHOLD] H_M WHERE (H_M.Mastery_Key IN(SELECT Mastery_Key FROM STG.HOUSEHOLD_IN_PROCESS) OR H_M.Match_Group_ID IN( SELECT H_M_TWO.Match_Group_ID FROM dbo.HOUSEHOLD H_M_TWO INNER JOIN STG.HOUSEHOLD_IN_PROCESS H_IP ON H_M_TWO.Household_ID = H_IP.Household_ID ) OR H_M.Master_Household_ID IN(SELECT Master_Household_ID FROM STG.HOUSEHOLD_IN_PROCESS WHERE LEFT(Household_ID,2) = 'SF' ) ) AND H_M.Household_ID NOT IN(SELECT Household_ID FROM STG.HOUSEHOLD_IN_PROCESS) ``` | All columns listed in both SELECT blocks (approximately 40 columns, e.g., Household_ID, SalesForce_Account_ID, Is_Master, Status, Branch_Code, Department_ID, Officer_ID, Credit_Information_Sharable, Client_Information_Sharable, Email_Solicitation_Allowed, Mail_Solicitation_Allowed, Phone_Solicitation_Allowed, Text_Solicitation_Allowed, Source_Application_ID, Last_Name, Income, Income_Date, Income_Source, Income_Level_ID, Income_Level_Date, Income_Level_Source, Investable_Assets, Investable_Assets_Date, Investable_Assets_Source, Sub_Persona_ID, Net_Worth, Added_Date, First_Account_Date, Maintenance_Date, Mastery_Key, CDC_FLAG, Record_Deleted, Is_Associate, Phase, Address_ID, Acquisition_Date, Attrition_Date, Record_Type) | • LEFT OUTER JOIN to **dbo.HOUSEHOLD** (alias H_M) on Household_ID where H_M.Is_Master = ‘No’. <br>• LEFT OUTER JOIN to **dbo.HOUSEHOLD** (alias H_SF) on Master_Household_ID where H_SF.Is_Master = ‘Yes’. <br>• UNION ALL with a second SELECT pulling from **dbo.HOUSEHOLD** where any of the following is true: <br>  ‑ Mastery_Key exists in the staging table, <br>  ‑ Match_Group_ID matches a record linked to the staging table, <br>  ‑ Master_Household_ID references a Salesforce‑prefixed household. <br>• Excludes households already present in the staging table. | The query builds a unified view of “in‑process” household records together with existing master household records to decide whether each row represents an INSERT, UPDATE, or MATCH scenario for downstream processing. |
| **tMSSqlInput – DBInput_2** | ```sql SELECT [Sub_Persona_ID] ,[Priority] FROM [REF].[PERSONA] ``` | Sub_Persona_ID, Priority | None (full table scan) | Provides persona priority information that can be used to enrich household rows during the merge. |
| **tHashInput** | *No SQL statement supplied* (reads a previously created hash file) | Not specified in source metadata | Not specified in source metadata | Supplies pre‑hashed intermediate rows that are later written back to the database via tDBOutput/tDBRow components. |

---  

## Transformation Logic  

### Overview  
The job contains three **tMap** components (tMap_2, tMap_3, tMap_4). The source metadata indicates **“No tMap expressions found.”** Consequently, the maps act primarily as **pass‑through** connectors that forward fields from upstream components to downstream ones while preserving the row structure.  

### Detailed Steps  

| Step | Component | Primary Action | Derived / Calculated Fields | Business Purpose |
|------|-----------|----------------|-----------------------------|------------------|
| 1 | **tMap_2** | Merges the two input streams: <br>• DBInput_1 (household composite view) <br>• DBInput_2 (persona reference) | None (direct field mapping) | Aligns household rows with their corresponding persona priority, enabling downstream survivorship and partitioning logic. |
| 2 | **tMatchGroup_1** | Groups rows based on the **Match_Group_ID** (present in the household payload) | Implicit grouping key – Match_Group_ID | Identifies sets of records that belong to the same logical household match group for later deduplication. |
| 3 | **tPartitioner_1** | Partitions the grouped stream into parallel sub‑flows (internal Talend mechanism) | No new columns | Improves performance by allowing concurrent processing of independent match groups. |
| 4 | **tCollector_1** | Collects partitioned rows back into a single flow | No new columns | Re‑assembles the data after parallel processing, preserving order for deterministic survivorship evaluation. |
| 5 | **tMap_3** | Pass‑through map before survivorship rule | None | Provides a hook point for future derived calculations (currently unused). |
| 6 | **tRuleSurvivorship_1** | Applies a **survivorship rule set** (configuration not shown) to resolve duplicate records within each match group. | May set a flag such as **Is_Master** or select the “best” record based on business criteria (e.g., latest Maintenance_Date, highest Priority). | Guarantees that only one definitive household record per match group proceeds further, enforcing master‑record integrity. |
| 7 | **tMap_4** | Pass‑through map after survivorship | None | Allows additional field routing before final partitioning. |
| 8 | **tDepartitioner_1** | Splits the stream again for parallel downstream writes | No new columns | Enables concurrent loading into the hash store and relational target. |
| 9 | **tRecollector_1** | Re‑combines the departed streams into a single flow destined for the hash output. | No new columns | Ensures ordered delivery to the hash store. |
|10| **tHashOutput_1** | Writes the final row set to a **hash file** (temporary storage) | No new columns | Provides a fast, disk‑based staging area for bulk database load operations performed later by tDBRow / tDBOutput components. |
|11| **tDBRow / tDBOutput** (downstream of tHashInput) | Reads the hash file and performs **INSERT / UPDATE** statements against the target **dbo.HOUSEHOLD** (or related tables). | Uses CDC_FLAG (‘INSERT’, ‘UPDATE’, ‘MATCH_NEW_HOUSEHOLD’) to decide operation
