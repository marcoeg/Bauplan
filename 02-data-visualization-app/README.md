# Data visualization app with Streamlit

In this example, we use the [TLC NY Taxi dataset](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) and build a pipeline that produces a table showcasing the boroughs and zones in New York City with the highest number of taxi pickups.
We'll then use Streamlit to visualize this table in an interactive app.

1. 📚 Description of the Pipeline
This Bauplan pipeline processes New York taxi trip data and outputs the top pickup locations ordered by number of trips.
It is structured into two models, forming a simple DAG:

➔ Step 1: trips_and_zones
Performs two S3 scans over the taxi_fhvhv and taxi_zones tables.

Filter pushdown: restricts taxi trips to pickups between January 1st, 2023 and February 2nd, 2023.

Column pruning: selects only necessary fields (e.g., pickup and dropoff locations, trip miles, base fare, tips).

Join operation: joins the trip data with the zone metadata on PULocationID and LocationID using PyArrow.

Optimization: after the join, combines Arrow chunks into a single, contiguous Arrow table.

Debugging: prints out table size and row count.

Environment: Python 3.11 with Arrow native support.

➔ Step 2: top_pickup_locations
Takes the output of trips_and_zones as input.

Converts the Arrow table into a Pandas DataFrame.

Groups and aggregates by PULocationID, Borough, and Zone.

Counts the number of trips per pickup location.

Sorts the locations by trip count in descending order.

Materializes the final output into the Bauplan catalog (strategy: REPLACE).

Environment: Python 3.11 with Pandas 2.2.0.

✨ Architectural Features:
Strong separation of Arrow processing vs Pandas processing.

Filter pushdowns and column pruning to optimize S3 reads.

Explicit materialization for key tables.

Multi-environment isolation for different stages (Arrow vs Pandas heavy workloads).

