# Day 16 – August 22, 2026 – Daily Practice Log

## Focus:
- Finding the nearest feature with a PostGIS KNN spatial index pattern
- Distinguishing nearest-neighbor analysis from a within-distance test
- Comparing PostGIS nearest-neighbor SQL with GeoPandas sjoin_nearest()

## Activities Planned:
- Micro-lesson: Use the <-> operator to order candidate geometries by proximity
- Practice problem: Find the nearest hospital for every school
- Prompt drill: Ask AI to write a nearest-neighbor query that returns both the matched feature and a meter-based distance

## Key Learning Goals:
- Explain when to use a nearest-neighbor pattern instead of ST_DWithin
- Use CROSS JOIN LATERAL to select one closest match for each source feature
- Use ST_Distance to report the actual distance after choosing the nearest candidate
- Project data to an appropriate local CRS, or use geography, when reporting distance in meters

## Sample Prompt (Planned):
"Write a PostGIS SQL query to find the nearest hospital for every school. Tables: schools(id, name, geom) and hospitals(id, name, geom). The geometry columns use EPSG:4326. Return each school name, the nearest hospital name, and the distance in meters. Use CROSS JOIN LATERAL and the <-> nearest-neighbor operator to select one hospital per school. Cast the geometries to geography when calculating the reported distance. Explain the query in plain English and explain why this is different from using ST_DWithin."

## Sample Query (Planned): PostGIS SQL:

```
SELECT
 s.name AS school_name,
 nearest_hospital.hospital_name,
 ST_Distance(
 s.geom::geography,
 nearest_hospital.geom::geography
 ) AS distance_meters
FROM schools AS s
CROSS JOIN LATERAL (
 SELECT
 h.name AS hospital_name,
 h.geom
 FROM hospitals AS h
 ORDER BY s.geom <-> h.geom
 LIMIT 1
) AS nearest_hospital
ORDER BY distance_meters;

```

## GeoPandas Python:

```
schools_projected = schools.to_crs("EPSG:26916")
hospitals_projected = hospitals.to_crs("EPSG:26916")

```

## Reflection (Planned):
- ST_DWithin answers, “Which features are within this distance?” A nearest-neighbor query answers, “Which one is closest?”
- CROSS JOIN LATERAL runs the nearest-hospital lookup once for each school.
- The <-> operator orders candidate hospitals by spatial proximity, then LIMIT 1 keeps the closest match.
- ST_Distance reports the measured result. It does not choose the nearest record by itself.
- Next, I’ll test the output on a small dataset and inspect a few school-to-hospital connections in QGIS to verify that the selected match makes sense.

## Code Comments
Plain-English read: for each school, find the one hospital closest to it, then report how far away that hospital is. You are extending the Day 12 distinction correctly: ST_DWithin filters by a rule, ST_Distance measures, and nearest-neighbor logic chooses the closest match.

Before publishing, replace the sample CRS, table names, and field names with the real data you actually run. 

That keeps the GitHub log a record of real implementation work.
