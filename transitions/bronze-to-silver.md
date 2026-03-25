# Bronze -> Silver Transition

## Overview

Apply validation, cleaning, standardization, and deduplication to make data trustworthy and consistent.

## Key Steps

### Validation and Cleaning
- Check non-null on critical fields
- Check value ranges and domains
- Trim whitespace and remove unnecessary characters
- Handle nulls (default values)
- Address data gaps that a user can address.

### Standardization
- Cast to correct data types (int, date, datetime, varchar, etc.)
- Normalize values (e.g., is_master_fund: Y/N -> TRUE/FALSE)
- Apply consistent formats and naming (naming of funds, internal investor mappings)

### Deduplication
- If duplicates exist (whether due to vendor error or data sources with the same information), implement logic to choose between them.
- Extract out items of interest (investors, )

### Enrichment
- Add autoincrementing unique identifiers
- Add lookups for related tables
- Derive simple flags (e.g. is_master_fund)