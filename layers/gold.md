# Gold Layer (Business Ready)

## Overview

The Gold layer contains aggregated, business-aligned data optimized for analytics, reporting, and machine learning. This is what business users and BI tools consume directly.

## Purpose

- Business alignment — organized around business entities and metrics
- Performance optimization — pre-calculated aggregations and denormalization
- Analytics readiness — directly feeds dashboards, reports, and ML models
- User accessibility — intuitive schema matching business terminology
- Multiple marts — supports different analytical perspectives (sales, marketing, finance)

## Why it's Important

- Accelerates analytics and reduces query time
- Breaks down silos between business units
- Enables self-service analytics
- Reduces query burden on source systems
- Improves BI tool performance

## How to Achieve It

### 1. Create Business-Focused Tables

Design tables with business questions in mind:

```
- Fact tables: transactions, events, orders, pageviews
- Dimension tables: customers, products, dates, geographies
- Aggregate tables: daily revenue, monthly sales, customer cohorts
- Summary tables: KPI dashboards, executive metrics
```

### 2. Optimize for Analytics

Structure data for efficient querying:

```
- Denormalize related fields (avoid multiple joins)
- Pre-calculate common metrics (sums, weighted averages, counts)
- Create summary tables for common queries
- Add calculated columns for frequent computations
```

### 3. Apply Business Logic

Encode business rules directly:

```
- Calculate KPIs and metrics (LTV, CAC, churn rate)
- Apply business rules and definitions (fiscal calendar, GL coding)
- Create business segments and cohorts (VIP, high-value, at-risk)
- Implement complex hierarchies (organization structure)
```

### 4. Example Gold Table Structures

Customer metrics (fact table):

```
gold.customer_monthly_metrics
├── customer_id (INT) [Key]
├── metric_month (DATE) [Period]
├── total_revenue (DECIMAL) [Metric]
├── order_count (INT) [Metric]
├── avg_order_value (DECIMAL) [Metric]
├── repeat_customer (BOOLEAN) [Dimension]
├── customer_segment (VARCHAR) [Business classification]
├── churn_probability (DECIMAL) [ML score]
└── last_updated (TIMESTAMP)
```

Product performance (aggregate):

```
gold.product_daily_sales
├── product_id (INT) [Key]
├── sale_date (DATE) [Period]
├── category (VARCHAR) [Product dimension]
├── sales_amount (DECIMAL) [Metric]
├── units_sold (INT) [Metric]
├── rank_by_category (INT) [Derived ranking]
├── yoy_growth (DECIMAL) [YoY comparison]
└── created_at (TIMESTAMP)
```

## Design Patterns

### Star Schema

Connect dimension tables to fact tables:

```
FACT: orders
├── FK: customer_id → DIM: customers
├── FK: product_id → DIM: products
├── FK: date_id → DIM: dates
└── FK: region_id → DIM: regions
```

Benefits:
- Efficient joins
- Easy to understand for end users
- Optimized for BI tools

### Slowly Changing Dimensions (SCD)

For dimensions like products with changing attributes:

**Type 0:** Retain original (rarely used)

**Type 1:** Overwrite old values
```
update product_dim set price = $29 where product_id = 123
```

**Type 2:** Maintain history
```
└── Add effective_date, end_date, is_current columns
```

**Type 3:** Limited history
```
└── Add current and previous value columns
```

### Conformed Dimensions

Reuse dimensions across fact tables for consistency:

```
dim_customer (shared)
├── Used by fact_orders
├── Used by fact_payments
└── Used by fact_supports
```

### Factless Fact Tables

Tables of dimension key combinations for analysis:

```
gold.website_events
├── user_id (FK)
├── session_id (FK)
├── device_id (FK)
├── event_date (FK)
└── event_count (1 per row)
```

## Common Gold Layer Objects

### Mart Tables

One per business area:

- Sales Mart: customers, products, orders, revenue
- Marketing Mart: campaigns, channels, conversions, attribution
- Finance Mart: GL codes, transactions, journal entries
- Operations Mart: incidents, resolutions, SLAs

### Aggregate Tables

Pre-computed summaries:

```
gold.customer_lifetime_value
├── customer_id
├── total_revenue_usd
├── transaction_count
├── avg_transaction_value
└── days_as_customer
```

### Role-Based Aggregations

Different perspectives for different users:

```
Finance view: revenue, COGS, gross margin
Sales view: pipeline, close rate, average deal size
Operations view: cycle time, quality metrics, utilization
```

### Temporal Tables

Track metrics over time:

```
gold.monthly_cohort_retention
├── cohort_month
├── age_in_months
├── retention_rate
├── count_users
└── revenue_per_user
```

## Implementation Approaches

### Incremental Processing

Load only changed data for efficiency:

```sql
INSERT INTO gold.customer_metrics
SELECT ...
FROM silver.orders O
WHERE O._processed_at >= (
  SELECT MAX(_processed_at) FROM gold.customer_metrics
)
```

### Full Replace

Simpler, used for lower-volume tables:

```sql
TRUNCATE gold.product_daily_sales;
INSERT INTO gold.product_daily_sales
SELECT ... FROM silver.orders;
```

### Materialized Views

Automatic refresh on source table changes (if supported):

```sql
CREATE MATERIALIZED VIEW gold.revenue_by_region AS
SELECT region, DATE(order_date), SUM(amount)
FROM silver.orders
GROUP BY region, DATE(order_date);
```

### Streaming Aggregations

Real-time updates for fast-changing metrics:

```
Kafka stream → Spark Structured Streaming → Delta Table → BI Tool
```

## Performance Optimization

### Indexing

For faster lookups (database tables):

```
- Cluster index on frequently filtered columns
- Non-clustered indexes on join keys
```

### Partitioning

```
- Partition by date for time-series data
- Partition by region for geographic filtering
```

### Columnar Storage

- Use Parquet with projection pushdown
- Consider ORC for compression vs. query speed tradeoffs

### Statistics

Enable query optimizer to make good decisions:

```sql
ANALYZE TABLE gold.customer_metrics COMPUTE STATISTICS;
```

### Caching

Pre-load frequently used tables into memory.

## Business Logic Examples

### Customer Segmentation

```sql
CASE
  WHEN ltv > (SELECT PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY ltv) 
              FROM gold.customer_ltv)
    THEN 'VIP'
  WHEN ltv > (SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY ltv) 
              FROM gold.customer_ltv)
    THEN 'High Value'
  WHEN recent_activity_days > 90
    THEN 'At Risk'
  ELSE 'Standard'
END as customer_segment
```

### YoY Comparison

```sql
SELECT
  product_id,
  current_month,
  current_revenue,
  LAG(current_revenue, 12) OVER (
    PARTITION BY product_id
    ORDER BY current_month
  ) as prior_year_revenue,
  ROUND(
    (current_revenue / LAG(current_revenue, 12) OVER (...) - 1) * 100, 2
  ) as yoy_growth_pct
FROM gold.product_monthly_sales
```

### Fiscal Calendar Alignment

```sql
CASE MONTH(order_date)
  WHEN 1,2,3 THEN YEAR(order_date)
  WHEN 4,5,6 THEN YEAR(order_date)
  WHEN 7,8,9 THEN YEAR(order_date)
  WHEN 10,11,12 THEN YEAR(order_date) + 1
END as fiscal_year
```

## Tools and Technologies

### dbt

Gold layer models:

```yaml
# fact_orders.sql
{{ config(
  materialized='table',
  partition_by={
    "field": "order_date",
    "data_type": "date",
    "granularity": "month"
  }
) }}

SELECT ...
```

### Business Intelligence Tools

- Tableau, Power BI, Looker
- Query directly from Gold tables
- Semantic layers for user-friendly definitions

### Data Warehouses

- Snowflake
- BigQuery
- Redshift
- Azure Synapse

### Streaming Platforms

- Kafka + Spark
- Apache Flink
- Cloud Dataflow

## Governance

- Document business definitions for all metrics
- Establish ownership (Finance owns LTV, Sales owns pipeline)
- Implement access controls (row-level security)
- Monitor data freshness SLAs
- Track user queries and usage patterns

## Monitoring

Track usage metrics:

- Query count and latency
- Data freshness (age of latest refresh)
- Row counts and growth trends
- User engagement and adoption
- Failed queries or data quality issues
