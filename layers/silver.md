# Silver Layer (Normalized Database)

## Overview

The Silver layer ensures data integrity for data coming from the Bronze layer.  Data is validated and so should always have the expected data types, format, and relationship with other pieces of data (e.g. if an investor is invested in a fund, that fund must exist in the Silver layer).

Data is organized to be optimized for writes and reduce duplication.  As a rule of thumb, if data needs to be updated (e.g. an investor's name changes), we only need to update in one place (e.g. the single record in the investor table).

We should strive for Boyce-Codd (3.5) normal form at minimum.

## Purpose

- Data quality enforcement - remove duplicates, handle nulls, validate formats, apply common-sense checks
- Standardization - apply consistent conventions for data types, formats, and naming fields
- Deduplication - identify and remove duplicate records
- Non-nullability - fields that are required should always have a value
- Referential integrity - if FieldA relies on FieldB, FieldB must exist with a proper value 

## Implementation Details

### Idempotency

Transformations from Bronze to Silver should be idempotent.  If Bronze data is unchanged, running the same piece of ETL multiple times should create the same data in the Silver layer.

### Traceability

Add fields as applicable so that source data from bronze can be identified.

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
