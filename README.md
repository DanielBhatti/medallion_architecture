# Medallion Architecture

The Medallion Architecture is a data engineering standard that organizes data into three distinct layers: Bronze, Silver, and Gold. Each layer progressively refines and transforms raw data into a form amenable to business intelligence and analytics.

## Architecture Overview

```mermaid
graph LR
    A["Data Sources<br/>(APIs, Databases, Files)"] --> B["Bronze Layer<br/>(Raw Data)"]
    B --> C["Silver Layer<br/>(Normalized)"]
    C --> D["Gold Layer<br/>(Business Ready)"]
    D --> E["Analytics/BI<br/>(Reports, Dashboards)"]
    style B fill:#D4A574,stroke:#333,stroke-width:2px,color:#000
    style C fill:#C0C0C0,stroke:#333,stroke-width:2px,color:#000
    style D fill:#FFD700,stroke:#333,stroke-width:2px,color:#000
```

## Example:

```mermaid
graph LR
    SRC["Fund Admin API<br/>"]
    
    SRC --> ING["Staging<br/>"]
    ING --> B["BRONZE<br/>fund_info<br/>investor_info<br/>commitments<br/>capital_activities<br/>rollforward"]
    
    B --> CLEAN["Validation<br/>Reconciliation<br/>Deduplication<br/>Normalization"]
    CLEAN --> S["SILVER<br/>fund<br/>investor<br/>commitment<br/>capital_activity<br/>rollforward<br/>fund_category"]
    
    S --> AGG["Calculations<br/>Transformations<br/>Business Rules<br/>Joins<br/>Star Schema"]
    AGG --> G["GOLD<br/>fund_metrics<br/>investor_portfolio<br/>fund_performance<br/>allocation_summary"]
    
    G --> USE["Usage<br/>(Reporting, Analytics,<br/>Risk Management)"]
    
    style B fill:#D4A574,stroke:#333,stroke-width:2px,color:#000
    style S fill:#C0C0C0,stroke:#333,stroke-width:2px,color:#000
    style G fill:#FFD700,stroke:#333,stroke-width:2px,color:#000
```

## Documentation

- [Bronze Layer](layers/bronze.md) - Raw data landing zone, staging area
- [Silver Layer](layers/silver.md) - Cleaned and normalized, 
- [Gold Layer](layers/gold.md) - Source of truth for any reporting
- [Bronze → Silver Transition](transitions/bronze-to-silver.md) - Data quality and cleaning
- [Silver → Gold Transition](transitions/silver-to-gold.md) - Aggregation and business logic
- [Best Practices](best-practices.md) - Governance and optimization
- [Technology Stack](technology-stack.md) - Tools and platforms