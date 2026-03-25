# Silver -> Gold Transition: Star Schema Data Warehosue

## Overview

The star schema is a common standard for creating a data warehouse.  The key idea is that there will be many central fact tables surrounded by dimension tables. The name comes from it creating a "star" with the fact table connected to the dimension tables.  This optimizes query performance by materializing the data and simplifies analytics by intentfully denormalizing data from the Silver layer.

## Star Schema Structure

### Fact Table (Center)
Contains:
- Foreign keys to dimension tables
- Measures (metrics/quantities): counts, amounts, balances
- Dates

### Dimension Tables (Points)
Each contains:
- Business key as primary key (e.g., fund_id, investor_id)
- Attributes that describe the entity
- Change tracking (if needed: effective_date, end_date)

## Implementation Steps

### 1. Design Dimensions
Create separate dimension tables from Silver data using business keys as primary keys:
- `dim_fund`: fund_id (PK), fund_name, strategy, inception_date, currency
- `dim_investor`: investor_id (PK), investor_name, investor_type, geography
- `dim_date`: date (PK), year, quarter, month, fiscal_period
- `dim_capital_activity`: activity_type (PK), activity_type_name, category

### Build Fact Table
Create central fact table with measures and foreign keys:
- `fact_capital_activity`: fact_id, fund_id, investor_id, date_id, activity_type_id, amount, units, balance
- Foreign keys point to dimension surrogate keys
- Measures contain amounts and counts

### Populate With Data
Determine what the best way to populate the tables are:
- Truncate and load (best for small sizes)
- Incremental (choose some key field to determine when to load - usualy date-driven)
- Add computed columns
- Perform aggregations and add measures (e.g. total_nav, change_in_nav)
- Ignore metadata fields (e.g. inserted_date)
- Can be as simple as creating a view and materializing into the relevant table

## Example: Fund Management Star Schema

### Dimension Tables

**dim_fund**
- fund_id (primary key, business key)
- fund_name, strategy, base_currency
- inception_date, manager_id

**dim_investor**
- investor_id (primary key, business key)
- investor_name, investor_type, geography

**dim_date**
- date (primary key)
- year, quarter, month, fiscal_period

**dim_activity_type**
- activity_type (primary key)
- activity_type_name, category

### Fact Table

**fact_capital_activity**
```
- fund_id_fk → dim_fund.fund_id
- investor_id_fk → dim_investor.investor_id
- activity_date_fk → dim_date.date
- activity_type_fk → dim_activity_type.activity_type
- amount (measure)
- beginning_balance (measure)
- ending_balance (measure)
```

Grain: One row per fund-investor-activity-date combination
