# SQL Server Naming Conventions

This document outlines the naming conventions for databases, tables, columns, and other database objects in SQL Server environments within the Medallion Architecture.

## General Principles

- Use snake_case for all database object names (e.g., `customer_order` instead of `CustomerOrder`).
- Names should be descriptive and self-explanatory.
- Avoid abbreviations unless they are widely understood and documented.
- Use singular nouns for table names (e.g., `customer` not `customers`).
- Keep names concise but meaningful.
- Prefixes and suffixes should be used consistently for clarity.

## Databases

- Use descriptive names that reflect the purpose or domain.
- Example: `sales_data`, `inventory_management`, `customer_analytics`.

## Tables

- Use singular nouns in snake_case.
- Examples:
  - `customer`
  - `order`
  - `product_category`

## Columns

- Use snake_case for column names.
- Primary key columns should end with `_id` (e.g., `customer_id`).
- Foreign key columns should match the referenced primary key name.
- Boolean columns should start with `is_` or `has_` (e.g., `is_active`, `has_discount`).
- Date/time columns should include `date` or `time` in the name (e.g., `created_date`, `modified_time`).
- Examples:
  - `first_name`
  - `order_date`
  - `unit_price`

## Primary Keys

- Name: `id` for the primary key column.
- Constraint: `pk_table_name` (e.g., `pk_customer`).

## Foreign Keys

- Column name: Match the referenced primary key (e.g., `customer_id`).
- Constraint: `fk_table_name_referenced_table_name` (e.g., `fk_order_customer`).

## Indexes

- Non-clustered indexes: `ix_table_name_column_name` (e.g., `ix_order_order_date`).
- Unique indexes: `ux_table_name_column_name` (e.g., `ux_customer_email`).
- Clustered indexes: `cx_table_name_column_name` (if not the primary key).

## Constraints

- Check constraints: `ck_table_name_column_name` (e.g., `ck_product_unit_price`).
- Default constraints: `df_table_name_column_name` (e.g., `df_order_order_date`).

## Views

- Prefix with `vw_` followed by snake_case name (e.g., `vw_customer_orders`).

## Stored Procedures

- Prefix with `usp_` followed by snake_case name (e.g., `usp_get_customer_by_id`).

## Functions

- Scalar functions: `fn_function_name` (e.g., `fn_calculate_tax`).
- Table-valued functions: `tvf_function_name` (e.g., `tvf_get_active_customers`).

## Triggers

- Prefix with `tr_` followed by the table name and trigger type (e.g., `tr_customer_insert`, `tr_order_update`).

## Schemas

- Use descriptive names in snake_case (e.g., `sales`, `hr`, `reporting`).


## Best Practices

- Consistency is key: Apply these conventions across all databases and projects.
- Document any deviations in the project documentation.
- Use tools like SQL Server Management Studio's naming templates to enforce conventions.
- Regularly review and refactor names as the schema evolves.