# Bauplan Python Snippets

This document contains a collection of Python code snippets , showcasing data processing tasks in Bauplan pipelines. Each entry includes a code snippet and associated metadata such as tags, type, difficulty, and preferred tool.

---

## Snippet 1: Join and Combine Chunks

```python
pickup_location_table = trips.join(zones, 'PULocationID', 'LocationID').combine_chunks()
```

- **Tags**: arrow, performance_optimization, join
- **Type**: snippet
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Snippet 2: Arrow to Pandas Conversion

```python
df = data.to_pandas()
```

- **Tags**: arrow, pandas, conversion
- **Type**: snippet
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Snippet 3: Timestamp Filter

```python
time_filter = pd.to_datetime('2022-01-01').tz_localize('UTC')
df = df[df['pickup_datetime'] >= time_filter]
```

- **Tags**: pandas, data_cleaning, timestamp_filter
- **Type**: snippet
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Snippet 4: Outlier Removal

```python
df = df[df['trip_miles'] > 0.0]
df = df[df['trip_miles'] < 200.0]
```

- **Tags**: pandas, data_cleaning, outlier_removal
- **Type**: snippet
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Snippet 5: Log Transform

```python
df['log_trip_miles'] = np.log10(df['trip_miles'])
```

- **Tags**: pandas, feature_engineering, log_transform
- **Type**: snippet
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Snippet 6: Data Size Debugging

```python
size_in_gb = round(data.nbytes / math.pow(1024, 3), 3)
print(f"\nThis table is {size_in_gb} GB and has {data.num_rows} rows\n")
```

- **Tags**: debugging, arrow, data_exploration
- **Type**: snippet
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

This collection of Python snippets illustrates common data processing tasks in Bauplan, including joins, data cleaning, feature engineering, and debugging.