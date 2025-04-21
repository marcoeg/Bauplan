# Embedding-Based Recommender Systems with Bauplan and MongoDB

Recommender systems are at the core of many modern applications, from e-commerce to content streaming. In this blog post, we explore how to build an embedding-based recommender system using [Bauplan](https://docs.bauplanlabs.com/) for data orchestration and [MongoDB](https://www.mongodb.com/) for storing and querying embeddings. This approach leverages machine learning to generate item embeddings and recommend similar items based on user preferences.

## Introduction

Embedding-based recommender systems represent items (e.g., products, movies) as dense vectors in a high-dimensional space. By computing similarity between these vectors, we can recommend items that are "close" to a user's interests. This post demonstrates how to:
- Generate item embeddings using a pre-trained model.
- Store embeddings in MongoDB with vector search capabilities.
- Build a data pipeline with Bauplan to orchestrate the process.

## Prerequisites

- Bauplan CLI installed (`pip install bauplan`)
- MongoDB Atlas account with vector search enabled
- Python libraries: `pymongo`, `sentence-transformers`, `pandas`
- AWS S3 access for input/output data
- A Bauplan account and API key

## Setup

1. **Clone the example repository**:
   ```bash
   git clone https://github.com/BauplanLabs/wap_with_bauplan_and_prefect.git
   cd wap_with_bauplan_and_prefect
   ```

2. **Install dependencies**:
   ```bash
   pip install bauplan pymongo sentence-transformers pandas
   ```

3. **Configure environment variables**:
   Set the following in a `.env` file or your shell:
   ```bash
   export BAUPLAN_API_KEY=<your-api-key>
   export AWS_ACCESS_KEY_ID=<your-access-key>
   export AWS_SECRET_ACCESS_KEY=<your-secret-key>
   export MONGODB_URI=<your-mongodb-atlas-uri>
   ```

4. **Set up MongoDB**:
   - Create a MongoDB Atlas cluster.
   - Enable vector search and create a collection (e.g., `items`) for storing embeddings.
   - Create a vector search index on the embedding field.

## Pipeline Overview

The pipeline performs the following steps:
1. **Read**: Load item metadata (e.g., product descriptions) from S3.
2. **Generate Embeddings**: Use a pre-trained model (e.g., Sentence Transformers) to create embeddings.
3. **Store**: Save embeddings and metadata to MongoDB.
4. **Query**: Perform vector search to recommend similar items.

## Pipeline Code

The pipeline is implemented in `recommender_pipeline.py`. Below is an example:

```python
from prefect import flow, task
import bauplan
import pandas as pd
from sentence_transformers import SentenceTransformer
from pymongo import MongoClient

@task
def read_data(source: str) -> pd.DataFrame:
    # Read item metadata from S3
    df = bauplan.read(source)
    return df

@task
def generate_embeddings(df: pd.DataFrame) -> pd.DataFrame:
    # Load pre-trained model
    model = SentenceTransformer('all-MiniLM-L6-v2')
    # Generate embeddings for item descriptions
    df['embedding'] = df['description'].apply(lambda x: model.encode(x).tolist())
    return df

@task
def store_embeddings(df: pd.DataFrame, mongodb_uri: str, db_name: str, collection_name: str):
    # Connect to MongoDB
    client = MongoClient(mongodb_uri)
    collection = client[db_name][collection_name]
    # Insert items with embeddings
    records = df.to_dict('records')
    collection.insert_many(records)

@task
def query_recommendations(mongodb_uri: str, db_name: str, collection_name: str, embedding: list, top_k: int = 5):
    # Connect to MongoDB
    client = MongoClient(mongodb_uri)
    collection = client[db_name][collection_name]
    # Perform vector search
    pipeline = [
        {
            "$vectorSearch": {
                "index": "vector_index",
                "path": "embedding",
                "queryVector": embedding,
                "numCandidates": 100,
                "limit": top_k
            }
        }
    ]
    results = collection.aggregate(pipeline)
    return list(results)

@flow(name="recommender-pipeline")
def recommender_pipeline(source: str, mongodb_uri: str, db_name: str, collection_name: str):
    df = read_data(source)
    df_with_embeddings = generate_embeddings(df)
    store_embeddings(df_with_embeddings, mongodb_uri, db_name, collection_name)

if __name__ == "__main__":
    recommender_pipeline(
        source="s3://my-bucket/items/",
        mongodb_uri=os.getenv("MONGODB_URI"),
        db_name="recommender_db",
        collection_name="items"
    )
```

## Running the Pipeline

1. **Start the Prefect server**:
   ```bash
   prefect server start
   ```

2. **Deploy the pipeline**:
   ```bash
   prefect deploy -f recommender_pipeline.py
   ```

3. **Run the pipeline**:
   Execute the pipeline locally:
   ```bash
   prefect run -n recommender-pipeline
   ```

## Querying Recommendations

To query recommendations for a specific item:
```python
# Example: Get recommendations for an item
sample_embedding = SentenceTransformer('all-MiniLM-L6-v2').encode("Sample item description").tolist()
results = query_recommendations(
    mongodb_uri=os.getenv("MONGODB_URI"),
    db_name="recommender_db",
    collection_name="items",
    embedding=sample_embedding,
    top_k=5
)
for result in results:
    print(result['name'], result['description'])
```

## Monitoring

- Monitor pipeline runs in the Prefect UI: `http://localhost:4200`
- Check Bauplan logs for processing details:
  ```bash
  bauplan logs
  ```

## Notes

- Ensure the MongoDB vector search index is configured correctly (e.g., `vector_index` on the `embedding` field).
- The `all-MiniLM-L6-v2` model is lightweight but effective. For better accuracy, consider larger models like `paraphrase-mpnet-base-v2`.
- Optimize performance by batching embedding generation for large datasets.
- Verify S3 and MongoDB permissions before running the pipeline.

For more details, refer to the [Bauplan documentation](https://docs.bauplanlabs.com/), [MongoDB vector search guide](https://www.mongodb.com/docs/atlas/atlas-search/vector-search/), or [Sentence Transformers documentation](https://www.sbert.net/).