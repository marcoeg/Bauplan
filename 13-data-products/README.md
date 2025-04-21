# Building a Data Product with Bauplan and Prefect

This example demonstrates how to build a data product using [Bauplan](https://docs.bauplanlabs.com/) and [Prefect](https://docs.prefect.io/). A data product is a curated, reusable dataset or output (e.g., a report, dashboard dataset) that delivers value to end users. This pipeline reads raw data, processes it, and produces a data product stored in a destination.

## Overview

The pipeline performs the following steps:
1. **Read**: Load raw data from a source (e.g., S3 bucket, database).
2. **Process**: Clean, transform, or aggregate the data to create a curated output.
3. **Write**: Save the data product to a destination (e.g., S3 bucket, database, or API endpoint).

Bauplan handles data orchestration and execution, while Prefect manages workflow scheduling and monitoring.

## Prerequisites

- Bauplan CLI installed (`pip install bauplan`)
- Prefect installed (`pip install prefect`)
- AWS credentials configured for S3 access
- Python libraries: `pandas`
- A Bauplan account and API key

## Setup

1. **Clone the example repository**:
   ```bash
   git clone https://github.com/BauplanLabs/wap_with_bauplan_and_prefect.git
   cd wap_with_bauplan_and_prefect
   ```

2. **Install dependencies**:
   ```bash
   pip install bauplan prefect pandas
   ```

3. **Configure environment variables**:
   Set the following in a `.env` file or your shell:
   ```bash
   export BAUPLAN_API_KEY=<your-api-key>
   export AWS_ACCESS_KEY_ID=<your-access-key>
   export AWS_SECRET_ACCESS_KEY=<your-secret-key>
   ```

## Pipeline Code

The pipeline is defined in `dataproduct_pipeline.py`. Below is an example implementation that creates a curated dataset:

```python
from prefect import flow, task
import bauplan
import pandas as pd

@task
def read_data(source: str) -> pd.DataFrame:
    # Read raw data from source (e.g., S3)
    df = bauplan.read(source)
    return df

@task
def process_data(df: pd.DataFrame) -> pd.DataFrame:
    # Example: Clean and aggregate data
    # Remove null values
    df = df.dropna()
    # Aggregate by a column (e.g., 'category')
    df_agg = df.groupby('category').agg({
        'value': 'sum',
        'count': 'count'
    }).reset_index()
    # Add a timestamp
    df_agg['created_at'] = pd.Timestamp.now()
    return df_agg

@task
def write_data(df: pd.DataFrame, destination: str):
    # Write data product to destination
    bauplan.write(df, destination)

@flow(name="dataproduct-pipeline")
def dataproduct_pipeline(source: str, destination: str):
    df = read_data(source)
    processed_df = process_data(df)
    write_data(processed_df, destination)

if __name__ == "__main__":
    dataproduct_pipeline(
        source="s3://my-bucket/raw-data/",
        destination="s3://my-bucket/dataproduct/"
    )
```

## Running the Pipeline

1. **Start the Prefect server**:
   ```bash
   prefect server start
   ```

2. **Deploy the pipeline**:
   ```bash
   prefect deploy -f dataproduct_pipeline.py
   ```

3. **Run the pipeline**:
   Execute the pipeline locally or schedule it:
   ```bash
   prefect run -n dataproduct-pipeline
   ```

## Monitoring

- Use the Prefect UI to monitor pipeline runs: `http://localhost:4200`
- Check Bauplan logs for data processing details:
  ```bash
  bauplan logs
  ```

## Notes

- Customize the `process_data` task to fit your specific data product requirements (e.g., different aggregations, joins).
- Ensure the output format (e.g., CSV, Parquet) is compatible with downstream consumers of the data product.
- Verify S3 bucket permissions for read/write access.
- For production, consider scheduling the pipeline with Prefect to run periodically (e.g., daily).
- Document the data product schema and metadata for end users.

For more details, refer to the [Bauplan documentation](https://docs.bauplanlabs.com/) or [Prefect documentation](https://docs.prefect.io/).