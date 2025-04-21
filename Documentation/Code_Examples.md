# Bauplan Code Examples

This document contains a collection of code examples, showcasing various data processing tasks using Bauplan. Each example includes a code snippet and associated metadata such as tags, type, difficulty, and preferred tool.

---

## Example 1: Trips and Zones

```python
@bauplan.model()
@bauplan.python('3.11')
def trips_and_zones(
    trips=bauplan.Model(
        'taxi_fhvhv',
        columns=[
            'pickup_datetime', 'dropoff_datetime', 'PULocationID', 'DOLocationID',
            'trip_miles', 'trip_time', 'base_passenger_fare', 'tolls', 'sales_tax', 'tips'
        ],
        filter="pickup_datetime >= '2022-12-15T00:00:00-05:00' AND pickup_datetime < '2023-01-01T00:00:00-05:00'"
    ),
    zones=bauplan.Model('taxi_zones')
):
    pickup_location_table = trips.join(zones, 'PULocationID', 'LocationID').combine_chunks()
    return pickup_location_table
```

- **Tags**: arrow, filter_pushdown, materialization, model_definition
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 2: Normalized Taxi Trips

```python
@bauplan.model(materialization_strategy='REPLACE')
@bauplan.python('3.11', pip={'pandas': '1.5.3', 'numpy': '1.23.2'})
def normalized_taxi_trips(
    data=bauplan.Model('trips_and_zones')
):
    import pandas as pd
    import numpy as np
    import math

    size_in_gb = round(data.nbytes / math.pow(1024, 3), 3)
    print(f"\nThis table is {size_in_gb} GB and has {data.num_rows} rows\n")

    df = data.to_pandas()
    time_filter = pd.to_datetime('2022-01-01').tz_localize('UTC')
    df = df[df['pickup_datetime'] >= time_filter]
    df = df[df['trip_miles'] > 0.0]
    df = df[df['trip_miles'] < 200.0]
    df['log_trip_miles'] = np.log10(df['trip_miles'])
    return df
```

- **Tags**: pandas, data_cleaning, transformation, model_definition
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Example 3: Simple Model Definition

```python
@bauplan.model()
def my_model(data=bauplan.Model('some_table')):
    # Transformation logic here
    return data
```

- **Tags**: model_definition
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 4: Taxi FHVHV Model

```python
trips=bauplan.Model(
    'taxi_fhvhv',
    columns=[
        'pickup_datetime', 'dropoff_datetime', 'PULocationID', 'DOLocationID',
        'trip_miles', 'trip_time', 'base_passenger_fare', 'tolls', 'sales_tax', 'tips'
    ],
    filter="pickup_datetime >= '2022-12-15T00:00:00-05:00' AND pickup_datetime < '2023-01-01T00:00:00-05:00'"
)
```

- **Tags**: filter_pushdown, column_pruning, model_definition
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 5: Trips and Zones (Short)

```python
@bauplan.model() @bauplan.python('3.11') def trips_and_zones(...):
```

- **Tags**: arrow, model_definition, filter_pushdown
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 6: Top Pickup Locations

```python
@bauplan.model(materialization_strategy='REPLACE') @bauplan.python('3.11', pip={'pandas': '2.2.0'}) def top_pickup_locations(...):
```

- **Tags**: pandas, aggregation, materialization
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Example 7: Taxi FHVHV Model (Short)

```python
trips=bauplan.Model('taxi_fhvhv', columns=[...], filter='...')
```

- **Tags**: filter_pushdown, model_definition, performance_optimization
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 8: Join Operation

```python
pickup_location_table = (trips.join(zones, 'PULocationID', 'LocationID').combine_chunks())
```

- **Tags**: arrow, join, performance_optimization
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 9: Materialization Strategy

```python
@bauplan.model(materialization_strategy='REPLACE')
```

- **Tags**: materialization, iceberg
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 10: Arrow to Pandas Conversion

```python
df = data.to_pandas()
```

- **Tags**: arrow, pandas, conversion
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 11: Aggregation and Sorting

```python
top_pickup_table = ( df.groupby([...]) .agg(...) .reset_index() .sort_values(...) )
```

- **Tags**: pandas, aggregation, sorting
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Example 12: Clean Taxi Trips

```python
@bauplan.model()
@bauplan.python('3.11', pip={'pandas': '2.2.0'})
def clean_taxi_trips(data=bauplan.Model('taxi_fhvhv', columns=[...], filter='...')):
    df = data.to_pandas()
    df = df[(df['trip_miles'] > 1.0) & (df['tips'] > 0.0) & (df['base_passenger_fare'] > 1.0)]
    return df
```

- **Tags**: arrow, pandas, model_definition
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 13: Training Dataset Preparation

```python
@bauplan.model()
@bauplan.python('3.10', pip={'pandas': '1.5.3', 'scikit-learn': '1.3.2'})
def training_dataset(data=bauplan.Model('clean_taxi_trips')):
    df = data.to_pandas()
    df = df.dropna()
    df['log_trip_miles'] = np.log10(df['trip_miles'])
    scaler = StandardScaler()
    scaled_features = scaler.fit_transform(df[['log_trip_miles', 'base_passenger_fare', 'trip_time']])
    scaled_df = pd.DataFrame(scaled_features, columns=['log_trip_miles', 'base_passenger_fare', 'trip_time'])
    return scaled_df
```

- **Tags**: pandas, feature_engineering, scaling, model_definition
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Example 14: Train Regression Model

```python
@bauplan.model()
@bauplan.python('3.11', pip={'pandas': '2.2.0', 'scikit-learn': '1.3.2'})
def train_regression_model(data=bauplan.Model('training_dataset')):
    df = data.to_pandas()
    train_set, remaining_set = train_test_split(df, train_size=0.8)
    validation_set, test_set = train_test_split(remaining_set, test_size=0.5)
    reg = LinearRegression().fit(train_set[['log_trip_miles', 'base_passenger_fare', 'trip_time']], train_set['tips'])
    from bauplan.store import save_obj
    save_obj("regression", reg)
    return test_set
```

- **Tags**: pandas, model_training, linear_regression, model_definition
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Example 15: Tip Predictions

```python
@bauplan.model(materialization_strategy='REPLACE')
@bauplan.python('3.11', pip={'scikit-learn': '1.3.2', 'pandas': '2.1.0'})
def tip_predictions(data=bauplan.Model('train_regression_model')):
    from bauplan.store import load_obj
    reg = load_obj("regression")
    test_set = data.to_pandas()
    y_hat = reg.predict(test_set[['log_trip_miles', 'base_passenger_fare', 'trip_time']])
    prediction_df = test_set.copy()
    prediction_df['predictions'] = y_hat
    return prediction_df
```

- **Tags**: pandas, model_inference, materialization, model_definition
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Example 16: Taxi FHVHV Model (Date Range)

```python
data=bauplan.Model('taxi_fhvhv', columns=['pickup_datetime', 'dropoff_datetime', 'PULocationID', 'DOLocationID', 'trip_miles', 'trip_time', 'base_passenger_fare', 'tolls', 'sales_tax', 'tips'], filter="pickup_datetime >= '2023-01-01T00:00:00-05:00' AND pickup_datetime < '2023-03-31T00:00:00-05:00'")
```

- **Tags**: filter_pushdown, column_pruning
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 17: Arrow to Pandas Conversion (Repeated)

```python
df = data.to_pandas()
```

- **Tags**: arrow, pandas, conversion
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 18: Feature Scaling

```python
scaler = StandardScaler()
scaled_features = scaler.fit_transform(features)
```

- **Tags**: feature_scaling, standardization
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Example 19: Train-Test Split

```python
train_set, remaining_set = train_test_split(df, train_size=0.8, random_state=42)
validation_set, test_set = train_test_split(remaining_set, test_size=0.5, random_state=42)
```

- **Tags**: train_test_split, ml_pipeline
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 20: Linear Regression Training

```python
reg = LinearRegression().fit(X_train, y_train)
```

- **Tags**: model_training, linear_regression
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 21: Save Model

```python
from bauplan.store import save_obj
save_obj("regression", reg)
```

- **Tags**: model_persistence, bauplan_store
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 22: Load Model

```python
from bauplan.store import load_obj
reg = load_obj("regression")
```

- **Tags**: model_loading, bauplan_store
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 23: Model Evaluation

```python
reg.score(X_test, y_test)
```

- **Tags**: model_evaluation, regression
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 24: Normalized Taxi Trips with Join

```python
@bauplan.model()
@bauplan.python('3.11', pip={'pandas': '2.2.0'})
def normalized_taxi_trips(trips=bauplan.Model('taxi_fhvhv', columns=[...], filter='...'), zones=bauplan.Model('taxi_zones')):
    pickup_location_table = trips.join(zones, 'PULocationID', 'LocationID').combine_chunks()
    return pickup_location_table
```

- **Tags**: arrow, model_definition, filter_pushdown
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 25: Taxi Trip Waiting Times

```python
@bauplan.model(materialization_strategy='REPLACE')
@bauplan.python('3.11', pip={'pandas': '2.2.0'})
def taxi_trip_waiting_times(data=bauplan.Model('normalized_taxi_trips')):
    waiting_time_min = pc.minutes_between(data['request_datetime'], data['on_scene_datetime'])
    data = data.append_column('waiting_time_minutes', waiting_time_min)
    return data
```

- **Tags**: arrow, time_difference, column_append, model_definition
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 26: Zone Average Waiting Times

```python
@bauplan.model(materialization_strategy='REPLACE')
@bauplan.python('3.11', pip={'duckdb': '0.10.3'})
def zone_avg_waiting_times(taxi_trip_waiting_times=bauplan.Model('taxi_trip_waiting_times')):
    sql_query = "SELECT Borough, Zone, AVG(waiting_time_minutes) AS avg_waiting_time FROM taxi_trip_waiting_times GROUP BY Borough, Zone ORDER BY avg_waiting_time DESC;"
    data = duckdb.sql(sql_query).arrow()
    return data
```

- **Tags**: aggregation, materialization, model_definition
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: duckdb

---

## Example 27: Null Values Expectation Test

```python
@bauplan.expectation()
@bauplan.python('3.11')
def test_null_values_on_scene_datetime(data=bauplan.Model('normalized_taxi_trips')):
    column_to_check = 'on_scene_datetime'
    _is_expectation_correct = expect_column_no_nulls(data, column_to_check)
    assert _is_expectation_correct, f"expectation test failed: we expected {column_to_check} to have no null values"
    return _is_expectation_correct
```

- **Tags**: expectation, data_quality, null_check
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 28: Join Operation (Repeated)

```python
pickup_location_table = trips.join(zones, 'PULocationID', 'LocationID').combine_chunks()
```

- **Tags**: arrow, join, performance_optimization
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 29: Taxi FHVHV Model (Columns)

```python
trips=bauplan.Model(
    'taxi_fhvhv',
    columns=[
        'PULocationID', 'request_datetime', 'on_scene_datetime', 'pickup_datetime', 'dropoff_datetime'
    ],
    filter="pickup_datetime >= '2022-12-01T00:00:00-05:00' AND pickup_datetime < '2023-01-01T00:00:00-05:00'"
)
```

- **Tags**: filter_pushdown, column_pruning, performance_optimization
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 30: Time Difference Calculation

```python
waiting_time_min = pc.minutes_between(data['request_datetime'], data['on_scene_datetime'])
```

- **Tags**: arrow, time_difference, pyarrow_compute
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 31: Append Column

```python
data = data.append_column('waiting_time_minutes', waiting_time_min)
```

- **Tags**: arrow, column_append, schema_update
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 32: SQL Aggregation Query

```python
sql_query = """
SELECT Borough, Zone, AVG(waiting_time_minutes) AS avg_waiting_time
FROM taxi_trip_waiting_times
GROUP BY Borough, Zone
ORDER BY avg_waiting_time DESC;
"""
```

- **Tags**: duckdb, sql_template, aggregation
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Example 33: DuckDB SQL to Arrow

```python
data = duckdb.sql(sql_query).arrow()
```

- **Tags**: duckdb, arrow_integration, query_execution
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Example 34: Expectation Decorator

```python
@bauplan.expectation()
@bauplan.python('3.11')
```

- **Tags**: expectation, pipeline_quality
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 35: Null Check Expectation

```python
_is_expectation_correct = expect_column_no_nulls(data, 'on_scene_datetime')
```

- **Tags**: expectation, standard_expectations, null_check
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 36: Assertion for Expectation

```python
assert _is_expectation_correct, f"expectation test failed: we expected on_scene_datetime to have no null values"
```

- **Tags**: expectation, data_validation, error_handling
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Example 37: Purchase Sessions

```python
@bauplan.python('3.11', pip={'duckdb': '1.0.0'})
@bauplan.model(materialization_strategy='REPLACE')
def purchase_sessions(ecommerce_clean=bauplan.Model('ecommerce_clean')):
    import duckdb
    con = duckdb.connect()
    query = "SELECT user_session, event_hour, count(*) FROM ecommerce_clean WHERE event_type = 'purchase' GROUP BY 1, 2"
    data = con.execute(query).arrow()
    return data
```

- **Tags**: duckdb, sql_model, analytics
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Example 38: Metrics Orders

```python
@bauplan.python('3.11', pip={'duckdb': '1.0.0'})
@bauplan.model(materialization_strategy='REPLACE')
def metrics_orders(ecommerce_clean=bauplan.Model('ecommerce_clean')):
    import duckdb
    con = duckdb.connect()
    query = "SELECT brand, COUNT(product_id)::FLOAT/count(distinct user_session) AS products_per_user_session, round(sum(price),2) AS revenue FROM ecommerce_clean WHERE event_type = 'purchase' GROUP BY 1 ORDER BY 3 DESC"
    data = con.execute(query).arrow()
    return data
```

- **Tags**: duckdb, sql_model, analytics
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Example 39: Ecommerce Metrics Base

```python
@bauplan.python('3.11', pip={'duckdb': '1.0.0'})
@bauplan.model(materialization_strategy='REPLACE')
def ecommerce_metrics_base(purchase_sessions=bauplan.Model('purchase_sessions'), ecommerce_clean=bauplan.Model('ecommerce_clean')):
    import duckdb
    con = duckdb.connect()
    query = "SELECT event_hour, brand, SUM(CASE WHEN event_type = 'view' THEN 1 ELSE 0 END) AS views FROM ecommerce_clean GROUP BY 1,2"
    data = con.execute(query).arrow()
    return data
```

- **Tags**: duckdb, sql_model, analytics, join
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: duckdb

---

## Example 40: Create Public S3 Bucket

```python
def create_public_bucket(s3_client, bucket_name: str):
    if not does_bucket_exist(s3_client, bucket_name):
        s3_client.create_bucket(Bucket=bucket_name)
    bucket_policy = {...}
    s3_client.put_bucket_policy(Bucket=bucket_name, Policy=json.dumps(bucket_policy))
```

- **Tags**: s3, bucket_management, policy
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: aws

---

## Example 41: Random Events Sampling

```python
def get_random_events_from_source_table(client: bauplan.Client, namespace: str, branch: str, n: int = 2_000_000):
    result = client.query("SELECT * FROM ecommerce_clean WHERE event_hour BETWEEN ...").to_pandas()
    return result.sample(min(n, len(result)))
```

- **Tags**: data_sampling, bauplan, synthetic_data
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Example 42: Transaction Block

```python
with transaction():
    create_data_in_ingestion_bucket(...)
    ingest_on_a_branch(...)
    update_dashboard_tables(...)
```

- **Tags**: prefect, transaction, pipeline_flow
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: prefect

---

## Example 43: Data Ingestion

```python
client.create_branch(branch=ingest_branch, from_ref=dev_branch)
import_state = client.import_data(table='ecommerce_clean', search_uri='...', branch=ingest_branch, namespace=namespace)
if import_state.error:
    raise Exception("Error during ingestion")
```

- **Tags**: bauplan, data_ingestion, branching
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Example 44: Command Line Arguments

```python
parser = argparse.ArgumentParser()
parser.add_argument('--username', type=str)
parser.add_argument('--namespace', type=str)
parser.add_argument('--bucket_name', type=str)
parser.add_argument('--dev_branch', type=str)
args = parser.parse_args()
analytics_with_bauplan(...)
```

- **Tags**: cli, argparse, pipeline_execution
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: python

---

## Example 45: Serialized Walmart Products

```python
@bauplan.python('3.11', pip={'pandas': '2.2.0'}) @bauplan.model() def serialized_walmart_products(...)
```

- **Tags**: bauplan, data_cleaning, serialization
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 46: Serialized Amazon Products

```python
@bauplan.python('3.11', pip={'pandas': '2.2.0'}) @bauplan.model() def serialized_amazon_products(...)
```

- **Tags**: bauplan, data_cleaning, serialization
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 47: Product LLM Matches

```python
@bauplan.python('3.11', pip={'duckdb': '1.0.0', 'openai': '1.57.2'}) @bauplan.model(materialization_strategy='REPLACE', internet_access=True) def product_llm_matches(...)
```

- **Tags**: bauplan, llm_matching, duckdb_query
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: duckdb

---

## Example 48: Clean and Serialize Products

```python
def clean_and_serialize_products(df): apply math transforms + serialization pattern
```

- **Tags**: pandas, feature_engineering, serialization
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Example 49: Match with LLM

```python
def match_with_llm(_product_a_list, _product_b_list, _llm_client): loop over inputs, format prompts, call OpenAI
```

- **Tags**: openai, prompting, batch_prediction
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: openai

---

## Example 50: OpenAI Prediction Request

```python
def _request_prediction_from_open_ai(prompt, oai_client): call OpenAI and parse yes/no answer
```

- **Tags**: openai, response_parsing, boolean_conversion
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: openai

---

## Example 51: Playlists to Sequences

```python
@bauplan.model() playlists_to_sequences(...): DuckDB query to array-aggregate track_ids per playlist
```

- **Tags**: duckdb, sql_model, playlist_processing
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Example 52: Popular Tracks

```python
@bauplan.model() popular_tracks(...): DuckDB query unnesting track_ids and counting most common tracks
```

- **Tags**: duckdb, sql_model, aggregation
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## Example 53: Track Vectors with Metadata

```python
@bauplan.model(materialization_strategy='REPLACE') track_vectors_with_metadata(...): Train Skipgram, TSNE, upload to MongoDB
```

- **Tags**: embedding, tsne, mongo_upload
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Example 54: Skipgram Model

```python
def skipgram_model(sequence_data): Gensim Word2Vec model training over playlists
```

- **Tags**: gensim, embedding_training
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: gensim

---

## Example 55: TSNE Analysis

```python
def tsne_analysis(embeddings): TSNE dimensionality reduction on embeddings
```

- **Tags**: scikit_learn, tsne, dimensionality_reduction
- **Type**: code_example
- **Difficulty**: easy
- **Preferred Tool**: scikit_learn

---

## Example 56: Upload Vectors to MongoDB

```python
def upload_vectors_to_mongodb(mongo_uri, table, db, collection): Upload track vectors and metadata to MongoDB Atlas
```

- **Tags**: mongodb, upload_vectors, vector_search
- **Type**: code_example
- **Difficulty**: medium
- **Preferred Tool**: mongodb

---

This collection of examples illustrates various use cases for Bauplan, including data cleaning, machine learning, SQL-based analytics, and integration with tools like DuckDB, OpenAI, and MongoDB.