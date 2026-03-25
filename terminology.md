# Terminology

- **Insert**: Adding new rows to a table.
- **Update**: Updating field values in an existing row in a table.
- **Delete**: Removing an existing row in a table.
- **Upsert**: Update the field values in a row if it exists, otherwise insert it as a new record.
- **Data Quality**: The values of fields in the data make sense and are consistent with each other.
- **Data Integrity**: No duplicates, non-nullable fields exist, references to other entities exist, data types make sense
- **Schema**: Definition of tables, columns, data types, and constraints for a dataset.
- **Data Type**: The 'form' of what values the data is allowed to take or be mapped to, such as integer (1, 2, 100), numeric (1.0, 2.34, 100.494339), string/text ("Jane Doe", "Master Fund"), boolean (True/False or T/F or 0/1), etc.
- **Deduplication**: Process of removing duplicate rows based on natural keys or primary keys.
- **Data Lineage**: The pathway of data through the pipeline.
- **Partitioning**: Physical data layout strategy for performance. Common keys include date, source system, or region.
- **ACID**: Atomicity, Consistency, Isolation, Durability. A set of 4 transactional properties necessary for creating a relational database.
- **Immutability**: Data that cannot or should not ever change.
- **SLA (Service Level Agreement)**: Agreed time guarantees for pipeline completion and data availability.
- **ETL/ELT**: Extract, Transform, Load / Extract, Load, Transform. The order of steps taken to process data into a particular layer.
- **Reconciliation**: Matching and verifying row counts and values between source and target.
- **Golden Record/Master Source**: Best single version of a record consolidated from multiple sources, realized in the Gold layer.
