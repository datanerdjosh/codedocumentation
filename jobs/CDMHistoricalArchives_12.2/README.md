# CDMHistoricalArchives

## Executive Summary
Archive Current Day Tables into Monthly Snapshot Tables

## Business Description
12.2 Fix Success Failed notifications to say which table.

12.1 Archive Daily Also

For each table in a CDM HIST* schema, Delete data older than 6 months, and Load the current Month End from the corresponding current day table.

## Verified Metadata
| Attribute | Value |
|-----------|-------|
| Repository Job Name | CDMHistoricalArchives_12.2 |
| Display Label | CDMHistoricalArchives |
| Version | 12.2 |
| Status | DEV |
| Schedule | Not specified in source metadata. |
| Owner | Not specified in source metadata. |
| Talend Path | CDM |

## Technical Overview

### Components Summary
| Type | Count | Components |
|------|-------|------------|
| Sources | 2 | tMSSqlInput, tFixedFlowInput |
| Transforms | 10 | tJava, tJava, tJava, tJava, tJava, tJava, tJava, tJava, tJava, tJava |
| Targets | 0 | None identified |

## Release History
Not specified in source metadata.

## Notes
This README is rendered deterministically from verified Talend metadata for non-JOB_ ETL assets. See `schema.md` and `data_flow.md` for parsed source and target details.
