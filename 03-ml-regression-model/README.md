# A machine learning pipeline

In this example we show how to organize and run a simple machine learning project with bauplan. We will build and run a pipeline that takes some raw data from the [TLC NY taxi dataset](taxi), transforms it into a training dataset with the right features, trains a Linear Regression model to predict **the tip amount** of taxi rides, and writes the predictions to an Iceberg table. We will also use the Bauplan SDK in some notebooks to explore the dataset and the predictions.

---
1. 📚 Description of the Pipeline (ML Pipeline)
This Bauplan pipeline implements a complete supervised Machine Learning workflow for predicting taxi trip tips based on trip features.
It uses a combination of Apache Arrow, Pandas, and Scikit-learn components.

➔ Step 1: clean_taxi_trips
Performs a direct S3 scan on the Iceberg table taxi_fhvhv.

Filter pushdown: restricts data to trips between Jan–March 2023.

Column pruning: selects only essential fields (e.g., pickup/dropoff info, fares, tips).

Converts from Arrow to Pandas explicitly.

Cleans rows where:

trip_miles ≤ 1.0

tips ≤ 0.0

base_passenger_fare ≤ 1.0

Output: a clean Pandas dataframe.

➔ Step 2: training_dataset
Consumes cleaned taxi trip data.

Drops NaN values.

Feature engineering:

Adds a log_trip_miles column to normalize skewed mileage distributions.

Feature scaling:

Scales features (log_trip_miles, base_passenger_fare, trip_time) using StandardScaler to mean=0, std=1.

Prepares training features and target (tips).

Output: a scaled Pandas dataframe ready for ML training.

➔ Step 3: train_regression_model
Consumes the training dataset.

Splits into:

Training set (80%)

Validation set (10%)

Test set (10%)

Trains a Linear Regression model on the training set.

Saves the trained model into the Bauplan key-value store for reuse.

Returns the test set for final evaluation.

➔ Step 4: tip_predictions
Loads the previously trained regression model from the key-value store.

Applies it to the test set.

Generates predicted tips.

Materializes the final predictions output with materialization_strategy='REPLACE'.

Environment: Python 3.11, Pandas 2.1.0, Scikit-learn 1.3.2.

✨ Architectural Highlights:
Clear separation between data cleaning, feature engineering, training, and prediction.

Model persistence and retrieval integrated into the DAG.

Mix of Arrow, Pandas, and Scikit-learn, each used appropriately.

Modular and reproducible ML pipelines, easily extendable.

