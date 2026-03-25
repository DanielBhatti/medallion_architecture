# Database Naming Conventions

This document outlines the naming conventions for databases, tables, columns, and other database objects.

## General Principles

- Use snake_case for all database object names (e.g., `fund_investor_mapping`).
- Names should ideally be self-explanatory.
- Avoid abbreviations unless they are widely understood.
- Use singular nouns for table names (e.g., `investor` not `investors`).
- Keep names concise but meaningful.
- Prefixes and suffixes should be used consistently for clarity.

## Databases

- Use descriptive names that reflect the purpose or source
- Example: `investor_database`, `inventory_management`.

## Tables

- Use singular nouns in snake_case.
- Examples:
  - `fund`
  - `investor`
  - `fund_investor_mapping`

## Columns

- Use snake_case for column names.
- Primary key columns should end with `_id` (e.g., `fund_id`).
- Foreign key columns should match the referenced primary key name.
- Boolean columns should start with `is_` or `has_` (e.g., `is_active`, `is_master_fund`).
- Date/time columns should include `date` or `time` in the name (e.g., `created_date`, `modified_time`).  Default to UTC unless the time zone or offset is specified within the data.
- Data with units should include the unit in the name if not already clear:
  - `updated_timestamp_est`
  - `transaction_amount_usd`
  - `transaction_amount_local`

## Primary Keys

- Name: `table_name_id` for the primary key column.
- Constraint: `pk_table_name` (e.g., `pk_customer`).

## Foreign Keys

- Column name: Match the referenced primary key (e.g., `fund_id`).
- Constraint: `fk_table_name_referenced_table_name` (e.g., `fk_investor_fund`).

## Indexes

- Non-clustered indexes: `ix_table_name_column_name` (e.g., `ix_fund_fund_name`).
- Unique indexes: `ux_fund_fund_name` (e.g., `ux_customer_email`).
- Clustered indexes: `cx_table_name_column_name` (if not the primary key).

## Constraints

- Check constraints: `ck_table_name_column_name` (e.g., `ck_commitment_commitment_amount` when ensuring commitments are always positive).
- Default constraints: Only add for retrofitting new columns, `df_table_name_column_name` (e.g., `df_investor_erisa_flag`).

## Views

- Prefix with `vw_` followed by snake_case name (e.g., `vw_capital_activity`).

## Stored Procedures

- Prefix with `usp_` followed by snake_case name (e.g., `usp_get_customer_by_id`).

## Functions

- Scalar functions: `fn_function_name` (e.g., `fn_calculate_tax`).
- Table-valued functions: `tvf_function_name` (e.g., `tvf_get_active_customers`).

## Triggers

- Avoid these unless absolutely necessary.  Prefer a centralized scheduler.
- Prefix with `tr_` followed by the table name and trigger type (e.g., `tr_customer_insert`, `tr_order_update`).

## Schemas
- Naming convention?