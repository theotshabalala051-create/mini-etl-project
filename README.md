# Mini ETL Project: Movies (Databricks)

A small, end-to-end **ETL (Extract, Transform, Load)** pipeline built in **Databricks**. A CSV of movies is read into memory, cleaned and validated *before* it is saved, and only the finished data is loaded into the Silver and Gold tables.

I built this as a warm-up before tackling a larger retail sales ETL project. The dataset is deliberately tiny so the focus stays on the pattern rather than the data.

---

## Objectives

- **Extract** raw data from a CSV file
- **Transform** it in memory: fix types, clean text, remove duplicates and validate rules
- **Load** only the clean data into Silver, then build analytics-ready Gold tables
- Keep rejected rows in a separate table so nothing disappears silently
- Check data quality by reconciling row counts

## ETL vs ELT

In **ELT**, raw data is loaded into the platform first and cleaned there afterwards. In **ETL**, as in this project, the cleaning happens *before* loading, so bad rows never reach the main tables. The original CSV stays untouched in the Volume, which acts as the raw copy, so there is no Bronze table.

## Tech Stack

| Tool | Purpose |
| --- | --- |
| Databricks | Notebook environment and compute |
| PySpark | Reading and transforming the data |
| Spark SQL | Gold-layer aggregations |
| Delta tables | Storage format for the loaded tables |

---

## Architecture

```
  movies.csv  (stored untouched in a Databricks Volume)
      │
      ▼
┌───────────────────────┐
│  EXTRACT              │  Read the CSV into memory
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│  TRANSFORM            │  Fix types, trim text, remove duplicates,
│  (in memory)          │  validate rules, add decade, split out bad rows
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│  LOAD                 │  silver.movies           (clean rows)
│                       │  silver.movies_rejected  (invalid rows)
└──────────┬────────────┘
           ▼
┌───────────────────────────────┐
│  GOLD                         │  gold.avg_rating_by_genre
│  (built from Silver)          │  gold.top_rated_movies
│                               │  gold.movies_by_decade
└───────────────────────────────┘
```

---

## Notebooks

| Notebook | Purpose |
| --- | --- |
| `00_debug` | Check file paths and table names, peek at the data, confirm row counts |
| `01_extract` | Read `movies.csv` into memory. Nothing is saved yet. |
| `02_transform` | Clean and validate the data, and separate valid rows from rejected rows |
| `03_load` | Save the valid rows and rejected rows to Silver, then build the Gold tables |

---

## Dataset

`movies.csv` contains 10 movies with 5 columns:

| Column | Type (after transform) | Description |
| --- | --- | --- |
| `movie_id` | Integer | Unique identifier |
| `title` | Text | Movie title |
| `year` | Integer | Release year |
| `genre` | Text | Genre (e.g. Sci-Fi, Crime, Drama) |
| `rating` | Decimal | Rating out of 10 |

---

## Setup

1. Sign in to Databricks and create the notebooks listed above.
2. Upload `movies.csv` to a Volume in your workspace.
3. Create two schemas in your catalog: `silver` and `gold`.

> Catalog, schema and volume names differ between workspaces, so use whatever names your workspace shows.

---

## Pipeline Steps

### 1. Extract
- Read `movies.csv` from the Volume into memory
- Keep every column as text at this stage, so nothing is changed or lost on the way in

### 2. Transform
- Convert `movie_id` and `year` to integers and `rating` to a decimal
- Trim extra spaces from `title` and `genre`
- Remove duplicate rows based on `movie_id`
- Check each row against the rules: ID and title present, rating between 0 and 10, and a realistic release year
- Add a `decade` column (for example 1999 becomes 1990)
- Split the result into **valid rows** and **rejected rows**, adding a reason to each rejected row

### 3. Load
- Save valid rows to `silver.movies`
- Save rejected rows to `silver.movies_rejected`
- Build the Gold tables from `silver.movies`:
  - **`gold.avg_rating_by_genre`**: number of movies and average rating for each genre
  - **`gold.top_rated_movies`**: the five highest-rated movies, ranked
  - **`gold.movies_by_decade`**: number of movies and average rating for each decade

---

## Data Quality Checks

After running the pipeline, I confirm that:

- Rows extracted = rows in `silver.movies` + rows in `silver.movies_rejected`
- `silver.movies` has no duplicate `movie_id` values
- `silver.movies` has no missing values in `movie_id`, `title` or `rating`
- Every rejected row has a reason recorded

---

## Expected Results

With the provided 10-row file, every row passes validation, so `silver.movies` contains **10 rows** and `silver.movies_rejected` is empty.

**Gold: average rating by genre (top rows)**

| genre | movie_count | avg_rating |
| --- | --- | --- |
| Crime | 2 | 9.05 |
| Sci-Fi | 3 | 8.70 |
| Thriller | 1 | 8.60 |

**Gold: top rated movies**

| rank | title | rating |
| --- | --- | --- |
| 1 | The Godfather | 9.2 |
| 2 | Pulp Fiction | 8.9 |
| 3 | Inception | 8.8 |

**Gold: by decade:** the 2010s have the most movies (5), followed by the 1990s (3).

---

## Project Structure

```
mini-etl-project/
├── movies.csv
├── README.md
└── notebooks/
    ├── 00_debug
    ├── 01_extract
    ├── 02_transform
    └── 03_load
```

## Ideas for Extending It

- Add a few deliberately messy rows to `movies.csv` (blank titles, a rating of 15, a duplicated `movie_id`, extra spaces) and check that they end up in `silver.movies_rejected` with the right reason
- Load new data incrementally instead of overwriting
- Schedule the notebooks as a Databricks Job
- Build a small dashboard on top of the Gold tables

## What I Learned

- The difference between ETL and ELT, and where the cleaning happens in each
- How to validate data before loading it and keep rejected rows for review
- How to verify a pipeline by reconciling row counts

---

**Author:** Theo ML Tshabalala  
**Date:** October 2026
