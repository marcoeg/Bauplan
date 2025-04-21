# Data Quality and Expectations

In this example, we illustrate how to use expectation tests. These tests are statistical and quality checks applied to Bauplan models to ensure that the structure and values of the data meet our expectations. Expectation tests help detect data quality issues early and can be incorporated seamlessly into various workflows.

---
1. 📚 Description of the Pipeline
This Bauplan pipeline analyzes NYC taxi data to compute average taxi waiting times by pickup zone, while ensuring data quality through integrated expectations.

➔ Step 1: normalized_taxi_trips
S3 scan over taxi_fhvhv and taxi_zones tables.

Filter pushdown: selects December 2022 trips.

Arrow Join: joins trips with zones by PULocationID and LocationID.

Result: An Arrow table mapping each taxi trip to its pickup location metadata.

➔ Step 2: taxi_trip_waiting_times
Calculates waiting time (in minutes) between:

request_datetime (ride requested)

on_scene_datetime (taxi arrives at pickup)

Appends a new column waiting_time_minutes.

Result: Extended Arrow table including waiting times.

➔ Step 3: zone_avg_waiting_times
Uses DuckDB to:

Compute average waiting times for each Borough and Zone.

Order zones by descending waiting time.

Queries Arrow table directly without conversion (DuckDB reads Arrow natively).

Materialization strategy: REPLACE to persist the final result.

➔ Step 4: test_null_values_on_scene_datetime (Expectation)
Before allowing computations to proceed, tests that:

Column on_scene_datetime has no null values.

If nulls exist:

Raises an assertion error and stops the pipeline execution.

Quality assurance step: prevents incorrect waiting time calculations.

✨ Architectural Highlights:
Mix of Arrow and DuckDB: optimized for large-scale tabular computations.

Filter pushdown and join optimization at S3 scan level.

Integrated data expectations: built-in validation before heavy computation.

Clear, modular separation between models and expectations for maintainability.

