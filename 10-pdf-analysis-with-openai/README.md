# PDF Analysis with Bauplan, Prefect, and OpenAI

This example demonstrates how to build a data processing pipeline that extracts text from PDF files and analyzes it using OpenAI's large language model (LLM) with [Bauplan](https://docs.bauplanlabs.com/) and [Prefect](https://docs.prefect.io/). The pipeline reads PDFs from a source, extracts text, processes it with an LLM (e.g., for summarization or entity extraction), and writes the results to a destination.

## Overview

The pipeline performs the following steps:
1. **Read**: Load PDF files from a source (e.g., S3 bucket).
2. **Extract**: Extract text from PDFs using a library like `PyPDF2`.
3. **Process**: Use OpenAI's LLM to analyze the extracted text (e.g., summarize content, extract entities).
4. **Write**: Save the processed results to a destination (e.g., S3 bucket or database).

Bauplan manages data orchestration and execution, while Prefect handles workflow scheduling and monitoring.

## Prerequisites

- Bauplan CLI installed (`pip install bauplan`)
- Prefect installed (`pip install prefect`)
- AWS credentials configured for S3 access
- OpenAI API key
- Python libraries: `PyPDF2`, `pandas`
- A Bauplan account and API key

## Setup

1. **Clone the example repository**:
   ```bash
   git clone https://github.com/BauplanLabs/wap_with_bauplan_and_prefect.git
   cd wap_with_bauplan_and_prefect
   ```

2. **Install dependencies**:
   ```bash
   pip install bauplan prefect PyPDF2 pandas openai
   ```

3. **Configure environment variables**:
   Set the following in a `.env` file or your shell:
   ```bash
   export BAUPLAN_API_KEY=<your-api-key>
   export AWS_ACCESS_KEY_ID=<your-access-key>
   export AWS_SECRET_ACCESS_KEY=<your-secret-key>
   export OPENAI_API_KEY=<your-openai-api-key>
   ```

## Pipeline Code

The pipeline is defined in `pdf_analysis_pipeline.py`. Below is an example implementation:

```python
from prefect import flow, task
import bauplan
import pandas as pd
from openai import OpenAI
import PyPDF2
import os

@task
def read_pdf(source: str) -> list:
    # Read PDF files from source (e.g., S3)
    pdf_files = bauplan.read(source, return_type="file_list")
    return pdf_files

@task
def extract_text(pdf_files: list) -> pd.DataFrame:
    # Extract text from PDFs
    data = []
    for pdf_path in pdf_files:
        with open(pdf_path, 'rb') as file:
            reader = PyPDF2.PdfReader(file)
            text = ""
            for page in reader.pages:
                text += page.extract_text() or ""
            data.append({"filename": pdf_path, "text": text})
    return pd.DataFrame(data)

@task
def process_with_llm(df: pd.DataFrame) -> pd.DataFrame:
    # Initialize OpenAI client
    client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
    
    # Example: Summarize text
    df['summary'] = df['text'].apply(
        lambda x: client.completions.create(
            model="text-davinci-003",
            prompt=f"Summarize this text in 100 words or less: {x[:4000]}",  # Truncate to avoid token limits
            max_tokens=100
        ).choices[0].text.strip()
    )
    return df

@task
def write_data(df: pd.DataFrame, destination: str):
    # Write results to destination
    bauplan.write(df, destination)

@flow(name="pdf-analysis-pipeline")
def pdf_analysis_pipeline(source: str, destination: str):
    pdf_files = read_pdf(source)
    df = extract_text(pdf_files)
    processed_df = process_with_llm(df)
    write_data(processed_df, destination)

if __name__ == "__main__":
    pdf_analysis_pipeline(
        source="s3://my-bucket/pdfs/",
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
   prefect deploy -f pdf_analysis_pipeline.py
   ```

3. **Run the pipeline**:
   Execute the pipeline locally or schedule it:
   ```bash
   prefect run -n pdf-analysis-pipeline
   ```

## Monitoring

- Use the Prefect UI to monitor pipeline runs: `http://localhost:4200`
- Check Bauplan logs for data processing details:
  ```bash
  bauplan logs
  ```

## Notes

- Replace `text-davinci-003` with a newer model like `gpt-3.5-turbo` if preferred.
- Handle large PDFs by chunking text to stay within LLM token limits.
- Ensure `PyPDF2` can handle your PDF formats; for complex PDFs, consider alternatives like `pdfplumber`.
- Verify S3 bucket permissions for read/write access.
- For production, batch API calls to OpenAI to optimize costs and avoid rate limits.

For more details, refer to the [Bauplan documentation](https://docs.bauplanlabs.com/), [Prefect documentation](https://docs.prefect.io/), or [OpenAI API documentation](https://platform.openai.com/docs/).