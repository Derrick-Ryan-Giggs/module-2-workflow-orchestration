# Module 2: Workflow Orchestration

## Overview
Built end-to-end data pipelines using Kestra to ingest NYC Taxi data into GCP.

## What I Learned
-  Kestra workflow orchestration
-  Dynamic workflows with inputs and variables
-  GCS bucket management
-  BigQuery table creation and partitioning
-  MERGE operations for data deduplication
-  Schedule triggers and backfilling
-  Handling manual vs scheduled executions

## Architecture
```
GitHub (NYC Taxi CSVs)
    ↓ wget + gunzip
Kestra Worker (extract)
    ↓ upload
Google Cloud Storage (raw CSV)
    ↓ external table
BigQuery (staging)
    ↓ MERGE
BigQuery (production table - partitioned)
```

## Data Pipeline Stats
- **Data Sources:** Yellow & Green Taxi trips
- **Time Period:** 2020-2021
- **Total Records Processed:** ~26 million rows
- **Storage:** ~2 GB in GCS, ~1 GB in BigQuery
- **Cost:** < $1 (well within free tier!)

## Flows Implemented

### 1. `06_gcp_kv.yaml` - Credentials Management
Securely stored GCP credentials in Kestra's KV Store

### 2. `07_gcp_setup.yaml` - Infrastructure Setup
Created GCS bucket and BigQuery dataset

### 3. `08_gcp_taxi.yaml` - Manual Data Pipeline
- Downloads compressed CSV from GitHub
- Uploads to GCS
- Creates external and staging tables
- Merges into partitioned production table
- Prevents duplicates with MD5 hashing

### 4. `09_gcp_taxi_scheduled.yaml` - Automated Pipeline
Same as above but triggered on a schedule (monthly)

## Homework Results

### Question 1: File Size 
**Answer:** 128.3 MiB
- Extracted yellow_tripdata_2020-12.csv

### Question 2: Template Rendering 
**Answer:** green_tripdata_2020-04.csv
- Learned Pebble templating syntax

### Question 3: Yellow Taxi 2020 Rows 
**Answer:** 24,648,499 rows
- Loaded all 12 months of 2020

### Question 4: Green Taxi 2020 Rows 
**Answer:** 1,734,051 rows
- Loaded all 12 months of 2020

### Question 5: Yellow Taxi March 2021 
**Answer:** 1,925,152 rows
- Used allowCustomValue to input 2021

### Question 6: Timezone Configuration 
**Answer:** timezone: America/New_York
- Learned IANA timezone database format

## Key Learnings

### 1. Handling trigger.date for Manual Runs
**Problem:** Flow failed when run manually because `trigger.date` didn't exist

**Solution:** Used null coalescing operator
```yaml
file: "{{inputs.taxi}}_tripdata_{{(trigger.date ?? inputs.process_date) | date('yyyy-MM')}}.csv"
```

### 2. GCS Storage Class Compatibility
**Problem:** Can't use REGIONAL storage class with multi-region location (US)

**Solution:** Changed to STANDARD storage class
```yaml
storageClass: STANDARD  # Works with US multi-region
```

### 3. BigQuery Partitioning
Partitioned tables by pickup datetime for better query performance:
```sql
PARTITION BY DATE(tpep_pickup_datetime)
```

### 4. Deduplication Strategy
Used MD5 hashing to create unique row IDs:
```sql
MD5(CONCAT(
  COALESCE(CAST(VendorID AS STRING), ""),
  COALESCE(CAST(tpep_pickup_datetime AS STRING), ""),
  ...
)) AS unique_row_id
```

## Screenshots
![Kestra Dashboard] <img width="1920" height="1080" alt="Screenshot From 2026-01-29 21-39-26" src="https://github.com/user-attachments/assets/b4d064c3-c291-42cb-95c1-4cb7b8a548b3" />, 
<img width="1920" height="1080" alt="Screenshot From 2026-01-29 21-40-10" src="https://github.com/user-attachments/assets/472e7923-9dc0-4fef-ad0b-26787baa0248" />

![BigQuery Tables] <img width="1920" height="1080" alt="Screenshot From 2026-01-29 22-04-14" src="https://github.com/user-attachments/assets/7d27c47f-8b6c-428b-8f91-880db789c2bc" />
<img width="1920" height="1080" alt="Screenshot From 2026-01-29 22-08-33" src="https://github.com/user-attachments/assets/32c3a355-ce55-4259-a1fc-54dffb74858d" />

![Successful Execution] <img width="1920" height="1080" alt="Screenshot From 2026-01-29 22-23-05" src="https://github.com/user-attachments/assets/ec23a7f4-2a1b-459f-937c-63edf8a30d7f" />


## Files in This Directory
- `flows/` - All Kestra YAML workflow files
- `homework/` - Homework questions and SQL queries
- `screenshots/` - Project screenshots
- `README.md` - This file

## Resources
- [Kestra Documentation](https://kestra.io/docs)
- [NYC TLC Data](https://github.com/DataTalksClub/nyc-tlc-data)
- [BigQuery Best Practices](https://cloud.google.com/bigquery/docs/best-practices)

