# LLM Data Processing with Bauplan and Prefect

This example demonstrates how to build a data processing pipeline that leverages a large language model (LLM) using [Bauplan](https://docs.bauplanlabs.com/) and [Prefect](https://docs.prefect.io/). The pipeline reads input data, processes it with an LLM, and writes the results to a destination.

## Overview

The pipeline performs the following steps:
1. **Read**: Load data from a source (e.g., S3 bucket).
2. **Process**: Use an LLM to analyze or transform the data (e.g., text summarization, sentiment analysis).
3. **Write**: Save the processed results to a destination (e.g., S3 bucket or database).

Bauplan manages data orchestration and execution, while Prefect handles workflow scheduling and monitoring.

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

The pipeline is defined in `llm_pipeline.py`. Below is an example implementation:

```python
from prefect import flow, task
import bauplan
import pandas as pd
from openai import OpenAI

@task
def read_data(source: str) -> pd.DataFrame:
    # Read data from source (e.g., S3)
    df = bauplan.read(source)
    return df

@task
def process_with_llm(df: pd.DataFrame) -> pd.DataFrame:
    # Initialize LLM client
    client = OpenAI(api_key=os.getenv("LLM_API_KEY"))
    
    # Example: Summarize text in 'content' column
    df['summary'] = df['content'].apply(
        lambda x: client.completions.create(
            model="text-davinci-003",
            prompt=f"Summarize this text: {x}",
            max_tokens=100
        ).choices[0].text.strip()
    )
    return df

@task
def write_data(df: pd.DataFrame, destination: str):
    # Write results to destination
    bauplan.write(df, destination)

@flow(name="llm-pipeline")
def llm_pipeline(source: str, destination: str):
    df = read_data(source)
    processed_df = process_with_llm(df)
    write_data(processed_df, destination)

if __name__ == "__main__":
    llm_pipeline(
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
   prefect deploy -f llm_pipeline.py
   ```

3. **Run the pipeline**:
   Execute the pipeline locally or schedule it:
   ```bash
   prefect run -n llm-pipeline
   ```

## Monitoring

- Use the Prefect UI to monitor pipeline runs: `http://localhost:4200`
- Check Bauplan logs for data processing details:
  ```bash
  bauplan logs
  ```

## Notes

- Replace `text-davinci-003` with your preferred LLM model (e.g., `gpt-3.5-turbo` for OpenAI).
- Ensure your LLM API key has sufficient credits and permissions.
- For large datasets, consider batching LLM requests to manage costs and rate limits.
- Verify S3 bucket permissions for read/write access.

For more details, refer to the [Bauplan documentation](https://docs.bauplanlabs.com/) or [Prefect documentation](https://docs.prefect.io/).