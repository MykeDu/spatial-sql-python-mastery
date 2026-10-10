# Day 61: Reporting nearest-park evidence clearly

## Focus

Turn bulk nearest-park matches into evidence that another person can read, check, and sort without guessing what each value means.

The matching step already gives one selected park for each subdivision representative point. The reporting step needs to preserve that chain:

1. the subdivision,
2. the representative point used for access,
3. the prepared EPSG:2236 geometry,
4. the selected nearest park,
5. the measured `distance_ft`,
6. the final filters and sort order.

The main practice today was not changing the nearest-neighbor method. It was making the result explainable.

## What I Learned

A nearest-park result contains several different facts, and they should not be collapsed into one vague label.

The selected nearest park is the feature returned by the KNN lookup. It is the match, not just a park that happens to be nearby.

The representative point is the point used to stand in for the subdivision during the lookup. `ST_PointOnSurface` gives the query a usable point, but it does not mean every resident lives at that location.

The prepared EPSG:2236 geometry is the projected geometry used for the local distance workflow. It gives `ST_Distance` a unit that can be reported as US survey feet in this Marion County setup.

`distance_ft` is the measured distance between the representative point and the selected park geometry. It is evidence about the matching operation, not a claim about walking distance, travel time, or the distance from every home.

The final reporting and sorting happen after the match is selected. A report can filter to parks within one mile or sort by distance without changing which park was considered nearest.

## What I Practiced

I practiced keeping the matching evidence available in the result instead of returning only a park name and a rounded number.

```sql
WITH subdivision_points AS (
    SELECT
        s.subdivision_id,
        s.subdivision_name,
        s.geom_2236 AS subdivision_geom_2236,
        ST_PointOnSurface(s.geom_2236) AS representative_point_2236
    FROM marion_subdivisions AS s
    WHERE s.geom_2236 IS NOT NULL
      AND NOT ST_IsEmpty(s.geom_2236)
      AND ST_IsValid(s.geom_2236)
),
nearest_park_matches AS (
    SELECT
        sp.subdivision_id,
        sp.subdivision_name,
        sp.subdivision_geom_2236,
        sp.representative_point_2236,
        park.park_id,
        park.park_name,
        park.park_geom_2236,
        ST_Distance(
            sp.representative_point_2236,
            park.park_geom_2236
        ) AS distance_ft
    FROM subdivision_points AS sp
    CROSS JOIN LATERAL (
        SELECT
            p.park_id,
            p.park_name,
            p.geom_2236 AS park_geom_2236
        FROM marion_parks AS p
        WHERE p.geom_2236 IS NOT NULL
          AND NOT ST_IsEmpty(p.geom_2236)
        ORDER BY p.geom_2236 <-> sp.representative_point_2236,
                 p.park_id
        LIMIT 1
    ) AS park
)
SELECT
    subdivision_id,
    subdivision_name,
    park_id,
    park_name,
    ROUND(distance_ft::numeric, 1) AS distance_ft,
    ST_X(representative_point_2236) AS representative_x_2236,
    ST_Y(representative_point_2236) AS representative_y_2236
FROM nearest_park_matches
WHERE distance_ft <= 5280
ORDER BY distance_ft, subdivision_name, subdivision_id;
```

The selected park is preserved with its identifier and name. The representative point is exposed as coordinates so the report can show what was actually used. The distance is calculated after the lateral subquery returns one park.

The secondary `park_id` sort in the KNN ordering gives the query a stable tie-breaker when two parks have the same KNN distance. The final sort is a reporting choice. It makes the closest matches appear first, while the subdivision name and ID keep the order readable and repeatable.

## The Pattern

Match once, preserve the evidence, report last.

1. Validate the source geometry.
2. Create the representative point used by the lookup.
3. Keep the prepared EPSG:2236 geometry visible in the query structure.
4. Use `CROSS JOIN LATERAL` with `<->` and `LIMIT 1` to select one nearest park.
5. Calculate `ST_Distance` for that selected pair.
6. Return identifiers, names, point evidence, and `distance_ft`.
7. Apply final filters and sorting only after the nearest matches exist.

This separation prevents a common reporting mistake: treating a display rule as if it were part of the matching logic. The nearest park should be selected first. The report can then decide what to show, how to round it, and how to order it.

## AI Pairing Prompt

Review this PostGIS nearest-park reporting query as a careful coding partner.

Check that:

- the subdivision geometry and park geometry are prepared in EPSG:2236;
- invalid, empty, and null subdivision geometries are excluded before `ST_PointOnSurface`;
- the representative point is clearly distinguished from the source subdivision geometry;
- `CROSS JOIN LATERAL`, `<->`, and `LIMIT 1` select one nearest eligible park per subdivision;
- `ST_Distance` is calculated for the selected park and reported as `distance_ft`;
- the park eligibility filter is not confused with the final reporting filter;
- the KNN tie-breaker is deterministic and does not interfere with index use;
- the final sort is clearly a presentation choice;
- the returned columns provide enough evidence for someone to audit the result.

Explain the query in plain English first. Then list any correctness, unit, geometry-health, indexing, or interpretation risks. Suggest the smallest safe improvement and keep the bulk KNN pattern intact.

## Why This Matters

A nearest-park table is more useful when it shows how the answer was produced. Without the representative point, projected geometry context, selected park ID, and distance field, a reader may mistake the result for a direct measurement from every property or assume that the park name alone is enough evidence.

Clear reporting also protects the analysis from overclaiming. `distance_ft` supports a measured map-distance statement between two geometries. It does not support a walking-route statement unless a network analysis was run separately.

The practical goal is a result that can move from SQL to a report without losing its meaning.

## Next Steps

- Add a source or geometry status column when rows are excluded before matching.
- Compare rounded display values with the unrounded `distance_ft` used for filtering and sorting.
- Review whether park boundaries or park representative points are the correct target geometry for the use case.
- Test deterministic results when two parks are effectively tied.
- Keep the final report explicit about the representative-point assumption and the units of `distance_ft`.