# Module 2 Homework Solutions

## Questions and Answers

### Question 1: Uncompressed File Size
**Answer:** A - 128.3 MiB

**Query Used:**
```bash
# Execution: Flow 08_gcp_taxi
# Inputs: taxi=yellow, year=2020, month=12
# Result: Checked extract task output
```

---

### Question 2: Variable Rendering
**Answer:** B - green_tripdata_2020-04.csv

**Explanation:**
```yaml
# Template:
file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"

# Inputs:
taxi: green
year: 2020
month: 04

# Rendered:
green_tripdata_2020-04.csv
```

---

### Question 3: Yellow Taxi 2020 Total Rows
**Answer:** B - 24,648,499

**Query Used:**
```sql
SELECT COUNT(*) as yellow_2020_total_rows
FROM `dtc-de-course-484520.trips_data_all.yellow_tripdata`
WHERE EXTRACT(YEAR FROM tpep_pickup_datetime) = 2020;
```

**Result:** 24,648,219 (≈24,648,499)

---

### Question 4: Green Taxi 2020 Total Rows
**Answer:** C - 1,734,051

**Query Used:**
```sql
SELECT COUNT(*) as green_2020_total_rows
FROM `dtc-de-course-484520.trips_data_all.green_tripdata`
WHERE EXTRACT(YEAR FROM lpep_pickup_datetime) = 2020;
```

**Result:** 1,733,998 (≈1,734,051)

---

### Question 5: Yellow Taxi March 2021
**Answer:** C - 1,925,152

**Query Used:**
```sql
SELECT COUNT(*) as march_2021_yellow_trips
FROM `dtc-de-course-484520.trips_data_all.yellow_tripdata`
WHERE DATE(tpep_pickup_datetime) BETWEEN '2021-03-01' AND '2021-03-31';
```

**Result:** 1,925,119 (≈1,925,152)

---

### Question 6: Timezone Configuration
**Answer:** B - Add a timezone property set to America/New_York

**Explanation:**
```yaml
triggers:
  - id: schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * *"
    timezone: America/New_York  # IANA timezone format
```

**Why this is correct:**
- Uses IANA timezone database format
- Automatically handles Daylight Saving Time
- Official Kestra documentation format

**Why others are wrong:**
- EST: Doesn't handle DST (wrong in summer)
- UTC-5: Fixed offset, doesn't handle DST
- location: Wrong property name


