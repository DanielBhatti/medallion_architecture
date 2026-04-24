# Bronze Layer (Data Lake, Data Ocean, Data Swamps)

## Overview

The Bronze layer stores raw, unprocessed data exactly as it arrives from source systems with minimal transformation.  Data in this area is primarily insert-only and highly unstructured.  The Bronze layer primarily stores data so that it may be inspected, audited, or transformed into the Silver layer.

## Purpose

- Single source for raw data - preserves the original data before any modifications
- Receives data from APIs, databases, files, log streams, and other sources
- Audit trail - maintains complete history for compliance and troubleshooting

## Implementation Details

### Schemas
- Create a schema representing where the source data comes from.  Prepend with stage_ or bronze_ (e.g. stage_hedgeserv).
- When mapping data, if feasible, rename field/column names to snake_case.
  - InvestorInternalId -> investor_internal_id
  - fundName -> fund_name
  - lmifee -> lmi_fee  
- If feasible, create columns with the expected data type (i.e. don't make everything string type)

### Ingest Data with Minimal Processing

- Extract data from API responses, database clones, files and map fields to table columns
- If data fails to load, alert any relevant stakeholders
- Add additional checks/notifications for if the data source adds new fields or changes existing ones
- If ingestion requires any pre-processing (e.g. extracting JSON), do it in-memory or within temporary stages/tables.

### Add Metadata for Tracking

Add metadata columns with every load where applicable.  Some examples include:

```
- insert_timestamp (same timestamp for all records in batch)
- source
- source_file (full path, directory + filename)
- status
- load_id/_batch_id (unique batch identifier)
- version
- request
- input
- app_commit_hash (identifying app version responsible for loading the data)
```

## Design Principles

### Immutability

Bronze data should never be updated without reason.  If corrections are needed or a new version is available, new data should be loaded without replacing the previous data.  This incentivizes us to correct data at the source (the data providers) rather than creating additional work for us to correct the data.

### Retention

Store all data received, even if quality issues are identified later.  This is important to maintain an audit trail, identify what was loaded and when, if there were any issues, and have a historical record of available data.

If storage becomes a concern, old records can be deleted according to either a general or source-specific policy.

### Security/Permissioning

Only people responsible for maintaining the ETL should have access to Bronze data.  Because data is not validated, it's dangerous to use in any business-facing reporting without some proper vetting.