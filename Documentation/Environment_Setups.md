# Bauplan Environment Setups

This document contains a collection of environment setup examples, detailing Python environments and dependencies for Bauplan pipelines. Each entry includes an environment declaration or description and associated metadata such as tags, type, difficulty, and preferred tool.

---

## Environment Setup 1: Pandas and NumPy

```python
@bauplan.python('3.11', pip={'pandas': '1.5.3', 'numpy': '1.23.2'})
```

- **Tags**: env_setup, python_env, dependencies
- **Type**: env_setup
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Environment Setup 2: Pandas 2.2.0

```python
@bauplan.python('3.11', pip={'pandas': '2.2.0'})
```

- **Tags**: env_setup, python_version, pandas
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Environment Setup 3: Pandas and Scikit-Learn

```python
@bauplan.python('3.10', pip={'pandas': '1.5.3', 'scikit-learn': '1.3.2'})
```

- **Tags**: env_setup, python_version, scikit_learn
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Environment Setup 4: DuckDB 0.10.3

```python
@bauplan.python('3.11', pip={'duckdb': '0.10.3'})
```

- **Tags**: env_setup, python_version, duckdb
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Environment Setup 5: Default Arrow Environment

Default environment for pure Arrow table transformations in Bauplan is minimal (no pip dependencies needed unless specific libraries like Pandas are explicitly used).

- **Tags**: env_setup, default_behavior, arrow
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Environment Setup 6: WAP Ingestion Flow

Minimum runtime environment for this WAP ingestion flow: Python 3.10+ with Bauplan SDK and Prefect 3 installed. No additional pip dependencies required for basic flow operation.

- **Tags**: env_setup, python_env, dependencies
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: python

---

## Environment Setup 7: DuckDB 1.0.0

```python
@bauplan.python('3.11', pip={'duckdb': '1.0.0'})
```

- **Tags**: env_setup, duckdb, analytics
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Environment Setup 8: Pandas for Data Processing

```python
@bauplan.python('3.11', pip={'pandas': '2.2.0'})
```

- **Tags**: env_setup, pandas, data_processing
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Environment Setup 9: DuckDB and OpenAI

```python
@bauplan.python('3.11', pip={'duckdb': '1.0.0', 'openai': '1.57.2'})
```

- **Tags**: env_setup, duckdb, openai
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Environment Setup 10: Embedding Pipeline

```python
@bauplan.python('3.11', pip={'gensim': '4.3.3', 'scikit_learn': '1.5.2', 'duckdb': '1.0.0', 'pymongo': '4.10.1'})
```

- **Tags**: env_setup, python_env, embedding_pipeline
- **Type**: env_declaration
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

This collection of environment setups illustrates how to configure Python environments for various Bauplan pipelines, specifying versions and dependencies for tools like Pandas, DuckDB, and OpenAI.