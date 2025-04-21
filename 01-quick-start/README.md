# Quick Start

Welcome to you bauplan tutorial. Here you will learn the ropes and gain a good understanding of the main capabilities of the platform.

## Pipeline Description 
This pipeline is designed to demonstrate a modular and extensible transformation flow using **Bauplan models**. It chains together table transformations into a **directed acyclic graph (DAG)**.

**Key characteristics:**
- **Composable models**: Each transformation is a self-contained Python function, explicitly marked with `@bauplan.model()` and `@bauplan.python()`.
- **Isolated environments**: Each model can specify its own Python version and required packages (e.g., pandas, numpy), ensuring environment reproducibility and flexibility.
- **Efficient I/O**: Models operate on **Apache Arrow tables** in memory, minimizing serialization overhead.
- **Materialization control**: Output tables can be **materialized** (persisted) in the catalog, or kept ephemeral, depending on the model configuration.
- **Optimization-ready**: Columns and filters are specified early for S3 scan pushdowns, reducing data transfer and computation costs.

---

### Step-by-Step Overview:

1. **Load Raw Data (Input Models)**:
   - `trips` model scans an existing table (`taxi_fhvhv`) with **column selection** and **filter pushdown** on pickup time.
   - `zones` model loads the static **taxi zones lookup table** (`taxi_zones`).

2. **Join and Combine**:
   - `trips_and_zones` model **joins** the trips data with the zones table based on `PULocationID` and `LocationID`.
   - It **combines chunks** post-join to create a single Arrow table.
   - **Environment**: Python 3.11, no extra packages.

3. **Normalize and Filter**:
   - `normalized_taxi_trips` model **takes `trips_and_zones` output** and:
     - Converts it from Arrow to Pandas.
     - Applies **timestamp filters** to select recent trips.
     - Removes trips with 0 or excessive miles.
     - **Log-transforms** `trip_miles` to handle skewed distributions.
   - **Environment**: Python 3.11 with **pandas 1.5.3** and **numpy 1.23.2**.

4. **Materialization Strategy**:
   - `trips_and_zones`: ephemeral output (not persisted unless explicitly configured).
   - `normalized_taxi_trips`: persisted with `materialization_strategy='REPLACE'`, ensuring the latest normalized version is stored.

---

### Example Usage

You can run the full pipeline locally or remotely using:

```bash
$ bauplan branch create <username>.<branch_name>
$ bauplan checkout <username>.<branch_name>
$ bauplan run
```

Each model will be executed according to the DAG dependencies, materializing only the specified outputs.

---
In this example, we use the [TLC NY Taxi dataset](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) 
