# Silver Layer (Normalized Database)

## Overview

The Silver layer ensures data integrity for data coming from the Bronze layer. Data is validated and trustworthy but not in the most convenient format for analytics.

## Purpose

- Data quality enforcement - remove duplicates, handle nulls, validate formats, apply common-sense checks
- Standardization - apply consistent conventions for data types, formats, and naming fields
- Deduplication - identify and remove duplicate records
- Non-nullability - fields that are required should always have a value
- Referential integrity - if FieldA relies on FieldB, FieldB must exist with a proper value 

## Implementation Details

### Idempotency

Transformations from Bronze to Silver should be idempotent.  If Bronze data is unchanged, running the same process multiple times should create the same data in the Silver layer.

### Traceability

Add fields as applicable so that source data from bronze can be identified.

### Incrementality

Data from Bronze to Silver should be loaded incrementally.

## Database Normalization

Database Normalization is the process of transforming data into what's called "normalized form".  At a high-level, we take raw data and decompose data into the lowest form possible.  A table either consists of one dimension of the data or otherwise relates tables together.

### First Normal Form (1NF)

**High Level:** Every table cell should contain a single value, and there should be no repeating groups or arrays in columns. Basically, make sure each column has atomic (indivisible) values.

**Formal Definition:** A relation R is in first normal form (1NF) if and only if all underlying domains contain atomic values only, i.e., the value of each attribute contains only a single value from that domain.

**Bad Example:**

A fund commitment table storing multiple commitments in one row:
- `fund_id`, `investor_id`, `commitment_1_amount`, `commitment_1_date`, `commitment_2_amount`, `commitment_2_date`

**Good Example:**

Separate atomic rows for each commitment:
- `fund_id`, `investor_id`, `commitment_amount`, `commitment_date`

### Second Normal Form (2NF)

**High Level:** The table must be in 1NF, and every non-key attribute must depend on the entire primary key, not just part of it. This eliminates partial dependencies.

**Formal Definition:** A relation R is in second normal form (2NF) if it is in 1NF and every non-prime attribute is fully functionally dependent on the primary key.

**Bad Example:**

A `fund_investor_activity` table with composite key (`fund_id`, `investor_id`, `activity_date`) including attributes that depend only on part of the key:
- `fund_id`, `investor_id`, `activity_date`, `activity_type`, `amount`, `currency`, `fund_name`, `investor_name`

(`fund_name` depends only on `fund_id`, `investor_name` depends only on `investor_id`)

**Good Example:**

Split into normalized tables:
- `fund_investor_activity(fund_id, investor_id, activity_date, activity_type, amount, currency)`
- `funds(fund_id, fund_name, strategy)`
- `investors(investor_id, investor_name, investor_type)`

### Third Normal Form (3NF)

**High Level:** The table must be in 2NF, and no non-key attribute depends on another non-key attribute. This removes transitive dependencies.

**Formal Definition:** A relation R is in third normal form (3NF) if it is in 2NF and every non-prime attribute is non-transitively dependent on the primary key.

**Bad Example:**

A `capital_commitments` table with transitive dependency:
- `fund_id`, `investor_id`, `commitment_amount`, `commitment_date`, `fund_currency`

(`fund_currency` depends on `fund_id`, which is not the primary key)

**Good Example:**

Remove transitive dependency by moving to separate table:
- `capital_commitments(fund_id, investor_id, commitment_amount, commitment_date)`
- `funds(fund_id, fund_name, fund_currency)`

### Fourth Normal Form (4NF)

**High Level:** The table must be in 3NF and eliminate multi-valued dependencies. One entity should not have two or more independent one-to-many relationships in the same table.

**Formal Definition:** A relation R is in fourth normal form (4NF) if it is in Boyce-Codd normal form (BCNF) and has no non-trivial multi-valued dependencies other than a candidate key.

**Bad Example:**

A `fund_management` table mixing independent multi-valued relationships:
- `fund_id`, `investor_id`, `commitment_id`, `rollforward_date`, `capital_call_id`, `distribution_id`

(This combines commitments, rollforwards, and capital activities in one table)

**Good Example:**

Split into separate relationships:
- `fund_commitments(fund_id, investor_id, commitment_id, commitment_amount, commitment_date)`
- `capital_rollforwards(fund_id, rollforward_date, beginning_balance, contributions, distributions, ending_balance)`
- `capital_activities(fund_id, activity_id, activity_type, amount, activity_date)`

### Why Normalization in Silver Layer?

Normalization in the Silver layer helps create clean, consistent schemas that prevent anomalies and ensure data quality. However, for analytical workloads, some denormalization may be applied in Gold for performance.

## Schema Evolution

As data sources and business requirements change, Silver layer schemas must evolve while maintaining backward compatibility.

### Strategies

- **Additive Changes:** Add new columns without removing existing ones.
- **Versioned Schemas:** Maintain multiple versions of tables for different time periods.
- **Deprecation:** Mark old columns as deprecated before removal.
- **Migration Scripts:** Use automated scripts to transform data when schema changes.

### Best Practices

- Document all schema changes in a changelog.
- Test schema evolution with historical data.
- Use tools like dbt's schema evolution features.

## Data Lineage

Data lineage tracks the flow and transformation of data from Bronze to Silver, ensuring transparency and auditability.

### Importance

- Enables debugging of data issues.
- Supports compliance and governance.
- Helps understand data dependencies.

### Implementation

- Use surrogate keys to link records across layers.
- Maintain transformation logs with timestamps.
- Leverage tools like Apache Atlas or dbt's lineage features.

## Common Patterns

### SCD Type 2 (Slowly Changing Dimensions)

For dimensions like customers, track changes over time:

```
- Add effective_date and end_date columns
- Mark current records with is_current = true
- Maintain history of all changes
```

### Deduplication Strategy

Choose deduplication approach based on your domain:

```
- Most recent by timestamp
- Most complete row (fewest nulls)
- Source system priority (if multiple sources)
```

### Data Quality Scoring

Add a quality score reflecting null percentages and validation passes:

```sql
CASE
  WHEN validation_checks_passed AND null_count = 0 THEN 1.0
  WHEN validation_checks_passed THEN 0.8
  ELSE 0.5
END AS quality_score
```

### Type 1 Null Handling

For different null meanings, use categoricals:

```
- NULL → not provided
- 'UNKNOWN' → unknown
- 'N/A' → not applicable
- 'PENDING' → processing
```

## Tools and Technologies

### SQL

Basic building blocks:

```sql
SELECT
  CAST(customer_id AS INT) as customer_id,
  TRIM(UPPER(name)) as customer_name,
  LOWER(TRIM(email)) as email,
  COALESCE(signup_date, '1900-01-01') as signup_date,
  CASE
    WHEN status = 'ACTIVE' THEN TRUE
    ELSE FALSE
  END as is_active
FROM bronze.customers_raw
WHERE _ingested_at = (SELECT MAX(_ingested_at) FROM bronze.customers_raw)
  AND TRIM(customer_id) != ''
```

### dbt

Use dbt models for reproducible transformations:

- ref() for lineage tracking
- tests for data quality validation
- macros for reusable logic
- incremental models for performance

### Apache Spark / PySpark

DataFrame API for programmatic transformations:

```python
from pyspark.sql.functions import col, trim, lower, coalesce

silver_df = (bronze_df
  .select(
    col("customer_id").cast("INT"),
    trim(col("name")).alias("customer_name"),
    lower(trim(col("email"))).alias("email")
  )
  .dropDuplicates(["customer_id"])
  .filter(col("customer_id").isNotNull())
)
```

### Python (Pandas/Polars)

For smaller datasets or complex logic:

- Data validation with Great Expectations
- Complex business rules with custom logic
- Incremental updates

## Quality Gates

Implement automated checks before data reaches downstream users:

```
- NOT NULL constraints on critical columns
- Referential integrity checks
- Uniqueness constraints on business keys
- Value range validations
- Format compliance (email, phone)
- Completeness SLAs (% non-null)
- Recency validation (data not stale)
```

## Partitioning Strategy

Organize Silver data efficiently:

```
silver/customers_cleaned
├── year=2024
│   ├── month=01
│   ├── month=02
│   └── ...
```

This enables efficient filtering and parallel processing on date ranges.

## Storage Optimization

- Use Parquet for efficiency
- Consider Delta Lake for ACID compliance
- Compress with Snappy or GZIP
- Cluster by frequently filtered columns

## Monitoring

Track quality metrics:

- Percentage complete (non-nulls) per column
- Duplicate detection rate
- Failed quality checks per table
- Processing time and latency
- Row count changes from Bronze
- Data freshness
