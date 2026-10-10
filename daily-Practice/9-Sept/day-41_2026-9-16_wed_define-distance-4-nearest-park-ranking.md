# Day 41: Define the Distance for Nearest Park Ranking

## Focus

Define what "distance to the nearest park" actually means before ranking subdivisions.

Today's metric is straight-line distance from one representative point per subdivision to the nearest park. The goal is not to imply driving time, walking time, or boundary-to-park distance. It is a clear analytical measure that can be explained, reproduced, and improved later if the project needs a travel-time metric.

## What I Learned

The distance point is part of the definition of the metric.

For an irregular or concave subdivision polygon, a centroid may fall outside the polygon. `ST_PointOnSurface` gives me a representative point guaranteed to be on the polygon surface, which makes it safer for this use:

```sql
ST_PointOnSurface(s.geom_2236) AS access_point
```

I also kept the analysis in EPSG:2236, where the coordinates use US survey feet. That lets `ST_Distance` return a value I can label as feet without calculating in longitude and latitude degrees.

The result still needs a precise explanation: this is the straight-line distance from one representative point per subdivision to the nearest park. It is not driving distance, walking distance, travel time, or the distance from the subdivision boundary to a park.

## What I Practiced

I wrote the nearest-park query on the first try.

The query creates a representative point for each subdivision, finds the nearest park with `CROSS JOIN LATERAL`, calculates the distance in EPSG:2236, and returns the ten farthest subdivisions:

```sql
WITH subdivision_points AS (
    SELECT
        s.subdivision_id,
        s.name,
        s.geom_2236,
        ST_PointOnSurface(s.geom_2236) AS access_point
    FROM subdivisions AS s
),
nearest_parks AS (
    SELECT
        s.subdivision_id,
        s.name,
        p.park_id,
        p.name AS park_name,
        s.access_point,
        p.geom_2236 AS park_geom,
        ST_Distance(s.access_point, p.geom_2236) AS distance_ft
    FROM subdivision_points AS s
    CROSS JOIN LATERAL (
        SELECT
            p.park_id,
            p.name,
            p.geom_2236
        FROM parks AS p
        ORDER BY p.geom_2236 <-> s.access_point
        LIMIT 1
    ) AS p
)
SELECT
    subdivision_id,
    name,
    park_id,
    park_name,
    distance_ft
FROM nearest_parks
ORDER BY distance_ft DESC
LIMIT 10;
```

The important pattern is that the lateral subquery can use the current subdivision's `access_point`. The KNN operator `<->` finds the nearest candidate efficiently, and `ST_Distance` calculates the reported distance in feet.

## The Pattern

Name the workflow stages so the metric is easy to audit:

1. Load the subdivision and park layers.
2. Validate the working CRS.
3. Create one representative point with `ST_PointOnSurface`.
4. Find the nearest park with `CROSS JOIN LATERAL`.
5. Calculate `ST_Distance` in EPSG:2236.
6. Rank the subdivisions by distance.
7. Create connection lines for inspection.
8. Export browser copies in EPSG:4326.

Keep EPSG:2236 for analysis. Use EPSG:4326 only for browser-facing copies and web mapping. The CRS change belongs at the export boundary, not in the middle of the distance calculation.

## AI Pairing Prompt

> Review this SQL and the wording of its published metric. Confirm that the query uses one representative point per subdivision, finds the nearest park with a lateral KNN search, and calculates distance in EPSG:2236 US survey feet. Then list any wording that could accidentally imply driving distance, walking distance, travel time, or boundary-to-park distance. Include limitations a reader should see next to the result.

## Why This Matters

A ranking is only as trustworthy as the thing it measures.

"Nearest park" sounds obvious until I have to choose the point, CRS, distance function, and search pattern. Naming those choices prevents a technically correct query from producing a misleading conclusion. It also gives the project a clean upgrade path if I later replace straight-line distance with a network-based travel metric.

The refactor matters for the same reason. Named stages make it possible to inspect whether a problem came from CRS handling, representative points, nearest-neighbor selection, ranking, connection lines, or browser export.

## Next Steps

- Apply the staged workflow to `mc-park-access`.
- Keep all analytical geometry in EPSG:2236.
- Export separate EPSG:4326 copies for the browser map.
- Inspect the ten farthest results with connection lines.
- Publish the metric definition and limitations next to the ranking.
- Ask AI to review both the SQL and the wording before treating the result as a public claim.