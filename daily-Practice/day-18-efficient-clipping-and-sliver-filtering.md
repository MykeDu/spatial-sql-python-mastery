# Day 18 – August 24, 2026 – Daily Practice Log

## Focus

Learning the clipping pattern with ST_Intersection
Using ST_Intersects as a pre-filter before running expensive geometry operations
Avoiding sliver geometries when buildings only touch a district boundary
Choosing area thresholds based on whether the source data is measured in feet or meters

## Activities Planned

- Micro-lesson: Learn how ST_Intersection clips one geometry to another
- Practice problem: Clip building footprints to the Downtown historic district
- Prompt drill: Ask AI to refactor a clipping query so it filters candidates before computing intersections

## Key Learning Goals

- Use ST_Intersection to return the exact geometry shared by two overlapping features
- Use ST_Intersects before ST_Intersection to avoid running expensive clipping operations on every feature
- Use ST_Dimension to remove point-only or line-only boundary contacts
- Use ST_Area to remove tiny sliver polygons from the output
- Check the CRS before applying area thresholds, because square meters and square feet require different cutoff values

## Sample Prompt (Planned)

"Write a PostGIS SQL query to clip building footprints to the historic district named 'Downtown'. Tables: buildings(building_id, geom) and districts(district_id, name, geom). First isolate the Downtown district in a CTE. Then join buildings to the district using ST_Intersects so the query only clips buildings that overlap the district. Use ST_Intersection to return the clipped building geometry. Add filters to remove empty results, point-only or line-only boundary touches, and tiny sliver polygons. Explain how the area threshold should change if the data is stored in meters versus feet."

## Sample Query (Planned): PostGIS SQL

```sql

WITH target_district AS (
 SELECT
 geom
 FROM districts
 WHERE name = 'Downtown'
),
clipped AS (
 SELECT
 b.building_id,
 b.geom AS original_geom,
 ST_Intersection(b.geom, d.geom) AS clipped_geom
 FROM buildings AS b
 JOIN target_district AS d
 ON ST_Intersects(b.geom, d.geom)
)
SELECT
 building_id,
 clipped_geom
FROM clipped
WHERE NOT ST_IsEmpty(clipped_geom)
 AND ST_Dimension(clipped_geom) = 2
 AND ST_Area(clipped_geom) >= 10
 AND ST_Area(clipped_geom) / NULLIF(ST_Area(original_geom), 0) >= 0.05;


-- Notes on Feet vs Meters:
-- If the data is in meters:
-- Start with a minimum area threshold around 10 to 25 square meters.
AND ST_Area(clipped_geom) >= 10
-- If the data is in feet: -- Start with a minimum area threshold around 100 to 250 square feet.
AND ST_Area(clipped_geom) >= 100
--The relative area filter is useful in both cases:
AND ST_Area(clipped_geom) / NULLIF(ST_Area(original_geom), 0) >= 0.05

```

This keeps clipped pieces that represent at least 5% of the original building footprint.

## GeoPandas Python:

```python

target_district = districts.loc[
 districts["name"] == "Downtown",
 ["name", "geometry"]
]
buildings_projected = buildings.to_crs("EPSG:26916") district_projected = target_district.to_crs("EPSG:26916")
candidate_buildings = gpd.sjoin( buildings_projected, district_projected, how="inner", predicate="intersects" )
clipped_buildings = candidate_buildings.copy()
district_geom = district_projected.geometry.iloc[0]
clipped_buildings["original_area"] = clipped_buildings.geometry.area clipped_buildings["geometry"] = clipped_buildings.geometry.intersection(district_geom) clipped_buildings["clipped_area"] = clipped_buildings.geometry.area
clipped_buildings = clipped_buildings.loc[ (~clipped_buildings.geometry.is_empty) & (clipped_buildings.geometry.geom_type.isin(["Polygon", "MultiPolygon"])) & (clipped_buildings["clipped_area"] >= 10) & ( clipped_buildings["clipped_area"] / clipped_buildings["original_area"]
= 0.05 ) ]

```

## Reflection (Planned)

- ST_Intersects answers the candidate question: does this building touch or overlap the district?
- ST_Intersection answers the clipping question: what exact piece of the building is inside the district?
- ST_Dimension helps remove boundary-only contacts, such as a corner touch or edge touch.
- ST_Area helps remove tiny polygon scraps that are technically valid but not useful for analysis.
- Before trusting the output, I will verify the CRS, choose an area threshold in the correct units, and inspect several clipped features visually in QGIS.

## Code Comments

Plain-English read: select the Downtown district, find buildings that intersect it, clip those buildings to the district boundary, then remove empty results, boundary-only contacts, and tiny slivers.

The key pattern is filter cheaply first, compute geometry second. ST_Intersects narrows the candidate set. ST_Intersection does the heavier geometry work only after that screening step.

This extends the previous proximity and containment work into true overlay analysis. The first 18 days now show a clear implementation path: relationships, joins, aggregation, proximity, containment, clipping, and validation.
