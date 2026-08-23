# Day 12 – August 18, 2026 – Daily Practice Log

## Focus:
- Practicing proximity analysis with ST_DWithin
- Using meter-based distance calculations correctly in PostGIS
- Comparing a direct distance test with a buffer-based workflow

## Activities Planned:
- Micro-lesson: Choosing ST_DWithin for proximity searches
- Practice problem: Find libraries within 1,000 meters of schools
- Prompt drill: Draft an AI prompt that specifies the spatial relationship, distance, units, and output fields

## Key Learning Goals:
- Explain when ST_DWithin is a better fit than ST_Buffer plus ST_Intersects
- Use ::geography when calculating distances in meters from longitude/latitude data
- Apply the same proximity pattern in PostGIS and GeoPandas

## Sample Prompt (Planned):
"Write a PostGIS SQL query to find libraries within 1,000 meters of each school. Tables: schools(id, name, geom) and libraries(id, name, geom). The geometry columns use EPSG:4326. Cast geometries to geography so the distance is measured in meters. Return the school name, library name, and distance in meters. Explain the query in plain English."

## Sample Query (Planned): PostGIS SQL:

```
SELECT
 s.name AS school_name,
 l.name AS library_name,
 ST_Distance(s.geom::geography, l.geom::geography) AS distance_meters
FROM schools AS s
JOIN libraries AS l
 ON ST_DWithin(
 s.geom::geography,
 l.geom::geography,
 1000
 )
ORDER BY distance_meters;

```

## GeoPandas Python:

```
schools_projected = schools.to_crs("EPSG:26916")
libraries_projected = libraries.to_crs("EPSG:26916")
nearby = gpd.sjoin_nearest( schools_projected, libraries_projected[["name", "geometry"]], how="inner", max_distance=1000, distance_col="distance_meters" )

```

## Reflection (Planned):
- ST_DWithin directly tests whether two features are within a specified distance, so it is usually cleaner than creating buffers first.
- Casting EPSG:4326 geometries to geography is important when the distance value is in meters.
- In GeoPandas, I need to project the data to an appropriate projected CRS before using meter-based distances or buffers.
- Next, I’ll test this pattern with a small local dataset and inspect the results in QGIS before applying it to a larger workflow.

