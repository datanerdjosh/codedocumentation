# Business Glossary: CDMHistoricalArchives

## What This Job Does (Plain English)
Archive Current Day Tables into Monthly Snapshot Tables

## Business Description
12.2 Fix Success Failed notifications to say which table.

12.1 Archive Daily Also

For each table in a CDM HIST* schema, Delete data older than 6 months, and Load the current Month End from the corresponding current day table.

## Important Terms
- **tMSSqlInput**: A Talend source component used to read data from Microsoft SQL Server tables or queries.
- **tJava**: A Talend component used to run custom Java logic inside the ETL flow.

## Frequently Asked Questions

### What does this job do?
Archive Current Day Tables into Monthly Snapshot Tables

### When does it run?
Not specified in source metadata.

### Who owns it?
Not specified in source metadata.

### What are the main inputs and outputs?
Inputs are represented by: tMSSqlInput, tFixedFlowInput

Outputs are represented by: Not specified in source metadata.

## Known Metadata
| Attribute | Value |
|-----------|-------|
| Repository Job Name | CDMHistoricalArchives_12.2 |
| Display Label | CDMHistoricalArchives |
| Talend Path | CDM |

## Notes
This glossary is rendered deterministically from verified Talend metadata for non-JOB_ ETL assets.
