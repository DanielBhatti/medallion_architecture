# Bronze Layer (Data Lake, Data Ocean, Data Swamps)

## Overview

The Bronze layer stores raw, unprocessed data exactly as it arrives from source systems with minimal transformation.  Data in this area is primarily insert-only and highly unstructured.  The Bronze layer primarily stores data so that it may be inspected, audited, or transformed into the Silver layer.

## Purpose

- Single source for raw data — preserves the original data before any modifications
- Data ingestion hub — receives data from APIs, databases, files, log streams, and other sources
- Audit trail — maintains complete history for compliance and troubleshooting
- Baseline for recovery — allows the ability to re-process end-to-end if a downstream transformations fail

## Implementation Details

### Ingest Data with Minimal Processing

```
- Copy files as-is to Bronze layer
- Copy database records from other databases
- Extract API responses with minimal transformation (e.g. JSON -> table)
- If desired, can preserve original data types and formats
```

### Add Metadata for Tracking

Include metadata columns with every load where applicable.  Some examples include:

```
- _insert_timestamp
- _source
- _source_file
- _status
- _load_id (unique batch identifier)
- _version
- _request
- _input
```


## Design Principles

### Immutability

Bronze data should never be updated without reason. If corrections are needed or a new version is available, new data should be loaded alongside previous data. This maintains an audit trail and enables reproducibility.

### Retention

Store all data received, even if quality issues are identified later.

### Minimal Transformation

Apply only what's necessary to land the data into the database:
- Deserialization (JSON -> table columns)
- Basic data type casting if possible
- No business logic
- No derived fields
- No deduplication

### Ingestion

Prioritize ingesting fast and don't be concerned about data quality.  Excess data can always be removed later and validations are easier to perform after the data has landed into the database.