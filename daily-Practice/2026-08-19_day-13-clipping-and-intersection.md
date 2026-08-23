# Day 13 – August 19, 2026 – Daily Practice Log

## Focus:
- Practicing clipping and intersection with ST_Intersection
- Separating the spatial relationship test from the geometry-producing function
- Applying the same clipping pattern in PostGIS and GeoPandas

## Activities Planned:
- Micro-lesson: Understand ST_Intersection as the function that returns the shared geometry
- Practice problem: Clip park polygons to a neighborhood boundary
- Prompt drill: Draft an AI prompt that requests a safe, readable clipping query with the correct spatial predicate

## Key Learning Goals:
- Explain the difference between ST_Intersects and ST_Intersection
- Use ST_Intersects to identify candidate feature pairs before calculating an intersection geometry
- Create a clipped output layer that can be reviewed in QGIS or used in a later analysis

## Sample Prompt (Planned):
"Write a PostGIS SQL query to clip park polygons to one neighborhood boundary. Tables: parks(id, name, geom) and neighborhoods(id, name, geom). Return each park name and the clipped geometry. Filter to the neighborhood named 'Lakeview'. Use ST_Intersects to limit the pairs before using ST_Intersection, and explain the query in plain English."

## Sample Query (Planned): PostGIS SQL:

```
SELECT
 p.name AS park_name,
 n.name AS neighborhood_name,
 ST_Intersection(p.geom, n.geom) AS geom
FROM parks AS p
JOIN neighborhoods AS n
 ON ST_Intersects(p.geom, n.geom)
WHERE n.name = 'Lakeview';

```
## GeoPandas Python:

```
lakeview = neighborhoods.loc[ neighborhoods["name"] == "Lakeview", ["name", "geometry"]]

clipped_parks = gpd.overlay( parks[["name", "geometry"]], lakeview, how="intersection" )

```

## Reflection (Planned):
- ST_Intersects answers a yes-or-no question: do these two geometries overlap at all?
- ST_Intersection creates the actual shared geometry, which is the clipped feature I can map, export, or measure.
- Using ST_Intersects in the join condition keeps the spatial relationship visible and avoids calculating intersections for pairs that do not overlap.
- Next, I’ll load the clipped result in QGIS and compare it with the original parks and neighborhood boundary as a visual quality check.

