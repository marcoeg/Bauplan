# Retrieval-Augmented Generation (RAG) with Bauplan and Prefect

This example demonstrates how to build a Retrieval-Augmented Generation (RAG) pipeline using [Bauplan](https://docs.bauplanlabs.com/) and [Prefect](https://docs.prefect.io/). The pipeline processes documents, generates embeddings, stores them in a vector database, and retrieves relevant documents to augment responses from a large language model (LLM).

## Overview

The RAG pipeline performs the following steps:
1. **Read**: Load documents (e.g., text files, PDFs) from a source like S3.
2. **Process**: Generate embeddings for the documents using a pre-trained model.
3. **Store**: Save embeddings and metadata in a vector database (e.g., FAISS or Pinecone).
4. **Retrieve**: Query the vector database to retrieve relevant documents based on a user query.
5. **Generate**: Use an LLM to generate a response augmented with retrieved documents.

Bauplan handles data orchestration and execution, while Prefect manages workflow scheduling and monitoring.

## Prerequisites

- Bauplan CLI installed (`pip install bauplan`)
- Prefect installed (`pip install prefect`)
- AWS credentials configured for S3 access
- An LLM API key (e.g., OpenAI)
- A vector database (e.g., FAISS or Pinecone) with appropriate credentials
- Python libraries: `sentence-transformers`, `pandas`, `faiss-cpu` (or Pinecone client)
- A Bauplan account and API key

## Setup

1. **Clone the example repository**:
   ```bash
   git clone https://github.com/BauplanLabs/wap_with_bauplan_and_prefect.git
   cd wap_with_bauplan_and_prefect
   ```

2. **Install dependencies**:
   ```bash
   pip install bauplan prefect sentence-transformers pandas faiss-cpu openai
   ```

3. **Configure environment variables**:
   Set the following in a `.env` file or your shell:
   ```bash
   export BAUPLAN_API_KEY=<your-api-key>
   export AWS_ACCESS_KEY_ID=<your-access-key>
   export AWS_SECRET_ACCESS_KEY=<your-secret-key>
   export OPENAI_API_KEY=<your-openai-api-key>
   export VECTOR_DB_URL=<your-vector-db-url>  # Optional, depending on vector DB
   ```

4. **Set up the vector database**:
   - For FAISS, initialize a local index (handled in the code).
   - For Pinecone, create an index via their dashboard and note the API key and environment.

## Pipeline Code

The pipeline is defined in `rag_pipeline.py`. Below is an example implementation using FAISS as the vector database:

```python
from prefect import flow, task
import bauplan
import pandas as pd
from sentence_transformers import SentenceTransformer
from openai import OpenAI
import faiss
import numpy as np
import os

@task
def read_documents(source: str) -> pd.DataFrame:
    # Read documents from source (e.g., S3)
    documents = bauplan.read(source)
    return pd.DataFrame({"content": documents})

@task
def generate_embeddings(df: pd.DataFrame) -> tuple:
    # Load pre-trained embedding model
    model = SentenceTransformer('all-MiniLM-L6-v2')
    # Generate embeddings
    embeddings = model.encode(df['content'].tolist(), show_progress_bar=True)
    df['embedding'] = embeddings.tolist()
    return df, embeddings

@task
def store_in_vector_db(df: pd.DataFrame, embeddings: np.ndarray, index_path: str):
    # Initialize FAISS index
    dimension = embeddings.shape[1]
    index = faiss.IndexFlatL2(dimension)
    # Add embeddings to index
    index.add(embeddings)
    # Save index
    faiss.write_index(index, index_path)
    # Save metadata
    df[['content']].to_csv(index_path + "_metadata.csv", index=False)

@task
def retrieve_documents(query: str, index_path: str, top_k: int = 5) -> list:
    # Load embedding model and FAISS index
    model = SentenceTransformer('all-MiniLM-L6-v2')
    index = faiss.read_index(index_path)
    # Encode query
    query_embedding = model.encode([query])[0]
    # Search for top_k similar documents
    distances, indices = index.search(np.array([query_embedding]), top_k)
    # Load metadata
    metadata = pd.read_csv(index_path + "_metadata.csv")
    return [metadata.iloc[idx]['content'] for idx in indices[0]]

@task
def generate_response(query: str, documents: list) -> str:
    # Initialize OpenAI client
    client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
    # Combine documents and query for LLM
    context = "\n".join(documents)
    prompt = f"Based on the following context, answer the query: {query}\n\nContext:\n{context}"
    # Generate response
    response = client.completions.create(
        model="text-davinci-003",
        prompt=prompt,
        max_tokens=200
    ).choices[0].text.strip()
    return response

@flow(name="rag-pipeline")
def rag_pipeline(source: str, index_path: str, query: str):
    df = read_documents(source)
    df, embeddings = generate_embeddings(df)
    store_in_vector_db(df, embeddings, index_path)
    documents = retrieve_documents(query, index_path)
    response = generate_response(query, documents)
    return response

if __name__ == "__main__":
    rag_pipeline(
        source="s3://my-bucket/documents/",
        index_path="faiss_index",
        query="What is the main topic of the documents?"
    )
```

## Running the Pipeline

1. **Start the Prefect server**:
   ```bash
   prefect server start
   ```

2. **Deploy the pipeline**:
   ```bash
   prefect deploy -f rag_pipeline.py
   ```

3. **Run the pipeline**:
   Execute the pipeline locally or schedule it:
   ```bash
   prefect run -n rag-pipeline
   ```

## Monitoring

- Use the Prefect UI to monitor pipeline runs: `http://localhost:4200`
- Check Bauplan logs for data processing details:
  ```bash
  bauplan logs
  ```

## Notes

- Replace `text-davinci-003` with a newer model like `gpt-3.5-turbo` for better performance.
- Use a more robust vector database like Pinecone for production-scale applications.
- Optimize embedding generation by batching for large document sets.
- Ensure S3 bucket permissions allow read access for documents.
- For FAISS, store the index in a persistent location (e.g., S3) to reuse it across runs.

For more details, refer to the [Bauplan documentation](https://docs.bauplanlabs.com/), [Prefect documentation](https://docs.prefect.io/), [Sentence Transformers documentation](https://www.sbert.net/), or [OpenAI API documentation](https://platform.openai.com/docs/).