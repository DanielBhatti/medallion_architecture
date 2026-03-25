# Best Practices

## Architectural Principles

### Immutability

Bronze and Silver data should be immutable. Instead of updating records:

- Create new versions of tables with updated timestamps
- Use partitioning to manage historical data
- Maintain full audit trail of all changes
- Enable safe re-runs without side effects

Example:
```sql
-- Instead of UPDATE
UPDATE silver.customers SET status = 'ACTIVE' WHERE ...;

-- Do this
INSERT INTO silver.customers (version_date, is_current, ...)
SELECT CURRENT_DATE as version_date, TRUE as is_current, ...
WHERE ... AND is_current = FALSE;

-- Mark old version as stale
UPDATE silver.customers SET is_current = FALSE
WHERE id IN (...)
  AND version_date < CURRENT_DATE;
```

### Lineage Tracking

Maintain column-level lineage:

- Document source column → transformation → target column
- Use data catalog tools (Apache Atlas, Collibra, Alation)
- Track dependencies between tables
- Enable impact analysis for schema changes

dbt example:
```yaml
version: 2
models:
  - name: customers_cleaned
    description: Cleaned customer data from multiple sources
    columns:
      - name: customer_id
        description: Unique customer identifier
        tests:
          - unique
          - not_null
          - relationships:
              to: ref('orders')
              field: customer_id
```

### Schema Versioning

Manage schema changes systematically:

- Add new columns, don't remove (archive separately)
- Maintain backward compatibility in views
- Version your schema in Git
- Document breaking changes

Pattern:
```sql
-- Good: Add new column
ALTER TABLE silver.customers ADD COLUMN email_verified BOOLEAN DEFAULT FALSE;

-- Avoid: Drop column
ALTER TABLE silver.customers DROP COLUMN legacy_field;

-- Instead: Archive
INSERT INTO silver.customers_archive SELECT * FROM silver.customers;
```

### Quality Gates

Implement automated tests at each layer:

```
Bronze:
  - File integrity (checksum validation)
  - Row count expectations
  - Schema presence

Silver:
  - NOT NULL on critical fields
  - Uniqueness constraints
  - Format validation
  - Referential integrity

Gold:
  - Metric reasonableness checks
  - YoY/MoM sanity checks
  - Completeness (no missing combinations)
```

dbt tests:
```yaml
tests:
  - unique:
      column_name: customer_id
  - not_null:
      column_name: [customer_id, email]
  - relationships:
      column_name: country_code
      to: ref('reference_countries')
  - accepted_values:
      column_name: status
      values: ['ACTIVE', 'INACTIVE', 'SUSPENDED']
```

### Documentation

Clear documentation reduces cognitive load:

- Define business metrics (LTV, CAC, Churn)
- Document transformation logic
- Maintain data dictionaries
- Record owner per dataset
- Link to upstream and downstream consumers

Example:
```yaml
models:
  - name: customer_lifetime_value
    meta:
      owner: analytics-team
      sla: T+1 by 6am UTC
      refresh_frequency: daily
    description: Total revenue per customer across all time
    columns:
      - name: customer_id
        description: Unique customer identifier
        meta:
          pii: true
      - name: ltv_usd
        description: Lifetime revenue in USD, excluding refunds
        meta:
          transform: SUM(order_amount WHERE status IN ('completed', 'shipped'))
```

### Monitoring

Track pipeline health continuously:

- Data freshness (age of latest refresh)
- Row count changes (anomaly detection)
- Pipeline runtime (trend analysis)
- Failed runs (alert threshold)
- Data quality metrics
- Query latency

Example alerts:
```
- Silver table has 0 rows (typical: 1M) → CRITICAL
- Gold refresh took 2 hours (typical: 30 min) → WARNING
- Quality check failure rate > 5% → WARNING
- Data not updated in 24 hours → CRITICAL
```

## Governance Framework

### Data Classification

Categorize data by sensitivity:

```
PUBLIC: No restrictions (product catalog)
INTERNAL: Employees only (operational metrics)
CONFIDENTIAL: Limited access (financial data)
RESTRICTED: Regulatory compliance required (PII, PHI)
```

### Access Control

Implement role-based access (RBAC):

```
Bronze: Data engineers only
  ├─ Raw data access
  └─ Troubleshooting queries

Silver: Analytics team + Data engineers
  ├─ Trusted cleaned data
  └─ Reference data

Gold: All authorized users
  ├─ Self-service dashboards
  ├─ ML model training
  └─ Ad-hoc reporting
```

### PII Handling

For sensitive data, implement:

- Encryption at rest and in transit
- Tokenization or hashing
- Row-level security (RLS)
- Audit logging of access
- Data retention policies

Pattern:
```sql
-- Mask PII in lower environments
CASE
  WHEN environment = 'production' THEN email
  ELSE SUBSTR(email, 1, 3) || '****@' || SUBSTR(email, INSTR(email, '@')+1)
END as email
```

### Data Catalog

Maintain centralized metadata:

- Table ownership and contact info
- Business definition and valid values
- Data quality metrics
- Usage statistics
- Lineage diagrams
- Data classification level

Tools: Apache Atlas, Collibra, Alation, Datahub

## Performance Optimization

### Partitioning Strategy

Organize data for efficient access:

Partition by:
- Date (most common) — enables time-based queries
- Region — supports geographic filtering
- Source system — isolates quality issues
- Tenant/Customer — enables multi-tenant isolation

```
gold/monthly_sales/
├── year=2024
│   ├── month=01
│   ├── month=02
│   └── month=03
└── year=2023
    └── month=12
```

Benefits:
- Eliminate full table scans
- Parallel processing by partition
- Efficient incremental updates
- Simple data pruning

### Indexing (Database Only)

Create indexes for frequent queries:

```sql
-- Cluster index on join key
CREATE CLUSTERED INDEX idx_customer_id
ON gold.customer_metrics (customer_id);

-- Non-clustered on filter columns
CREATE NONCLUSTERED INDEX idx_date
ON gold.customer_metrics (metric_month);

-- Composite for common WHERE + JOIN patterns
CREATE NONCLUSTERED INDEX idx_customer_date
ON gold.customer_metrics (customer_id, metric_month)
INCLUDE (revenue, order_count);
```

### Pre-Aggregation

Create summary tables for fast dashboards:

```
gold/customer_metrics (row-level, 100M rows)
├─ gold/customer_metrics_monthly (aggregated, 5K rows)
├─ gold/customer_metrics_annual (aggregated, 500 rows)
└─ gold/top_100_customers (filtered, 100 rows)
```

### Caching

Cache frequently accessed data:

- In-memory: Redis for dashboards
- File cache: Parquet in SSD tier
- Materialized views for common aggregations
- Query result caching in BI tools

### Columnar Storage

Use Parquet/ORC for analytics:

- 10x compression vs. CSV
- Projection pushdown (read only needed columns)
- Predicate pushdown (filter before reading)
- Faster than row-based storage for analytics

### Statistics

Enable query optimizer:

```sql
ANALYZE TABLE gold.customer_metrics COMPUTE STATISTICS;
COMPUTE STATISTICS FOR TABLE gold.customer_metrics COLUMNS *;
```

This helps planners choose efficient execution paths.

## Data Quality Framework

### Quality Dimensions

- Accuracy: Data reflects true values
- Completeness: No missing required values
- Consistency: Uniform across sources
- Validity: Conforms to format rules
- Timeliness: Fresh and up-to-date
- Uniqueness: No unintended duplicates

### Quality Checks

Implement at each layer:

```python
# Great Expectations example
suite = context.create_expectation_suite("my_suite")

validator.expect_table_row_count_to_be_between(1000, 10000)
validator.expect_column_values_to_not_be_null("customer_id")
validator.expect_column_values_to_be_unique("email")
validator.expect_column_values_to_match_regex("email", r".*@.*\..*")
validator.expect_column_values_to_be_in_set("status", ["ACTIVE", "INACTIVE"])

context.save_expectation_suite(suite, overwrite=True)
```

### Anomaly Detection

Alert on unexpected changes:

```
Sudden drop in row count → ALERT
Metric outside 3-sigma range → ALERT
New null values in previously complete column → ALERT
Duplicate count spike → ALERT
```

## Security Best Practices

### Access Logs

Maintain audit trail:

```sql
CREATE TABLE audit_log (
  user_id STRING,
  table_name STRING,
  action STRING (SELECT|INSERT|UPDATE|DELETE),
  timestamp TIMESTAMP,
  row_count INT
);
```

### Secret Management

Never hardcode credentials:

```python
# Good: Use secret manager
from google.cloud import secretmanager

def access_secret_version(secret_id, version_id="latest"):
  client = secretmanager.SecretManagerServiceClient()
  secret = client.access_secret_version(request={...})
  payload = secret.payload.data.decode('UTF-8')
  return payload

# Bad: Hardcoded
password = "my_secret_password"  # DO NOT DO THIS
```

### Network Security

- VPC/Private subnets for databases
- Firewall rules limiting access
- SSL/TLS for data in transit
- IP whitelisting for ETL services

### Encryption

- Encryption at rest (database settings)
- Encryption in transit (TLS)
- Column-level encryption for sensitive data

## Scaling Considerations

### Volume Growth

As data grows:

1. Monitor partition sizes (5-50GB ideal)
2. Add partitioning levels (year → month → day)
3. Archive old partitions to cold storage
4. Implement retention policies

### Processing Time

If pipelines slow down:

1. Profile to find bottlenecks
2. Increase parallelism (more workers)
3. Add caching for frequently reused data
4. Create pre-aggregated tables
5. Consider materialized views

### Cost Management

- Archive old data to cold storage (S3 Glacier)
- Use reserved instances for steady-state workloads
- Schedule heavy computations off-peak
- Monitor per-query costs
- Use columnar formats and compression

## Change Management

### Testing Changes

Before production:

1. Test in development environment
2. Validate output against known values
3. Compare with previous version (if exists)
4. Run full suite of quality checks
5. Get peer review of logic changes

### Gradual Rollout

For major changes:

1. Run new and old logic in parallel
2. Compare results
3. Switch users gradually (canary deployment)
4. Have rollback plan ready
5. Monitor closely for issues

### Reverting Changes

Keep historical versions:

```
gold/customer_metrics/v1
gold/customer_metrics/v2 (current)
gold/customer_metrics/v1_backup (can revert to)
```

## Team Organization

### Responsibilities

- **Data Engineers**: Build and maintain pipelines
- **Analytics Engineers**: Build analytics-focused transformations
- **Data Analysts**: Use Gold layer for insights
- **ML Engineers**: Feature engineering, model training
- **Data Steward**: Governance, compliance, lineage

### Handoffs

Clear boundaries reduce conflicts:

- Data Engineer → Analytics Engineer: Clean, standardized Silver layer
- Analytics Engineer → Data Analyst: Business-ready Gold layer with documentation
- Analytics Engineer ↔ ML Engineer: Collaboration on feature sets

## Monitoring Dashboard Example

Create operational dashboard tracking:

```
Ingestion Metrics:
├─ Files ingested (daily count)
├─ Bronze row count trend
└─ Ingestion latency (p95 seconds)

Transformation Metrics:
├─ Silver table freshness
├─ Quality check pass rate
└─ Transformation duration (p95 min)

Gold Metrics:
├─ Business metric accuracy
├─ Gold table row counts
├─ Query latency (p95 seconds)

Alerts:
├─ Failed runs (email)
├─ Data quality failures (Slack)
└─ SLA breaches (dashboard highlight)
```
