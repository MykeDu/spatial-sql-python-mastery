# Day 46: Keep Transformations Out of the KNN Lookup

## Focus

Make the nearest-park lookup both correct and index-aware by preparing the geometry before the query runs. In the Marion County park-access analysis, the goal was to calculate distances in US survey feet without asking PostGIS to transform every candidate row during the KNN search.

## What I Learned

The KNN operator `<->` is most useful when it can compare geometry columns that already share the same projected coordinate system. For this analysis, that means storing prepared geometry in EPSG:2236 before running the nearest-neighbor lookup.

EPSG:2236 is appropriate for the working analysis because its coordinate units are US survey feet. That makes the final `ST_Distance` result readable and operational: a distance of `1,250` means approximately 1,250 US survey feet.

The important boundary is this:

- `ST_Transform` belongs in the preparation step.
- `<->` belongs in the index-aware lookup.
- `ST_Distance` belongs in the result calculation and validation step.

If `ST_Transform` is nested inside the KNN expression, PostGIS may not be able to use the spatial index as effectively. The query can still look logically correct while giving up the performance advantage that made the KNN pattern valuable.

## What I Practiced

I refactored the park-access query around prepared EPSG:2236 geometry columns:

- Subdivision representative points are stored in the analysis CRS.
- Park points are stored in the same analysis CRS.
- The nearest park is selected with `<->`.
- The reported distance is calculated with `ST_Distance` in feet.
- The browser-facing EPSG:4326 geometry stays separate from the analysis geometry.

The key implementation idea is to transform once when preparing the data, rather than transforming inside every nearest-neighbor lookup.

```sql
ALTER TABLE analysis.subdivisions
ADD COLUMN representative_point_2236 geometry(Point, 2236);

UPDATE analysis.subdivisions
SET representative_point_2236 =
    ST_Transform(
        ST_PointOnSurface(geom),
        2236
    );

CREATE INDEX subdivisions_representative_point_2236_gix
ON analysis.subdivisions
USING GIST (representative_point_2236);

ALTER TABLE analysis.parks
ADD COLUMN geom_2236 geometry(Point, 2236);

UPDATE analysis.parks
SET geom_2236 = ST_Transform(geom, 2236);

CREATE INDEX parks_geom_2236_gix
ON analysis.parks
USING GIST (geom_2236);
```

With the prepared columns in place, the lookup stays simple:

```sql
SELECT
    s.subdivision_id,
    s.subdivision_name,
    p.park_id,
    p.park_name,
    ST_Distance(
        s.representative_point_2236,
        p.geom_2236
    ) AS distance_feet
FROM analysis.subdivisions AS s
CROSS JOIN LATERAL (
    SELECT
        p.park_id,
        p.park_name,
        p.geom_2236
    FROM analysis.parks AS p
    WHERE p.geom_2236 IS NOT NULL
    ORDER BY p.geom_2236
        <-> s.representative_point_2236
    LIMIT 1
) AS p;
```

The `CROSS JOIN LATERAL` means: for each subdivision, run a small nearest-park search using that subdivision's representative point. The `<->` ordering is the KNN part. The `LIMIT 1` keeps only the closest park.

## The Pattern

Prepare geometry first, then search.

```sql
-- Preparation boundary
ST_Transform(source_geometry, 2236)

-- Index-aware nearest-neighbor search
ORDER BY park_geometry_2236 <-> subdivision_point_2236

-- Distance in US survey feet
ST_Distance(
    subdivision_point_2236,
    park_geometry_2236
)
```

Avoid this shape when the goal is an index-aware KNN lookup:

```sql
ORDER BY ST_Transform(p.geom, 2236)
    <-> ST_Transform(s.geom, 2236)
```

That version mixes coordinate preparation with candidate ranking. It is harder to reason about, can prevent the intended index-assisted plan, and repeats work during the search. The safer design is to materialize or otherwise prepare both sides in EPSG:2236, index those columns, and keep the KNN expression limited to the indexed geometry columns.

## AI Pairing Prompt

I am refactoring a PostGIS nearest-neighbor query. Review this query for index-aware KNN behavior.

Requirements:

1. Use prepared geometry columns in EPSG:2236.
2. Use `CROSS JOIN LATERAL` to find one nearest park per subdivision.
3. Use the `<->` operator in the `ORDER BY` clause.
4. Calculate the final distance with `ST_Distance` in US survey feet.
5. Keep `ST_Transform` out of the KNN lookup itself.
6. Suggest the spatial indexes and an `EXPLAIN` check I should run.

Explain any change in plain language, then return the corrected SQL.

## Why This Matters

Nearest-neighbor analysis is not only about returning the right row. It is also about preserving the query plan that lets the database find that row efficiently.

Preparing the geometry makes the coordinate system explicit, makes the distance units predictable, and gives the spatial index a stable expression to support. That matters when the same lookup runs across many subdivisions, when the park layer grows, or when the query becomes part of a repeatable pipeline.

The practical rule is simple: transform at the data-preparation boundary, not inside the KNN search.

## Next Steps

- Run `EXPLAIN (ANALYZE, BUFFERS)` on the lateral lookup.
- Confirm that the prepared geometry columns have SRID 2236.
- Check that the GiST indexes are present and used by the plan.
- Validate a sample of `distance_feet` values against known map locations.
- Keep EPSG:4326 for web output and EPSG:2236 for distance-based analysis.