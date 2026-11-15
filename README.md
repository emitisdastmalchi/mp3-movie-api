# mp3-movie-api

This FastAPI application stores a movie collection in a SQLite database and serves it through a REST API. `POST /upload` validates a CSV with the columns title, release_year, rating, and genre, then replaces the stored collection in a single SQL transaction. `GET /movies` returns movies and a total count using SQL, and `GET /movies/search` filters by minimum rating and returns only the requested columns. `POST /movies/process` reads the collection, cleans it with the MP1 validation and processing functions in `data_pipeline/`, and saves the result back to SQLite. Retrievals from `GET /movies` are cached with fakeredis for 90 seconds, and the cache is cleared after every successful upload or processing request so stale data is never returned. The code is organized into `src/database.py` (connection and table setup), `src/storage.py` (SQL queries), `src/cache.py` (caching), `src/models.py` (Pydantic models), and `src/middleware.py` (request logging). Because the data lives in `data/movies.db`, it persists across server restarts.

## Run

```bash
python -m uvicorn main:app --reload
python -m pytest tests -v
```

## Sample processing request
`POST /movies/process` after uploading `fixtures/sample_movies.csv`:

```json
{
  "validation": {"required_columns": ["title", "release_year", "rating", "genre"], "numeric_columns": ["release_year", "rating"]},
  "processing": {
    "remove_duplicates": true,
    "missing": {"enabled": true, "axis": "rows"},
    "outliers": {"enabled": true, "columns": ["rating"], "method": "iqr", "threshold": 1.5}
  }
}
```

Expected result:

```json
{
  "validation_rows_removed": 0,
  "cleaning_report": {"rows_before": 20, "rows_after": 16, "rows_removed": 4, "columns_before": 4, "columns_after": 4, "columns_removed": 0}
}
```