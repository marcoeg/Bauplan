# LLM Tabular Data Processing with Bauplan and Prefect

This example demonstrates how to build a data processing pipeline that uses a large language model (LLM) to process tabular data with [Bauplan](https://docs.bauplanlabs.com/) and [Prefect](https://docs.prefect.io/). The pipeline reads tabular data, applies LLM-based transformations (e.g., data enrichment, classification), and writes the results to a destination.

## Overview

The pipeline performs the following steps:
1. **Read**: Load tabular data (e.g., CSV, Parquet) from a source like S3.
2. **Process**: Use an LLM to enrich or transform the data (e.g., generate descriptions, classify rows).
3. **Write**: Save the processed data to a destination (e.g., S3 bucket or database).

Bauplan handles data orchestration and execution, while Prefect manages workflow scheduling and monitoring.

## Prerequisites

- Bauplan CLI installed (`pip install bauplan`)
- Prefect installed (`pip install prefect`)
- AWS credentials configured for S3 access
- An LLM API key (e.g., OpenAI, Hugging Face)
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
   export LLM_API_KEY=<your-llm-api-key>
   ```

## Pipeline Code

The pipeline is defined in `llm_tabular_pipeline.py`. Below is an example implementation:

```python
from prefect import flow, task
import bauplan
import pandas as pd
from openai import OpenAI
import os

@task
def read_data(source: str) -> pd.DataFrame:
    # Read tabular data from source (e.g., S3)
    df = bauplan.read(source)
    return df

@task
def process_with_llm(df: pd.DataFrame) -> pd.DataFrame:
    # Initialize LLM client
    client = OpenAI(api_key=os.getenv("LLM_API_KEY"))
    
    # Example: Classify sentiment in 'comment' column
    df['sentiment'] = df['comment'].apply(
        lambda x: client.completions.create(
            model="text-davinci-003",
            prompt=f"Classify the sentiment of this comment as Positive, Negative, or Neutral: {x}",
            max_tokens=10
        ).choices[0].text.strip()
    )
    return df

@task
def write_data(df: pd.DataFrame, destination: str):
    # Write results to destination
    bauplan.write(df, destination)

@flow(name="llm-tabular-pipeline")
def llm_tabular_pipeline(source: str, destination: str):
    df = read_data(source)
    processed_df = process_with_llm(df)
    write_data(processed_df, destination)

if __name__ == "__main__":
    llm_tabular_pipeline(
        source="s3://my-bucket/input/data.csv",
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
   prefect deploy -f llm_tabular_pipeline.py
   ```

3. **Run the pipeline**:
   Execute the pipeline locally or schedule it:
   ```bash
   prefect run -n llm-tabular-pipeline
   ```

## Monitoring

- Use the Prefect UI to monitor pipeline runs: `http://localhost:4200`
- Check Bauplan logs for data processing details:
  ```bash
  bauplan logs
  ```

## Notes

- Replace `text-davinci-003` with your preferred LLM model (e.g., `gpt-3.5-turbo` for OpenAI).
- For large datasets, batch LLM requests to avoid rate limits and reduce costs.
- Ensure your input data format (e.g., CSV, Parquet) is supported by Bauplan.
- Verify S3 bucket permissions for read/write access.

For more details, refer to the [Bauplan documentation](https://docs.bauplanlabs.com/) or [Prefect documentation](https://docs.prefect.io/).