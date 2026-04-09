# Schema Documentation: CDMHistoricalArchives  

## Overview  
The **CDMHistoricalArchives** job extracts metadata about historic (archive) tables stored in a Microsoft SQL Server database. It reads the `INFORMATION_SCHEMA.TABLES` view, filters for schemas that start with `HIST`, and outputs a simple list containing the logical source schema name, the original target schema name, and the archive table name. No further transformation or loading into downstream tables is defined in the supplied metadata.

---

## Source Components  

| Component | Description | Columns (as defined in Talend) |
|-----------|-------------|--------------------------------|
| **tMSSqlInput** | Reads the list of archive tables from the source SQL Server instance using the query shown below. | <details><summary>Click to expand</summary> <br>**SourceSchema** – `id_String(50)` – Not nullable – Logical schema name derived from the original `TABLE_SCHEMA`. <br>**TargetSchema** – `id_String(50)` – Nullable – Original `TABLE_SCHEMA` value (e.g., `HIST_SALES`). <br>**ArchiveTable** – `id_String(100)` – Nullable – Name of the archive table (`TABLE_NAME`). </details> |
| **tFixedFlowInput** | Provides a static flow that mirrors the structure of the `tMSSqlInput` component (used for schema alignment or testing). | Same three columns as `tMSSqlInput` (identical definitions). |

### SQL Query Executed by **tMSSqlInput**  

```sql
SELECT CASE [TABLE_SCHEMA]
           WHEN 'HIST' THEN 'dbo'
           ELSE REPLACE(TABLE_SCHEMA, 'HIST_', '')
       END AS SourceSchema,
       [TABLE_SCHEMA] AS TargetSchema,
       [TABLE_NAME]   AS ArchiveTable
FROM   [INFORMATION_SCHEMA].[TABLES]
WHERE  TABLE_TYPE = 'BASE TABLE'
  AND  TABLE_SCHEMA LIKE 'HIST%';
```

* **SourceSchema** – If the schema is exactly `HIST`, it is mapped to `dbo`; otherwise the prefix `HIST_` is stripped, yielding the logical source schema name.  
* **TargetSchema** – The original schema name as stored in the database (kept for reference).  
* **ArchiveTable** – The physical table name that holds the archived data.

---

## Target Components  

No target component (e.g., a database output) is defined in the supplied metadata. Consequently, there are **no target schemas** or derived columns to document.  

*If additional target tables such as `fact_transactions` or `ctr_reporting_queue` exist in the broader solution, their definitions are **not specified in the source metadata** provided for this job.*

---

## Data Type Mapping  

| Talend Type (as shown) | Approximate SQL Server Type | Approximate PostgreSQL Type | Approximate Oracle Type |
|------------------------|----------------------------|-----------------------------|--------------------------|
| `id_String(50)`  | `VARCHAR(50)` | `VARCHAR(50)` | `VARCHAR2(50)` |
| `id_String(100)` | `VARCHAR(100)`| `VARCHAR(100)`| `VARCHAR2(100)`|

*All three columns are character strings; nullability follows the column definitions above.*

---

## Derived Column Definitions  

No derived columns are produced by this job because the tMap component contains **no expressions**. All output fields are direct copies (or simple case‑based transformations) of the input columns as described in the SQL query.

---

## Additional Notes for Analysts  

* The job’s sole purpose is to generate a **catalog** of historic tables, which can be used for downstream processes such as data archiving, reporting, or migration.  
* Because the output consists only of schema names and table identifiers, there is no transactional or customer‑level data exposed here.  
* If you require details about the **structure** of the individual archive tables (column definitions, data types, business meanings), those would need to be retrieved separately—this job does **not** capture that level of detail.  

---  

*End of documentation.*
