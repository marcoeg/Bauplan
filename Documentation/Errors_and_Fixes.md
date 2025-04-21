# Bauplan Errors and Fixes

This document contains a collection of common errors and their fixes, providing solutions for issues encountered in Bauplan pipelines. Each entry includes a problem description, its fix, and associated metadata such as tags, type, difficulty, and preferred tool.

---

## Error and Fix 1: NaN Values in Training Dataset

**Problem**: NaN values in training dataset break feature scaling and model training.\
**Fix**: Drop rows with missing values beforeApplying transformations.

- **Tags**: error_handling, data_cleaning, ml_pipeline
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: pandas

---

## Error and Fix 2: Null Values in `on_scene_datetime`

**Problem**: Null values in `on_scene_datetime` cause incorrect waiting time calculations.\
**Fix**: Add a Bauplan expectation to enforce non-null constraint before computing differences.

- **Tags**: error_handling, data_validation, arrow
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Error and Fix 3: Incorrect Joins on `PULocationID`

**Problem**: Missing or incorrectly joined `PULocationID` columns after Arrow joins can silently produce wrong results.\
**Fix**: Validate join integrity post-join.

- **Tags**: error_handling, join_validation, arrow
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Error and Fix 4: Skewed Mileage Data

**Problem**: Skewed distributions in raw mileage data can cause unstable models.\
**Fix**: Apply log-transform before feature scaling to stabilize variances.

- **Tags**: feature_engineering, ml_pipeline, error_handling
- **Type**: errors_and_fixes
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Error and Fix 5: Branch Already Exists

**Problem**: Branch already exists when trying to create a new ingestion branch.\
**Fix**: Check existence first using `has_branch` API and raise a ValueError if needed.

- **Tags**: branching, error_handling, bauplan
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Error and Fix 6: Unexpected Null Values

**Problem**: Data contains unexpected null values in critical columns (e.g., 'age').\
**Fix**: Scan columns immediately after ingestion using Arrow and assert no nulls before merging.

- **Tags**: data_validation, error_handling, arrow
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: arrow

---

## Error and Fix 7: Missing Source Branch

**Problem**: Merge attempts fail because the source branch does not exist anymore.\
**Fix**: Always validate that the branch exists before merge attempts.

- **Tags**: branching, error_handling, bauplan
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Error and Fix 8: Missing S3 Bucket

**Problem**: S3 bucket does not exist when trying to upload synthetic events.\
**Fix**: Create bucket dynamically and apply public read policies if missing.

- **Tags**: s3, error_handling, bucket_creation
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: aws

---

## Error and Fix 9: Premature Branch Deletion

**Problem**: Merge fails because ingest branch was deleted prematurely.\
**Fix**: Ensure merge happens before any cleanup operations are triggered.

- **Tags**: branching, error_handling, bauplan
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: bauplan

---

## Error and Fix 10: Arrow Schema Mismatch

**Problem**: Arrow schema mismatch during ingestion of synthetic events.\
**Fix**: Enforce schema consistency by sampling from existing production datasets.

- **Tags**: data_ingestion, error_handling, schema_validation
- **Type**: errors_and_fixes
- **Difficulty**: medium
- **Preferred Tool**: pandas

---

## Error and Fix 11: Ambiguous OpenAI Responses

**Problem**: OpenAI responses may be ambiguous or unexpected.\
**Fix**: Always clean and normalize the LLM response before interpreting yes/no answers.

- **Tags**: openai, error_handling, response_validation
- **Type**: errors_and_fixes
- **Difficulty**: medium
- **Preferred Tool**: openai

---

## Error and Fix 12: Schema Mismatches in Joins

**Problem**: Schema mismatches when joining Walmart and Amazon products.\
**Fix**: Carefully select and serialize matching columns before join operations.

- **Tags**: data_joining, schema_validation, error_handling
- **Type**: errors_and_fixes
- **Difficulty**: medium
- **Preferred Tool**: duckdb

---

## Error and Fix 13: TSNE Output Mismatch

**Problem**: TSNE output size mismatch causes visualization crashes.\
**Fix**: Double-check perplexity, number of iterations, and ensure dataset size &gt;&gt; perplexity.

- **Tags**: tsne, error_handling, dimensionality_reduction
- **Type**: errors_and_fixes
- **Difficulty**: medium
- **Preferred Tool**: scikit_learn

---

## Error and Fix 14: MongoDB Schema Mismatch

**Problem**: MongoDB vector uploads fail silently if collection already exists with schema mismatch.\
**Fix**: Drop collection explicitly before new insertions.

- **Tags**: mongodb, error_handling, collection_management
- **Type**: errors_and_fixes
- **Difficulty**: easy
- **Preferred Tool**: mongodb

---

This collection of errors and fixes provides solutions to common issues in Bauplan pipelines, covering data validation, schema mismatches, and integration with tools like OpenAI and MongoDB.