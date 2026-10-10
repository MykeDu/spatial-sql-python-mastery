# Day 32: Index-Aware Spatial Searches

## Focus

Make the park-access query faster without changing what it means.

Todayâ€™s pattern is index-aware spatial searching: keep the analysis in projected geometry, use a GiST index, and write the distance predicate so PostgreSQL can use it.

The working setup:

- Projected geometry: `EPSG:3086`
- Distance threshold: `1609.344` meters
- Spatial index: GiST on `parks.geom_3086`
- Diagnostic tool: `EXPLAIN ANALYZE`

## What I Learned

A spatial query can be logically correct and still need performance work.

For distance-based searches, the important sequence is:

1. Keep both geometry columns in the projected CRS.
2. Store or transform the park geometry as `geom_3086`.
3. Add a GiST index to that geometry column.
4. Use `ST_DWithin()` with a distance expressed in meters.
5. Inspect the real execution plan with `EXPLAIN ANALYZE`.

The index does not replace the spatial predicate. It gives PostgreSQL a fast way to narrow the candidate geometries before checking the exact distance condition.

```sql
CREATE INDEX parks_geom_3086_gist
ON parks
USING GIST (geom_3086);
```

The spelling matters too. I typed `EXPAIN ANALYZE` on the first attempt. PostgreSQL could not interpret that command because the correct keyword is `EXPLAIN`.

## What I Practiced

I audited a query that returns subdivisions with no park within one mile.

The logic was correct on the first try: for each subdivision point, return the record only when no nearby park exists.

```sql
SELECT
    s.subdivision_id,
    s.name
FROM subdivision_points AS s
WHERE NOT EXISTS (
    SELECT 1
    FROM parks AS p
    WHERE ST_DWithin(
        s.geom_3086,
        p.geom_3086,
        1609.344
    )
);
```

The `NOT EXISTS` clause translates directly:

> Keep this subdivision only if PostgreSQL cannot find a park within one mile.

I then reviewed the execution plan with:

```sql
EXPLAIN ANALYZE
SELECT
    s.subdivision_id,
    s.name
FROM subdivision_points AS s
WHERE NOT EXISTS (
    SELECT 1
    FROM parks AS p
    WHERE ST_DWithin(
        s.geom_3086,
        p.geom_3086,
        1609.344
    )
);
```

For the project build, I continued rendering the subdivisions, parks, and ten farthest connection lines in the locally served Leaflet proof map.

The browser reached the map code, but the data-loading step raised:

```text
SyntaxError: JSON.parse: unexpected character at line 1 column 1 of the JSON data
```

The next debugging step is not to change the geometry logic. It is to inspect the browser Network tab. The likely cause is that a fetch request received an HTML 404 page instead of GeoJSON.

## The Pattern

```text
Keep geometry in EPSG:3086
 |
 v
Create a GiST index on parks.geom_3086
 |
 v
Use ST_DWithin(..., 1609.344)
 |
 v
Use NOT EXISTS for "no nearby park"
 |
 v
Run EXPLAIN ANALYZE
 |
 v
Inspect the actual browser response when JSON.parse fails
```

In plain English:

> Write the spatial condition so it is both correct and inspectable.

For the map, keep the responsibilities separate:

- PostgreSQL handles projected-distance analysis.
- GeoJSON handles browser-ready output.
- Leaflet handles rendering.
- The Network tab verifies that the browser received the file you expected.

## AI Pairing Prompt

```text
Review this spatial query plan and implementation.

Task:
Return subdivision_id and name for every record in subdivision_points
that has no park within one mile.

Tables and geometry columns:
- subdivision_points.geom_3086
- parks.geom_3086

Requirements:
- Both geometry columns use EPSG:3086.
- One mile is 1609.344 meters.
- parks.geom_3086 has a GiST index.
- The query should use NOT EXISTS and ST_DWithin.
- The diagnostic command must be EXPLAIN ANALYZE.

First, confirm whether the SQL logic is correct.
Then identify the narrowest issue in the EXPLAIN ANALYZE output that could explain
a slow query. Do not rewrite the whole workflow or suggest unrelated optimizations.
```

For the Leaflet error, the focused follow-up is:

```text
A locally served Leaflet map throws:

SyntaxError: JSON.parse: unexpected character at line 1 column 1 of the JSON data

The map fetches GeoJSON files from a data/ directory.

Give me the next diagnostic step only:
tell me what to inspect in the browser Network tab, how to recognize an HTML
404 response, and what server working directory and file paths should be verified.
```

## Why This Matters

Performance is part of correctness when the dataset grows.

A query that works on a small sample can become unusable across thousands of subdivision points and park features. The GiST index and `ST_DWithin()` pattern give PostgreSQL a spatial search strategy instead of forcing a full comparison against every park for every subdivision.

The same implementation-first habit applies in the browser. A `JSON.parse` error is usually a response problem before it is a parsing problem. Inspect the actual response, verify the server root, and confirm that the requested path matches the file layout.

The goal is not to guess. The goal is to make the database plan and browser request visible.

## Next Steps

- Re-run the query with the correctly spelled `EXPLAIN ANALYZE`.
- Confirm that PostgreSQL can use the GiST index on `parks.geom_3086`.
- Record the execution time and plan shape before making changes.
- Open the Leaflet mapâ€™s Network tab and inspect each GeoJSON request.
- Verify the server is running from the `web` folder:
  ```bash
  cd web
  python -m http.server 8000
  ```
- Confirm the requested paths match the actual files under `web/data/`.
- Fix the `JSON.parse` checkpoint only after confirming whether the response is GeoJSON or an HTML error page.