# Day 14 – August 20, 2026 – Daily Practice Log

## Focus:
- Using Common Table Expressions (CTEs) to organize multi-step spatial SQL
- Separating a non-spatial aggregation from geometry retrieval
- Applying a readable aggregation pattern in PostGIS and GeoPandas

## Activities Planned:
- Micro-lesson: Use a CTE to name and isolate an intermediate query result
- Practice problem: Count buildings in each neighborhood, then return the neighborhood geometry with the summary
- Prompt drill: Draft an AI prompt that requests a CTE-based spatial aggregation without grouping by geometry unnecessarily

## Key Learning Goals:
- Explain what a CTE does and why it improves query readability
- Aggregate building counts by neighborhood ID before joining back to neighborhood geometry
- Avoid grouping directly on a geometry column when the geometry is only needed in the final output

## Sample Prompt (Planned):
"Write a PostGIS SQL query to count buildings in each neighborhood. Tables: buildings(id, geom) and neighborhoods(id, name, geom). First, use a CTE to count buildings by neighborhood ID with a spatial join. Then join that summary back to the neighborhoods table to return the neighborhood name, building count, and geometry. Use ST_Within in the JOIN condition. Explain the query in plain English and explain why the geometry is not included in the GROUP BY step."

## Sample Query (Planned): PostGIS SQL:

```
WITH building_counts AS (
 SELECT
 n.id AS neighborhood_id,
 COUNT(b.id) AS building_count
 FROM neighborhoods AS n
 LEFT JOIN buildings AS b
 ON ST_Within(b.geom, n.geom)
 GROUP BY n.id
)
SELECT
 n.name AS neighborhood_name,
 bc.building_count,
 n.geom
FROM neighborhoods AS n
JOIN building_counts AS bc
 ON n.id = bc.neighborhood_id
ORDER BY bc.building_count DESC;

```

## GeoPandas Python:

```
building_counts = (
 gpd.sjoin(
 buildings,
 neighborhoods[["id", "geometry"]],
 predicate="within"
 )
 .groupby("id_right")
 .size()
 .reset_index(name="building_count")
)

neighborhood_summary = neighborhoods.merge( building_counts, left_on="id", right_on="id_right", how="left" ).fillna({"building_count": 0})

```

## Reflection (Planned):
- A CTE is a named intermediate result. It lets me solve the counting problem first, then attach geometry in a separate final step.
- The spatial relationship belongs in the JOIN condition: count buildings where each building falls within a neighborhood.
- Separating aggregation from geometry retrieval makes the query easier to read, test, and reuse.
- Next, I’ll run this against a small local dataset and compare the summary layer with the source buildings and neighborhoods in QGIS.

## Code Comments

Plain English: first count the buildings that fall inside each neighborhood. Then attach those counts back to the neighborhood polygons so the result can be mapped or exported.

This fits the actual portfolio sequence well. It turns the Day 10 join-and-aggregation idea into a cleaner, reusable query structure.
