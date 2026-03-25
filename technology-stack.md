# Technology Stack

## Overview

The medallion architecture is platform-agnostic. This guide covers popular technologies for each layer of the stack.

## Orchestration & Scheduling

Orchestration tools manage pipeline execution, dependencies, and error handling.

### Apache Airflow

**When to use**: Complex DAGs, on-premise deployments, maximum flexibility

**Pros**:
- Highly flexible Python-based configuration
- Rich UI for monitoring
- Large community and ecosystem
- Excellent for complex dependencies

**Cons**:
- Steep learning curve
- Requires infrastructure management
- Can be slower than alternatives for high-frequency tasks

**Example**:
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

dag = DAG('medallion_pipeline', start_date=datetime(2024, 1, 1))

def bronze_to_silver():
    # Run transformations
    pass

def silver_to_gold():
    # Aggregations
    pass

bronze_task = PythonOperator(
    task_id='bronze_to_silver',
    python_callable=bronze_to_silver,
    dag=dag
)

gold_task = PythonOperator(
    task_id='silver_to_gold',
    python_callable=silver_to_gold,
    dag=dag
)

bronze_task >> gold_task
```

### Databricks Workflows

**When to use**: Databricks customers, notebook-first workflows, tight Spark integration

**Pros**:
- Native Spark integration
- No infrastructure management
- Strong collaboration features
- Built-in notebook versioning

**Cons**:
- Databricks platform lock-in
- Limited to Databricks ecosystem
- Less flexibility than Airflow for complex logic

### Prefect

**When to use**: Modern UI, easy onboarding, strong observability

**Pros**:
- Beautiful UI and documentation
- Gentle learning curve
- Modern Python async support
- Built-in retries and error handling

**Cons**:
- Newer ecosystem (less third-party integrations)
- Cloud platform (harder to run on-premise)

### dbt (Transformation Orchestration)

**When to use**: SQL-centric transformations, team collaboration

**Pros**:
- SQL-first approach
- Excellent testing and documentation
- Built-in lineage tracking
- Active community

**Cons**:
- Limited to SQL transformations
- Doesn't handle non-SQL tasks
- Orchestration support is secondary

## Data Transformation Engines

### Apache Spark

**When to use**: Large-scale distributed processing, Python/Scala, complex transformations

**Pros**:
- Massive scale (petabytes)
- Streaming and batch support
- Machine learning libraries (MLlib)
- Broad industry adoption

**Cons**:
- Steeper learning curve
- Cluster management complexity
- Overkill for small datasets

**Example**:
```python
from pyspark.sql.functions import col, sum, count, date_trunc

orders = spark.read.parquet("s3://silver/orders")

monthly_metrics = (orders
  .groupBy(col("customer_id"), date_trunc("month", col("order_date")))
  .agg(sum("amount"), count("*"))
  .write.mode("overwrite")
  .parquet("s3://gold/metrics")
)
```

### SQL

**When to use**: Data warehouses (BigQuery, Snowflake, Redshift)

**Pros**:
- Widely understood
- Excellent performance on structured data
- Native to data warehouses
- Ecosystem maturity

**Cons**:
- Limited for unstructured data (JSON, images)
- Some operations awkward in SQL

### dbt

**When to use**: SQL transformations in warehouse, team-first approach

**Pros**:
- SQL with software engineering practices
- Tests and documentation first-class
- Version control friendly
- Built-in lineage and DAG management

**Cons**:
- SQL-only (no Python/complex logic)
- Requires data warehouse
- Learning curve for non-SQL engineers

**Example**:
```sql
-- models/gold/customer_metrics.sql
{{ config(
  materialized='incremental',
  partition_by={'field': 'metric_month', 'data_type': 'date'},
  tags=['daily']
) }}

SELECT
  customer_id,
  DATE_TRUNC(order_date, MONTH) as metric_month,
  SUM(amount) as revenue
FROM {{ ref('orders_cleaned') }}
GROUP BY 1, 2

{% if execute %}
  WHERE metric_month >= (SELECT MAX(metric_month) FROM {{ this }})
{% endif %}
```

### Python (Pandas/Polars)

**When to use**: Complex business logic, small-to-medium datasets

**Pros**:
- Flexible and expressive
- Easy to debug
- Great for data science workflows
- Rich ecosystem (NumPy, SciPy, etc.)

**Cons**:
- Single-machine processing
- Memory limitations
- Slower than specialized engines
- Not suitable for scaling

## Data Storage

### Delta Lake

**When to use**: ACID transactions, time travel, data governance

**Pros**:
- ACID compliance
- Schema enforcement
- Time travel queries
- Data versioning

**Cons**:
- Requires compute engine (Spark/query engine)
- Vendor lock-in to Delta-compatible tools

**Example**:
```python
df.write.format("delta").mode("overwrite").save("/mnt/gold/customers")

# Read with time travel
spark.read.format("delta").option("versionAsOf", 0).load("/mnt/gold/customers")
```

### Parquet

**When to use**: Open format, broad compatibility, compression needed

**Pros**:
- Open standard
- Excellent compression (10x vs CSV)
- Columnar format (analytics-friendly)
- Widely supported

**Cons**:
- No ACID transactions
- Schema evolution limited
- Immutable (must rewrite to change)

### Apache Iceberg

**When to use**: ACID + open format, enterprise requirements

**Pros**:
- ACID transactions
- Open standard (unlike Delta)
- Time travel
- Schema evolution

**Cons**:
- Newer (less mature)
- Smaller ecosystem than Delta
- More complex than Parquet

### Cloud Data Warehouses

**BigQuery** (GCP):
- Advantages: Serverless, fast, great UI, strong ML integration
- Disadvantages: Google ecosystem, data residency restrictions

**Snowflake**:
- Advantages: Multi-cloud, separation of compute/storage, excellent performance
- Disadvantages: Expensive, no free tier options

**Redshift** (AWS):
- Advantages: AWS integration, cost-effective for large volumes
- Disadvantages: Cluster management, less flexible than Snowflake

## Data Quality & Testing

### Great Expectations

**When to use**: Comprehensive data quality validation

**Pros**:
- Python-native
- Comprehensive validation types
- Documentation-first approach
- Visual data docs

**Cons**:
- Python-focused (not SQL)
- Steeper learning curve

### dbt Tests

**When to use**: SQL transformation testing

**Pros**:
- Integrated with dbt
- Run with transformations
- Version controlled
- Low friction adoption

**Cons**:
- Limited test types
- SQL-only

### SQLAlchemy

**When to use**: Custom SQL validation

**Pros**:
- Flexible custom SQL
- Database-agnostic
- Python ecosystem

**Cons**:
- More code needed
- No managed experience

## Monitoring & Observability

### Datadog

**When to use**: Comprehensive monitoring, multi-source observability

**Pros**:
- Multi-source integration
- Custom metrics
- Excellent alerting
- Beautiful dashboards

**Cons**:
- Costly
- Can be complex to configure

### Prometheus + Grafana

**When to use**: Cost-conscious, open-source preference

**Pros**:
- Open source
- Time-series database optimized
- Excellent dashboards (Grafana)
- Minimal cost

**Cons**:
- Self-hosted management
- Smaller ecosystem than Datadog

### Cloud-Native Monitoring

- GCP: Cloud Monitoring, Cloud Logging
- AWS: CloudWatch
- Azure: Azure Monitor

## Recommended Stacks

### On-Premise / Self-Hosted

```
Orchestration: Apache Airflow
Transformation: SQL (if warehouse) or PySpark
Storage: Parquet / Delta Lake on HDFS
Data Warehouse: Postgres / MySQL (small) or StarRocks
Quality: dbt tests + Great Expectations
Monitoring: Prometheus + Grafana
```

### AWS-First

```
Orchestration: AWS Step Functions or Airflow on EC2
Transformation: PySpark on EMR + dbt
Storage: S3 in Parquet format
Data Warehouse: Redshift or Athena
Quality: Great Expectations + Glue DataBrew
Monitoring: CloudWatch
```

### GCP-First

```
Orchestration: Cloud Composer (Airflow)
Transformation: Dataflow (Apache Beam) or dbt
Storage: Cloud Storage (Parquet/Avro)
Data Warehouse: BigQuery
Quality: Great Expectations + Data Quality API
Monitoring: Cloud Monitoring
```

### Databricks-Centric

```
Orchestration: Databricks Workflows
Transformation: PySpark + dbt-databricks
Storage: Delta Lake
Data Warehouse: Unity Catalog
Quality: Databricks Data Quality
Monitoring: Databricks built-in monitoring
```

### Enterprise / Data Mesh

```
Orchestration: Apache Airflow (multi-team)
Transformation: dbt (all teams) + PySpark
Storage: Data Lake on Cloud (S3/GCS/ADLS)
Data Warehouse: Snowflake (shared)
Data Catalog: Alation or Collibra
Quality: Great Expectations + dbt
Monitoring: Datadog or Splunk
Storage Layer: Apache Kafka (event streams)
```

## Technology Comparison Matrix

| Requirement | Best For |
|---|---|
| **Traditional warehouse** | SQL, dbt, cloud warehouse |
| **Real-time streaming** | Kafka + Spark + operational DB |
| **Large-scale batch** | Spark, cloud warehouse, Iceberg |
| **Complex transformations** | Spark + Python or dbt macros |
| **High governance** | dbt + data catalog + Great Expectations |
| **Cost-conscious** | On-prem + Postgres + Airflow + Prometheus |
| **Fully managed** | Databricks or cloud warehouses |
| **Multi-team (data mesh)** | Airflow + dbt + shared data warehouse + catalog |

## Migration Considerations

### From Legacy Warehouse

```
Step 1: Build Bronze layer with CDC
Step 2: Replicate cleaned data (Silver) to new warehouse
Step 3: Build Gold layer with new definitions
Step 4: Validate metrics match legacy system
Step 5: Gradually switch users to new Gold layer
```

### From Data Lake

```
Step 1: Organize lake into Bronze/Silver/Gold directories
Step 2: Implement data governance (ownership, classification)
Step 3: Add quality checks between layers
Step 4: Create data catalog
Step 5: Implement access controls
```

## Decision Framework

Choose your stack by answering:

1. **Scale**: How much data? (GB, TB, PB?)
   - < 1TB → SQL + single machine
   - 1-100TB → Cloud warehouse
   - > 100TB → Data lake + warehouse

2. **Velocity**: How often does data change?
   - Batch (hourly+) → Traditional stack
   - Streaming → Kafka + streaming processor

3. **Complexity**: How complex are transformations?
   - Simple ETL → dbt or SQL
   - Complex logic → Spark + Python

4. **Budget**: What's the cost constraint?
   - Unlimited → Fully managed (Databricks, Snowflake)
   - Limited → Open source + cloud storage

5. **Team**: What's the team's expertise?
   - SQL team → Cloud warehouse + dbt
   - Python team → Spark everywhere
   - Mixed → Airflow + dbt + Spark
