# Bauplan Best Practices

This document contains a collection of best practices, providing guidance on using Bauplan effectively for data processing pipelines. Each entry includes a best practice and associated metadata such as tags, type, difficulty, and preferred tool.

---

## Best Practice 1: Materialization Strategy

```python
@bauplan.model(materialization_strategy='REPLACE')
```

- **Tags**: materialization, model_definition
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Best Practice 2: Isolated Python Environments

In Bauplan, each model runs in its own isolated Python environment. You can specify a different interpreter version and dependencies per function.

- **Tags**: env_setup, environment_isolation, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Best Practice 3: Document Table Type

It's good practice to comment whether a table is Arrow or converted to Pandas. Readers should immediately understand the table type they are working with.

- **Tags**: documentation, developer_experience
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Best Practice 4: Ephemeral Outputs

If you don't specify a materialization_strategy, Bauplan treats the output as ephemeral. It will not persist automatically unless requested.

- **Tags**: materialization, pipeline_control
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Best Practice 5: DAG Construction

By passing previous model outputs as inputs into new models, Bauplan automatically constructs a Directed Acyclic Graph (DAG) of transformations.

- **Tags**: pipeline_design, dag_construction, dependency_management
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Best Practice 6: Filter Pushdown

Push filtering conditions down to S3 scans whenever possible. This minimizes data read and speeds up the pipeline significantly.

- **Tags**: filter_pushdown, performance_optimization
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Best Practice 7: Combine Chunks After Joins

Apply `.combine_chunks()` after Arrow table joins to optimize memory layout and improve query and computation performance downstream.

- **Tags**: arrow, performance_optimization, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Best Practice 8: Filter Pushdown for S3

Apply filter pushdown when scanning S3 datasets to minimize data transfer and processing load. Use WHERE clauses on timestamp or numeric ranges.

- **Tags**: arrow, filter_pushdown, s3_optimization
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Best Practice 9: Early Data Quality Checks

Use Bauplan expectations at early stages of pipelines to validate input data quality before heavy computation or materialization.

- **Tags**: expectation, data_quality, pipeline_validation
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Best Practice 10: Persist ML Models

Persist trained machine learning models immediately after training (not after evaluation) to ensure reproducibility and avoid recomputation.

- **Tags**: ml_pipeline, model_persistence, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Best Practice 11: Fixed Random Seeds

When splitting datasets into train, validation, and test sets, ensure that random seeds are fixed to maintain reproducibility across runs.

- **Tags**: ml_pipeline, dataset_split, reproducibility
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Best Practice 12: Update Downstream Dependencies

When materializing models using `materialization_strategy='REPLACE'`, ensure downstream dependencies are updated to reference the latest snapshot.

- **Tags**: materialization, iceberg, data_management
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Best Practice 13: Explicit ORDER BY in DuckDB

In DuckDB queries, prefer using explicit `ORDER BY` when calculating aggregations to avoid random ordering in result sets.

- **Tags**: duckdb, sql_best_practice, aggregation
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Best Practice 14: Dedicated Ingestion Branch

Always create a dedicated ingestion branch before importing new data into an Iceberg table. This isolates changes and enables safe testing.

- **Tags**: branching, data_ingestion, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Best Practice 15: Post-Ingestion Quality Checks

Run data quality checks after ingestion but before merging branches to avoid promoting invalid or corrupt data into production tables.

- **Tags**: data_validation, quality_checks, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Best Practice 16: Clean Up Ingestion Branches

Always clean up ingestion branches after merging or if the flow fails, to avoid cluttering the catalog with abandoned branches.

- **Tags**: branch_management, cleanup, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Best Practice 17: Prefect Transactions for Ingestion

Use Prefect transactions to encapsulate ingestion workflows, ensuring that failure scenarios automatically trigger rollback mechanisms.

- **Tags**: prefect, transactions, best_practice
- **Type**: best_practice
- **Difficulty**: medium
- **Preferred Tool**: prefect

---

## Best Practice 18: WAP Pattern for Ingestion

When ingesting new data, always use a dedicated branch (WAP pattern) before merging into the main development branch. This isolates ingestion errors.

- **Tags**: branching, data_ingestion, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Best Practice 19: Sync Dashboard Tables

After ingestion, immediately run analytics model updates to keep dashboard tables synchronized with the latest source data.

- **Tags**: analytics, model_update, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Best Practice 20: Retry Ingestion Steps

Use Prefect transactions or retry flows around ingestion steps to recover automatically from transient S3 or ingestion errors.

- **Tags**: prefect, transactions, best_practice
- **Type**: best_practice
- **Difficulty**: medium
- **Preferred Tool**: prefect

---

## Best Practice 21: Validate Synthetic Data

Always validate synthetic event generation pipelines to maintain schema consistency with production tables during tests.

- **Tags**: synthetic_data, schema_validation, best_practice
- **Type**: best_practice
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Best Practice 22: Reuse Cleaning Functions

Reuse cleaning and serialization functions across multiple models (Walmart and Amazon) to maintain consistency and avoid duplication.

- **Tags**: data_cleaning, reusability, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Best Practice 23: Preprocess for LLM Matching

Use LLMs for matching only after thorough data cleaning and feature normalization to reduce hallucinations and improve matching accuracy.

- **Tags**: llm_matching, data_preprocessing, best_practice
- **Type**: best_practice
- **Difficulty**: medium
- **Preferred Tool**: openai

---

## Best Practice 24: Control LLM API Costs

Limit the number of LLM calls using a parameter like `max_k` to control API usage costs during batch predictions.

- **Tags**: llm_matching, api_cost_management, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: openai

---

## Best Practice 25: Materialize Match Results

Materialize match results using `materialization_strategy='REPLACE'` to ensure a clean, updated prediction table without duplications.

- **Tags**: materialization, iceberg, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Best Practice 26: Materialize Embedding Tables

Always materialize final embedding tables with `materialization_strategy='REPLACE'` to ensure a clean snapshot of latest training results.

- **Tags**: materialization, embedding, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Best Practice 27: Validate TSNE Output

Use TSNE dimensionality reduction carefully: always cross-validate that output dimensions match your downstream visualization expectations.

- **Tags**: tsne, visualization, best_practice
- **Type**: best_practice
- **Difficulty**: medium
- **Preferred Tool**: scikit_learn

---

## Best Practice 28: Clean Input for Word2Vec

Train Word2Vec skipgram models only after ensuring your input sequences are clean and logically ordered (e.g., by track position in playlists).

- **Tags**: word2vec, embedding_training, best_practice
- **Type**: best_practice
- **Difficulty**: medium
- **Preferred Tool**: gensim

---

## Best Practice 29: Validate MongoDB Connection

Always validate MongoDB connection success before attempting vector uploads to prevent silent failures during ingestion.

- **Tags**: mongodb, connection_validation, best_practice
- **Type**: best_practice
- **Difficulty**: easy
- **Preferred Tool**: mongodb

---

This collection of best practices provides actionable guidance for optimizing Bauplan pipelines, improving performance, ensuring data quality, and managing dependencies effectively.