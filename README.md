# Incremental Booking & Customer Data Pipeline using Databricks

## Overview

This project implements an end-to-end Incremental Data Pipeline using Databricks, PySpark, Delta Lake, Unity Catalog Volumes, and Databricks Workflows.

The pipeline processes daily booking and customer CSV files stored in Google Cloud Storage (GCS), performs data quality validation, loads aggregated booking data into a Delta Fact Table, maintains Customer Dimension history using Slowly Changing Dimension Type 2 (SCD2), and is orchestrated through a parameterized Databricks Workflow.

---

## Tech Stack

* Databricks
* PySpark
* Delta Lake
* Unity Catalog
* Unity Catalog Volumes
* Google Cloud Storage (GCS)
* Databricks Workflows
* Delta Merge Operations

---

## Data Source

Daily files are stored in a Unity Catalog Volume backed by Google Cloud Storage.

### Booking Files

```text
/Volumes/incremental_load/default/orders_data/booking_data/
```

Example:

```text
bookings_2024-07-25.csv
bookings_2024-07-26.csv
bookings_2024-07-27.csv
```

### Customer Files

```text
/Volumes/incremental_load/default/orders_data/customer_data/
```

Example:

```text
customers_2024-07-25.csv
customers_2024-07-26.csv
```

---

## Pipeline Workflow

### Step 1: Workflow Parameter

A Databricks Job passes the processing date as a parameter.

Example:

```text
arrival_date = 2024-07-25
```

The notebook dynamically constructs file paths using the supplied date.

---

### Step 2: Read Incremental Files

The pipeline reads:

```text
bookings_<arrival_date>.csv
customers_<arrival_date>.csv
```

Only the files for the supplied processing date are loaded.

This makes the pipeline incremental.

---

### Step 3: Data Quality Validation

Data quality checks are executed before any transformation.

#### Booking Data Checks

* File is not empty
* booking_id is not null
* customer_id is not null
* amount is not null
* amount is non-negative
* quantity is non-negative
* discount is non-negative
* booking_id is unique

#### Customer Data Checks

* File is not empty
* customer_id is not null
* customer_name is not null
* customer_address is not null
* email is not null
* customer_id is unique

---

### Step 4: Pipeline Validation

If any data quality check fails:

```text
Pipeline Stops
```

If all validations pass:

```text
Pipeline Continues
```

---

### Step 5: Data Enrichment

Booking data is enriched by:

* Adding ingestion timestamp
* Joining customer master data

Join Key:

```text
customer_id
```

---

### Step 6: Business Transformation

Total booking cost is calculated as:

```text
total_cost = amount - discount
```

Invalid records with quantity <= 0 are removed.

---

### Step 7: Booking Aggregation

Data is aggregated by:

```text
booking_type
customer_id
```

Metrics generated:

* total_amount_sum
* total_quantity_sum

---

### Step 8: Incremental Fact Table Load

Target Table:

```text
incremental_load.default.booking_fact
```

The pipeline performs Delta Merge operations.

#### Existing Records

Updates aggregated metrics.

#### New Records

Inserts new rows.

This ensures incremental loading without reprocessing historical data.

---

### Step 9: SCD Type 2 Customer Dimension

Target Table:

```text
incremental_load.default.customer_dim
```

The pipeline maintains customer history using SCD Type 2.

For changed customer records:

1. Existing active record is expired.
2. valid_to is updated.
3. New customer version is inserted.
4. Historical records are preserved.

Columns used:

```text
valid_from
valid_to
```

---

## Databricks Workflow

A parameterized Databricks Workflow is configured to execute the notebook.

### Job Parameter

| Key          | Value           |
| ------------ | --------------- |
| arrival_date | Processing Date |

Example:

```text
arrival_date = 2024-07-25
```

---

## Incremental Processing Example

### Day 1

Input:

```text
bookings_2024-07-25.csv
customers_2024-07-25.csv
```

Output:

* booking_fact populated
* customer_dim populated

### Day 2

Input:

```text
bookings_2024-07-26.csv
customers_2024-07-26.csv
```

Output:

* booking_fact incrementally updated via Delta Merge
* customer_dim updated using SCD2 logic


---

## Solution Architecture

```text
GCS Bucket
    │
    ▼
Unity Catalog Volume
    │
    ▼
Databricks Workflow
(arrival_date parameter)
    │
    ▼
Read Daily Booking & Customer Files
    │
    ▼
Data Quality Validation
    │
    ├── Fail → Stop Pipeline
    │
    └── Pass
            │
            ▼
      Join & Transform Data
            │
            ▼
      Aggregate Booking Data
            │
            ▼
      Delta Merge
      booking_fact
            │
            ▼
      SCD Type 2 Merge
      customer_dim
            │
            ▼
      Incremental Delta Tables
```

---

## Key Features

* Incremental Processing
* Parameter Driven Execution
* Delta Lake Merge Operations
* Slowly Changing Dimension Type 2
* Data Quality Validation
* Unity Catalog Governance
* Google Cloud Storage Integration
* Databricks Workflow Orchestration
* Historical Customer Tracking
* Production-Ready ETL Design
