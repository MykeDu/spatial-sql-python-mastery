# Day 58: Coordinate Transformation With Named CTEs

## Focus

Today's focus is making coordinate transformation explicit before doing distance work in PostGIS. In the Marion County park-access workflow, the prepared analysis geometry lives in EPSG:2236, while web-inspection geometry stays in EPSG:4326. The key move is to transform the source geometries inside named CTEs, then use those prepared results in `ST_Distance`.

You successfully implemented this pattern. The query is easier to read because each spatial preparation step has a name and a clear purpose.

## What I Learned

`ST_Distance` uses the units of the coordinate system attached to the geometry. If both geometries are in EPSG:4326, the result is in decimal degrees, not feet or meters. That can look like a valid number while still being the wrong measurement for a local access analysis.

For Marion County work, EPSG:2236 is the working coordinate system for local distance calculations. Transform the source geometry into EPSG:2236 before measuring distance. Keep the EPSG:4326 version when you need to inspect or publish the geometry in a web map.

The SRID label and the actual coordinate values need to agree. Changing a label with `ST_SetSRID` does not reproject coordinates. `ST_Transform` performs the coordinate conversion.

## What I Practiced

I used named CTEs instead of hiding transformations inside deeply nested subqueries. One CTE prepares the park access point, another prepares the candidate park geometries, and the final query calculates distance in the projected system.

```sql
WITH access_point AS (
    SELECT
        1 AS access_id,
        ST_Transform(
            ST_SetSRID(
                ST_MakePoint(-80.1918, 25.7743),
                4326
            ),
            2236
        ) AS geom_2236
),
parks_for_analysis AS (
    SELECT
        p.park_id,
        p.park_name,
        ST_Transform(p.geom, 2236) AS geom_2236
    FROM marion_county_parks AS p
    WHERE p.geom IS NOT NULL
),
park_distances AS (
    SELECT
        p.park_id,
        p.park_name,
        ST_Distance(a.geom_2236, p.geom_2236) AS distance_feet
    FROM access_point AS a
    CROSS JOIN parks_for_analysis AS p
)
SELECT
    park_id,
    park_name,
    ROUND(distance_feet::numeric, 1) AS distance_feet
FROM park_distances
ORDER BY distance_feet
LIMIT 10;
```

The important detail is not the number of CTEs. It is that the geometry used by `ST_Distance` is clearly prepared in the same projected CRS.

## The Pattern

Use this sequence:

1. Identify the source geometry's current SRID.
2. Use `ST_SetSRID` only when the coordinates have no correct SRID label.
3. Use `ST_Transform` to convert coordinates into the analysis CRS.
4. Name the transformed geometries in CTEs.
5. Run `ST_Distance` only after both inputs are in the same appropriate CRS.
6. Preserve a separate EPSG:4326 geometry when the result needs to be inspected on the web.

The reusable shape is:

```sql
WITH source_a AS (
    SELECT ST_Transform(geom, 2236) AS geom_2236
    FROM source_table_a
),
source_b AS (
    SELECT ST_Transform(geom, 2236) AS geom_2236
    FROM source_table_b
)
SELECT ST_Distance(a.geom_2236, b.geom_2236)
FROM source_a AS a
CROSS JOIN source_b AS b;
```

This keeps the coordinate decision visible and prevents a unit mistake from being buried inside a calculation.

## AI Pairing Prompt

Review this PostGIS query as a GIS database partner:

```sql
WITH access_point AS (
    SELECT ST_Transform(access_geom, 2236) AS geom_2236
    FROM park_access_points
    WHERE access_id = 42
),
parks_for_analysis AS (
    SELECT
        park_id,
        park_name,
        ST_Transform(geom, 2236) AS geom_2236
    FROM marion_county_parks
    WHERE geom IS NOT NULL
)
SELECT
    p.park_id,
    p.park_name,
    ST_Distance(a.geom_2236, p.geom_2236) AS distance_feet
FROM access_point AS a
CROSS JOIN parks_for_analysis AS p
ORDER BY distance_feet
LIMIT 10;
```

Check the query for SRID correctness, distance units, unnecessary transformations, null handling, index opportunities, and whether the CTE names make the workflow clear. Point out any place where EPSG:4326 should be retained for web inspection instead of analysis.

## Why This Matters

Coordinate transformation is not a cosmetic cleanup step. It controls whether a distance result is meaningful. A degree-based result can pass through a query, a notebook, and a map without raising an obvious error, but it does not answer a local question such as which Marion County parks are closest to an access point.

Named CTEs make the spatial logic reviewable. They also give you a clean place to add filters, validate geometry preparation, and rejoin results to web-ready geometry later.

## Next Steps

- Add a validation query that reports the SRID and geometry type at each preparation stage.
- Compare the EPSG:2236 distance result with a known park-access measurement.
- Keep analysis geometry and web-inspection geometry as separate, clearly named columns.
- Practice converting the same pattern to `ST_DWithin` when the question is whether a park falls within a distance threshold.