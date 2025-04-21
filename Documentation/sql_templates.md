# Bauplan SQL Templates

This document contains a collection of SQL query templates, designed for use with DuckDB in Bauplan pipelines. Each entry includes a SQL query and associated metadata such as tags, type, difficulty, and preferred tool.

---

## SQL Template 1: Average Waiting Times by Zone

```sql
SELECT Borough, Zone, AVG(waiting_time_minutes) AS avg_waiting_time
FROM taxi_trip_waiting_times
GROUP BY Borough, Zone
ORDER BY avg_waiting_time DESC;
```

- **Tags**: duckdb, sql_template, aggregation
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 2: Trip Count by Pickup Location

```sql
SELECT PULocationID, COUNT(*) AS trip_count
FROM taxi_fhvhv
WHERE pickup_datetime BETWEEN '2022-12-01' AND '2023-01-01'
GROUP BY PULocationID
ORDER BY trip_count DESC;
```

- **Tags**: duckdb, sql_template, aggregation
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 3: Filtered Trips

```sql
SELECT *
FROM trips_and_zones
WHERE trip_miles > 1.0 AND tips > 0.0;
```

- **Tags**: duckdb, sql_template, filter_pushdown
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 4: Purchase Sessions

```sql
SELECT user_session, event_hour, count(*) FROM ecommerce_clean WHERE event_type = 'purchase' GROUP BY 1, 2
```

- **Tags**: duckdb, sql_template, aggregation
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 5: Products per User Session

```sql
SELECT brand, COUNT(product_id)::FLOAT/count(distinct user_session) AS products_per_user_session, round(sum(price),2) AS revenue FROM ecommerce_clean WHERE event_type = 'purchase' GROUP BY 1 ORDER BY 3 DESC
```

- **Tags**: duckdb, sql_template, aggregation
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 6: Event Views by Brand

```sql
SELECT event_hour, brand, SUM(CASE WHEN event_type = 'view' THEN 1 ELSE 0 END) AS views FROM ecommerce_clean GROUP BY 1,2
```

- **Tags**: duckdb, sql_template, aggregation
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 7: Sample Matching Products

```sql
SELECT ltable_id, rtable_id, label FROM matching_products WHERE label = 1 LIMIT {max_k} UNION ALL SELECT ltable_id, rtable_id, label FROM matching_products WHERE label = 0 LIMIT {max_k}
```

- **Tags**: duckdb, sql_template, sampling
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 8: Join Product Matches

```sql
SELECT t.ltable_id as walmart_id, t.rtable_id as amazon_id, t.label, a.serialized_product as amazon_product, w.serialized_product as walmart_product FROM test_matches t JOIN amazon_products a ON t.rtable_id = a.id JOIN walmart_products w ON t.ltable_id = w.id
```

- **Tags**: duckdb, sql_template, join
- **Type**: sql_templates
- **Difficulty**: medium
- **Preferred Tool**: duckdb

---

## SQL Template 9: Playlist Track Aggregation

```sql
SELECT playlist_id, array_agg(track_uri ORDER BY pos ASC) as track_ids FROM tracks GROUP BY 1 ORDER BY 1 ASC
```

- **Tags**: duckdb, sql_template, aggregation
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 10: Popular Tracks

```sql
SELECT unnest(track_ids) as track_id FROM playlists_to_sequences GROUP BY 1 ORDER BY COUNT(*) DESC LIMIT {top_k}
```

- **Tags**: duckdb, sql_template, unnest
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

## SQL Template 11: Track Metadata Join

```sql
SELECT t.track_id, track_metadata.track_name, track_metadata.artist_name FROM track_table t JOIN track_metadata ON t.track_id = track_metadata.track_uri
```

- **Tags**: duckdb, sql_template, join
- **Type**: sql_templates
- **Difficulty**: easy
- **Preferred Tool**: duckdb

---

This collection of SQL templates demonstrates common analytical queries in Bauplan using DuckDB, covering aggregations, joins, filtering, and sampling.