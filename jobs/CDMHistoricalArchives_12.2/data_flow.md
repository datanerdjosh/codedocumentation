# Data Flow: CDMHistoricalArchives  

## Visual Flow Diagram  
```
+-------------------+          +----------------+          +-------------------+
|   tMSSqlInput     |--------->| tDBConnection  |--------->|   tDBInput        |
| (SQL extraction) |          |   (to DB)      |          | (reads tables)    |
+-------------------+          +----------------+          +-------------------+
                                   |                               |
                                   v                               v
                           +----------------+               +------------------+
                           | tFlowToIterate |-------------->| tFixedFlowInput  |
                           +----------------+               +------------------+
                                   |                               |
                                   v                               v
                           +----------------+               +------------------+
                           | tDBConnection  |------------->| tDBSP (stored   |
                           |   (per iter.)  |              | procedure)       |
                           +----------------+               +------------------+

Other control components (tJava, tSetGlobalVar, tDie, etc.) are linked via
trigger and run‑if connections as shown in the component connection list.
```

*The diagram reflects the actual components and their connections as
described in the source metadata. No Oracle or PostgreSQL tables, tMap
transformations, or lookup joins are defined in the provided job.*

## Source Extraction  

| Source Component | SQL Query (as defined) | Extracted Columns | Filter Conditions & Business Meaning |
|------------------|-----------------------|-------------------|---------------------------------------|
| **tMSSqlInput** | ```sql SELECT CASE [TABLE_SCHEMA] WHEN 'HIST' THEN 'dbo' ELSE REPLACE(TABLE_SCHEMA, 'HIST_', '') END AS SourceSchema , [TABLE_SCHEMA] AS TargetSchema , [TABLE_NAME] AS ArchiveTable FROM [INFORMATION_SCHEMA].[TABLES] WHERE TABLE_TYPE = 'BASE TABLE' AND TABLE_SCHEMA LIKE 'HIST%'; ``` | `SourceSchema`, `TargetSchema`, `ArchiveTable` | *Only base tables whose schema name starts with “HIST” are selected. This isolates historical tables for further processing.* |
| **tFixedFlowInput** | Not specified in source metadata | Not specified in source metadata | Not specified in source metadata |

## Transformation Logic  

- **tMap Component**: No tMap component or expressions are present in the supplied metadata (`tMap Transformation Expressions: No tMap expressions found`). Consequently, there are no calculated fields, derivations, or output mappings to document.  

- **Other Transformations**: Control flow components (`tJava`, `tSetGlobalVar`, etc.) manage job execution logic but do not perform data transformations on record sets.

## Data Lineage  

Because no explicit mapping component (e.g., tMap, tJoin) is defined, a detailed column‑level lineage cannot be derived from the available metadata.

| Source Table / Component | Source Column | Transformation | Target Table / Component | Target Column |
|--------------------------|---------------|----------------|--------------------------|---------------|
| Not specified in source metadata | Not specified | Not specified | Not specified | Not specified |

## Aggregation Logic  

No aggregation component (such as `tAggregateRow`) appears in the component list; therefore, no grouping or aggregation logic is defined for this job.  

---  

*All information presented above is extracted directly from the provided job metadata. Any element not explicitly described in the source data is indicated as “Not specified in source metadata.”*
