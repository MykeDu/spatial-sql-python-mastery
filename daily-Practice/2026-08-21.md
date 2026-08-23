# Day 15 – August 21, 2026 – Daily Practice Log

## Focus:
- Aggregating geometries with ST_Union
- Using a CTE to keep spatial summaries readable
- Comparing PostGIS geometry aggregation with GeoPandas dissolve()

## Activities Planned:
- Micro-lesson: Understand the difference between counting records and combining geometries
- Practice problem: Create one dissolved neighborhood geometry for each district
- Prompt drill: Draft an AI prompt that requests a CTE-based spatial aggregation with ST_Union

## Key Learning Goals:
- Explain what ST_Union returns and when it is useful
- Group neighborhood polygons by a shared district value
- Avoid the GROUP BY geometry trap by aggregating geometry intentionally
- Use GeoPandas dissolve() when a grouped result needs to retain geometry

## Sample Prompt (Planned):
"Write a PostGIS SQL query to dissolve neighborhood polygons into one geometry per district. Table: neighborhoods(id, name, district, geom). Use a CTE to calculate the neighborhood count and combined geometry for each district. Return the district name, neighborhood count, and dissolved geometry. Use ST_Union for the geometry aggregation. Explain the query in plain English, including why ST_Union is used instead of adding geom to the GROUP BY clause."

## Sample Query (Planned): PostGIS SQL:

```
WITH district_summary AS (
 SELECT
 district,
 COUNT(*) AS neighborhood_count,
 ST_Union(geom) AS geom
 FROM neighborhoods
 GROUP BY district
)
SELECT
 district,
 neighborhood_count,
 geom
FROM district_summary
ORDER BY district;

```

## GeoPandas Python:

```
district_summary = (
 neighborhoods
 .dissolve(by="district", aggfunc={"id": "count"})
 .rename(columns={"id": "neighborhood_count"})
 .reset_index()
)

```

## Reflection (Planned):
- ST_Union combines the geometries in each group into a single output geometry.
- The query groups by district because that is the reporting unit. The geometry is intentionally aggregated with ST_Union, rather than placed in GROUP BY.
- n GeoPandas, dissolve() is the geometry-aware equivalent of grouping features and combining their shapes.
- Next, I’ll inspect the dissolved district layer in QGIS, checking for unexpected gaps, overlaps, and multipart geometries.

## Code Comments
Plain-English read: group neighborhoods by district, count the neighborhoods in each group, and combine their polygons into one district geometry.

This is a useful next step because it builds directly on your CTE and aggregation work while adding a new portfolio-level output: a clean dissolved layer. 

Before publishing, replace the sample table and field names with real data you.
