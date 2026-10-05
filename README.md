# Mini ETL Project: Movies (Medallion Architecture on Databricks)

A small, end-to-end ETL pipeline that loads a CSV of movies into **Databricks** and refines it through the three layers of the **Medallion Architecture**: Bronze, Silver and Gold.

I built this as a warm-up before tackling a larger retail sales ETL project. The dataset is deliberately tiny so the focus stays on the *pattern* rather than the data.

---

## Objectives

- Ingest a raw CSV file and keep an untouched copy (Bronze)
- Clean, type-cast and validate the data (Silver)
- Produce analytics-ready summary tables (Gold)
- Check data quality by comparing row counts between layers

## Tech Stack

| Tool | Purpose |
| --- | --- |
| Databricks | Notebook environment and compute |
| PySpark | Reading, cleaning and writing data |
| Spark SQL | Gold-layer aggregations |
| Delta tables | Storage format for each layer |

---

## Architecture

```
  movies.csv
      │
      ▼
┌──────────────┐   Raw copy, all columns as text,
│    BRONZE    │   plus load metadata
│ bronze.movies│
└──────┬───────┘
       ▼
┌──────────────┐   Typed, trimmed, de-duplicated,
│    SILVER    │   validated, enriched with decade
│ silver.movies│
└──────┬───────┘
       ▼
┌─────────────────────────────┐
│            GOLD             │
│ gold.avg_rating_by_genre    │
│ gold.top_rated_movies       │
│ gold.movies_by_decade       │
└─────────────────────────────┘
```

| Layer | What happens | Why |
| --- | --- | --- |
| **Bronze** | Load the CSV as-is (every column as text) and add `load_date` and `source_file` | Keeps a faithful record of the source so the pipeline can always be re-run from scratch |
| **Silver** | Fix data types, trim text, remove duplicates, filter invalid rows, add a `decade` column | Creates a clean, trustworthy single source of truth |
| **Gold** | Aggregate into business-friendly tables | Ready for dashboards and reporting |

---

## Dataset

`movies.csv` contains 10 movies with 5 columns:

| Column | Type (Silver) | Description |
| --- | --- | --- |
| `movie_id` | Integer | Unique identifier |
| `title` | Text | Movie title |
| `year` | Integer | Release year |
| `genre` | Text | Genre (e.g. Sci-Fi, Crime, Drama) |
| `rating` | Decimal | Rating out of 10 |

---

## Setup

1. Sign in to Databricks and create a new Python notebook.
2. Upload `movies.csv` to a Volume in your workspace.
3. Create three schemas in your catalog: `bronze`, `silver` and `gold`.

> Catalog, schema and volume names differ between workspaces, so use whatever names your workspace shows.

---

## Pipeline Steps

### 1. Bronze: raw ingestion
- Read `movies.csv` with every column treated as text, so nothing is changed or lost on the way in
- Add a `load_date` column (when the data was loaded) and a `source_file` column (where it came from)
- Save the result as the table `bronze.movies`

### 2. Silver: clean and validate
- Convert `movie_id` and `year` to integers and `rating` to a decimal
- Trim extra spaces from `title` and `genre`
- Remove duplicate rows based on `movie_id`
- Drop rows that break the rules: missing ID or title, a rating outside 0-10, or an impossible release year
- Add a `decade` column (for example 1999 becomes 1990)
- Save the result as the table `silver.movies`

### 3. Gold: analytics tables
- **`gold.avg_rating_by_genre`**: number of movies and average rating for each genre
- **`gold.top_rated_movies`**: the five highest-rated movies, ranked
- **`gold.movies_by_decade`**: number of movies and average rating for each decade

---

## Data Quality Checks

After running the pipeline, I confirm that:

- The Bronze row count matches the number of rows in the source file
- The Silver row count is equal to or lower than Bronze, and I can explain any difference
- Silver has no duplicate `movie_id` values
- Silver has no missing values in `movie_id`, `title` or `rating`

---

## Expected Results

With the provided 10-row file, every row passes validation, so Bronze and Silver both contain **10 rows**.

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
├── movies.csv       # source data
├── README.md        # this file
└── notebook/        # exported Databricks notebook
```

## Ideas for Extending It

- Add a few deliberately messy rows to `movies.csv` (blank titles, a rating of 15, a duplicated `movie_id`, extra spaces) and watch Silver clean or reject them
- Save rejected rows to a separate table instead of silently dropping them
- Load new data incrementally instead of overwriting
- Schedule the notebook as a Databricks Job
- Build a small dashboard on top of the Gold tables

## What I Learned

- Why the raw layer is kept untouched and what each later layer adds
- How data moves from raw to clean to analytics-ready
- How to verify a pipeline by reconciling row counts

---

**Author:** Theo ML Tshabalala  
**Date:** October 2026
