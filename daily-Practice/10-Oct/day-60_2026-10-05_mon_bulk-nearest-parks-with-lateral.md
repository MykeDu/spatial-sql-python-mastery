# Day 60: Bulk nearest-park matching with LATERAL

## Focus

Match every usable subdivision representative point to its nearest park in one PostGIS query.

The pattern combines prepared EPSG:2236 geometries, `CROSS JOIN LATERAL`, the `<->` KNN operator, `LIMIT 1`, and `ST_Distance`. The goal is to let the database repeat the nearest-park lookup once for each subdivision without writing a Python loop or building a large self-join.

## What I Learned

`CROSS JOIN LATERAL` lets the park subquery use the current subdivision row. For each subdivision point, PostGIS can order candidate parks by the KNN distance operator:

```sql
ORDER BY p.geom_2236 <-> sp.access_geom
LIMIT 1
```

The `<->` operator is the fast nearest-neighbor ordering step. It identifies the closest candidate using the indexed geometry. After that candidate is selected, `ST_Distance` calculates the final distance value that will be reported.

The two operations have different jobs:

- `<->` finds the nearest candidate efficiently.
- `ST_Distance` returns the measurement for the selected match.

Because both geometries are already prepared in EPSG:2236, the distance is expressed in US survey feet for this Marion County workflow.

## What I Practiced

I practiced structuring the query so geometry preparation and matching are clear before any reporting filters are applied.

```sql
WITH subdivision_points AS (
    SELECT
        s.subdivision_id,
        s.subdivision_name,
        ST_PointOnSurface(s.geom_2236) AS access_geom
    FROM marion_subdivisions AS s
    WHERE s.geom_2236 IS NOT NULL
      AND NOT ST_IsEmpty(s.geom_2236)
      AND ST_IsValid(s.geom_2236)
),
nearest_parks AS (
    SELECT
        sp.subdivision_id,
        sp.subdivision_name,
        p.park_id,
        p.park_name,
        ST_Distance(sp.access_geom, p.geom_2236) AS distance_ft
    FROM subdivision_points AS sp
    CROSS JOIN LATERAL (
        SELECT
            p.park_id,
            p.park_name,
            p.geom_2236
        FROM marion_parks AS p
        WHERE p.geom_2236 IS NOT NULL
          AND NOT ST_IsEmpty(p.geom_2236)
        ORDER BY p.geom_2236 <-> sp.access_geom
        LIMIT 1
    ) AS p
)
SELECT
    subdivision_id,
    subdivision_name,
    park_id,
    park_name,
    ROUND(distance_ft::numeric, 1) AS distance_ft
FROM nearest_parks
WHERE distance_ft <= 5280
ORDER BY distance_ft, subdivision_name;
```

The lateral subquery runs once per subdivision point and returns one park. The outer query then calculates the distance for that selected park and applies a reporting filter for subdivisions within one mile.

The `WHERE` clause inside the lateral subquery defines which park records are eligible for matching. The final `WHERE` clause reports only matches within one mile. Keeping those responsibilities separate makes the query easier to audit and prevents a reporting preference from changing the nearest-park match.

## The Pattern

Prepare first, match second, report last.

1. Build a usable subdivision point in prepared EPSG:2236 geometry.
2. Use `CROSS JOIN LATERAL` to run one nearest-park lookup per subdivision.
3. Order candidates with `<->` and keep only the first row with `LIMIT 1`.
4. Use `ST_Distance` on the selected pair to produce the reported distance.
5. Apply reporting filters only after all nearest matches have been created.

This is the bulk version of the single-feature nearest-neighbor exercise. The query expresses the repeated operation directly, so the database handles the set of subdivisions instead of the application managing a loop.

## AI Pairing Prompt

Review this PostGIS query as a coding partner.

Check that:

- `geom_2236` is a prepared, projected geometry column in EPSG:2236.
- The subdivision geometry gate excludes null, empty, and invalid geometries before `ST_PointOnSurface`.
- `CROSS JOIN LATERAL` correctly makes the current subdivision point available to the park subquery.
- `<->` is used for KNN ordering and `LIMIT 1` returns one nearest park per subdivision.
- `ST_Distance` is calculated for the selected match and interpreted in US survey feet.
- Matching eligibility filters are not confused with final reporting filters.
- The query will use an appropriate spatial index on `marion_parks.geom_2236`.

Explain the query in plain English first. Then identify any correctness, indexing, tie-breaking, or geometry-health risks. Suggest the smallest safe improvement, and do not replace the KNN pattern with a manual loop or an unnecessary distance-ordered self-join.

## Why This Matters

A park-access workflow usually needs one answer for many subdivisions, not one hand-built answer at a time. LATERAL makes that repeated lookup readable while KNN ordering keeps the nearest-neighbor search aligned with PostGIS indexing.

Separating matching from reporting is equally important. If a one-mile filter is pushed into the wrong part of the query, it can change which park is selected instead of simply filtering the completed results. A reliable workflow first finds the nearest eligible park, then decides which matches belong in a report, dashboard, or portfolio analysis.

## Next Steps

1. Confirm that the prepared `geom_2236` columns have spatial indexes.
2. Add a stable tie-breaker if two parks can have the same KNN distance.
3. Compare the LATERAL result with a smaller hand-checked sample.
4. Rejoin park attributes such as park type, acreage, or amenities after the match is validated.
5. Save the query and its plain-English explanation as a portfolio-ready example.

The next improvement is not to make the SQL more complicated. It is to make the geometry gate, nearest-match rule, distance units, and reporting decision visible enough that another person can reproduce and trust the result.