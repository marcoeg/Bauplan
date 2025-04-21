# Near Real-Time Processing with Bauplan and Prefect

This example shows how to build a near-real-time (NRT) data processing pipeline using [Bauplan](https://docs.bauplanlabs.com/) and [Prefect](https://docs.prefect.io/). The pipeline reads data from a source, processes it, and writes the results to a destination with minimal latency.

## Overview

The pipeline performs the following steps:
1. **Read**: Continuously monitor a source (e.g., S3 bucket) for new data.
2. **Process**: Apply transformations or computations to the incoming data.
3. **Write**: Save the processed data to a destination (e.g., another S3 bucket or database).

Bauplan handles the data orchestration and execution, while Prefect manages the workflow scheduling and monitoring.

## Prerequisites

- Bauplan CLI installed (`pip install bauplan`)
- Prefect installed (`pip install prefect`)
- AWS credentials configured for S3 access
- A Bauplan account and API key

## Setup

1. **Clone the example repository**:
   ```bash
   git clone https://github.com/BauplanLabs/wap_with_bauplan_and_prefect.git
   cd wap_with_bauplan_and_prefect
   ```

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure environment variables**:
   Set the following in a `.env` file or your shell:
   ```bash
   export BAUPLAN_API_KEY=<your-api-key>
   export AWS_ACCESS_KEY_ID=<your-access-key>
   export AWS_SECRET_ACCESS_KEY=<your-secret-key>
   ```

## Pipeline Code

The pipeline is defined in `nrt_pipeline.py`. Below is an example implementation:

```python
from prefect import flow, task
import bauplan
import pandas as pd

@task
def read_data(source: str) -> pd.DataFrame:
    # Read new data from source (e.g., S3)
    df = bauplan.read(source)
    return df

@task
def process_data(df: pd.DataFrame) -> pd.DataFrame:
    # Example transformation: add a new column
    df['processed'] = df['value'].apply(lambda x: x * 2)
    return df

@task
def write_data(df: pd.DataFrame, destination: str):
    # Write results to destination
    bauplan.write(df, destination)

@flow(name="nrt-pipeline")
def nrt_pipeline(source: str, destination: str):
    df = read_data(source)
    processed_df = process_data(df)
    write_data(processed_df, destination)

if __name__ == "__main__":
    nrt_pipeline(
        source="s3://my-bucket/input/",
        destination="s3://my-bucket/output/"
    )
```

## Running the Pipeline

1. **Start the Prefect server**:
   ```bash
   prefect server start
   ```

2. **Deploy the pipeline**:
   ```bash
   prefect deploy -f nrt_pipeline.py
   ```

3. **Run the pipeline**:
   Execute the pipeline locally or schedule it:
   ```bash
   prefect run -n nrt-pipeline
   ```

## Monitoring

- Use the Prefect UI to monitor pipeline runs: `http://localhost:4200`
- Check Bauplan logs for data processing details:
  ```bash
  bauplan logs
  ```

## Notes

- The pipeline checks for new data every 60 seconds by default. Adjust the schedule in Prefect as needed.
- Ensure your S3 buckets are properly configured with read/write permissions.
- For production, consider deploying to a cloud-based Prefect instance.

For more details, refer to the [Bauplan documentation](https://docs.bauplanlabs.com/) or [Prefect documentation](https://docs.prefect.io/).