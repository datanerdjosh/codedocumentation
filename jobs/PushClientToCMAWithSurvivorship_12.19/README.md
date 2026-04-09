# PushClientToCMAWithSurvivorship

## Executive Summary
Pushes Clients to CMA

## Business Description
-Change stage to Phase

-Add Acquisition/Attrition Date Persisting

Queries [dbo].[CLIENT] in CDM for match groups with single record pushes as is for match groups with multiple records performs survivorship

## Verified Metadata
| Attribute | Value |
|-----------|-------|
| Repository Job Name | PushClientToCMAWithSurvivorship_12.19 |
| Display Label | PushClientToCMAWithSurvivorship |
| Version | 12.19 |
| Status | DEV |
| Schedule | Not specified in source metadata. |
| Owner | Not specified in source metadata. |
| Talend Path | CDM/CMA/PushToCMA |

## Technical Overview

### Components Summary
| Type | Count | Components |
|------|-------|------------|
| Sources | 4 | tMSSqlInput, tMSSqlInput, tMSSqlInput, tHashInput |
| Transforms | 21 | tJava, tJava, tJava, tJava, tJava, tJava, tJava, tMap, tJava, tJava, tJava, tJava, tJava, tJava, tJava, tMap, tMap, tJava, tJava, tJava, tJava |
| Targets | 3 | tMSSqlOutput, tHashOutput, tMSSqlOutput |

## Release History
- 12.19 -  Trust Household
- 12.18 - Survive Last Name over blank.
- 12.17 - Credit Sharable Date all null?
- 12.16 - Fix Account Status on singleton clients
- 12.15 - SHARE_INFO_MAINT_DT to Client_Information_Sharable_Date
- 12.12 - Durable Client ID
- 12.11 - Incorporation Year
- 12.10 - Look in Account for Primary Client ID
- 12.7 - Consider Deleted records in survivorship.
- 12.6 - Add SWP
- 12.5 - Add Client And Credit information share dates
- 12.4 - Map PersonContactID to Salesforce_Contact_ID
- 12.2 - Partitioning Attempt
- 12.1 - Add Business Relationship Groups to Survivorship

## Notes
This README is rendered deterministically from verified Talend metadata for non-JOB_ ETL assets. See `schema.md` and `data_flow.md` for parsed source and target details.
