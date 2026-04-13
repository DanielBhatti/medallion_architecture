# Medallion Architecture

The Medallion Architecture is a data engineering standard that organizes data into three distinct layers: Bronze, Silver, and Gold. Each layer progressively refines and transforms raw data into a form amenable to business intelligence and analytics.

## Architecture Overview

```mermaid
graph LR
    DS["Data Sources<br/>(APIs, Databases, Files)"] --> B["Bronze Layer<br/>(Data Lake)"]
    B --Clean<br>Validate--> S["Silver Layer<br/>(Normalized)"]
    S --Denormalize<br>Model<br>Materialize--> G["Gold Layer<br/>(Star Schema)"]
    S --Display missing data/gaps<br>--> U["GUI"]
    U --User-provided data changes--> S
    G --> A["Analytics/BI<br/>(Reports, Dashboards)"]
    style B fill:#D4A574,stroke:#333,stroke-width:2px,color:#000
    style S fill:#C0C0C0,stroke:#333,stroke-width:2px,color:#000
    style G fill:#FFD700,stroke:#333,stroke-width:2px,color:#000
```


## Documentation

- [Bronze Layer](layers/bronze.md)
  - Save external data raw with minimal transformation
  - Generally insert-only, no modifying the data, allow processing data multiple times
  - Save metadata about where data came from, when it arrived, what inputs were used for obtaining it, etc.
  - Perform validations to ensure data is self-consistent
- [Bronze -> Silver Transition](transitions/bronze-to-silver.md)
  - Deduplication and cleanup
  - Assign unique identifiers
  - Map or create new internal identifiers
  - Transformation logic (preferably in-memory) into normalized tables
- [Silver Layer](layers/silver.md)
  - Normalized tables (in Snowflake, use hybrid tables)
  - Ensure uniqueness and referential integrity at all times
  - Address any business-level check issues here
- [Silver -> Gold Transition](transitions/silver-to-gold.md)
  - Denormalize into data models useful to the business
  - Transform data and materialize them into tables
  - Perform groupings, aggregations, pre-compute things, etc.
- [Gold Layer](layers/gold.md)
  - Source of truth - never allow incorrect data to exist here
  - Few tables with large number of columns