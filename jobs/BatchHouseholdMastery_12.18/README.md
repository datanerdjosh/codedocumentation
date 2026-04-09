# BatchHouseholdMastery

## Executive Summary
Mastery and Survivorship for Household

## Business Description
Loads Phones from HOUSEHOLD_IN_PROCESS and Performances the CDM Mastery And Survivorship rules on them and loads the newly mastered Household Groups in the Final HEOUSEHOLD Table for CDM

Mastery Based on Household Last Name, Address_Line_1, Address_Line_2, City, State, Zip

## Verified Metadata
| Attribute | Value |
|-----------|-------|
| Repository Job Name | BatchHouseholdMastery_12.18 |
| Display Label | BatchHouseholdMastery |
| Version | 12.18 |
| Status | DEV |
| Schedule | Not specified in source metadata. |
| Owner | Not specified in source metadata. |
| Talend Path | CDM/CDM/MasterySurvivorship/CDMBatch |

## Technical Overview

### Components Summary
| Type | Count | Components |
|------|-------|------------|
| Sources | 3 | tMSSqlInput, tMSSqlInput, tHashInput |
| Transforms | 19 | tJava, tJava, tJava, tJava, tJava, tJava, tMap, tJava, tJava, tJava, tJava, tJava, tJava, tJava, tMap, tMap, tJava, tJava, tJava |
| Targets | 2 | tHashOutput, tMSSqlOutput |

## Release History
- 12.18 - Make sure Salesforce Records match with the records that created them.
- 12.17 - Persist previous household ID.
- 12.15 - Friendly Household ID
- 12.14 - Salesforce Officer
- 12.10 - Make Persistent Household record but better, simpler, and cooler.
- 12.9 - Add Persistent Household Record. Make Master Household ID durable.
- 12.8 - Income and Investable Asset survivorship part 2
- 12.6 - Add Persona Survivoreship
- 12.5 - Ignore case on exact match
- 12.4 - Add Income and Investable Assets survivorship
- 12.3 - Add Salesforce HousholdRecords
- 12.2 - Add Partitioning

## Notes
This README is rendered deterministically from verified Talend metadata for non-JOB_ ETL assets. See `schema.md` and `data_flow.md` for parsed source and target details.
