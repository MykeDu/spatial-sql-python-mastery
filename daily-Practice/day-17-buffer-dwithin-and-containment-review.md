# Day 17 – August 23, 2026 – Daily Practice Log

## Focus:
- Reviewing ST_Buffer, ST_DWithin, and ST_Within as three different spatial patterns
- Choosing the function based on the required output: a geometry, a yes-or-no test, or a containment relationship
- Checking distance units before running proximity analysis

## Activities Planned:
- Micro-lesson: Compare buffer creation, proximity testing, and containment testing
- Practice problem: Find libraries within 1,000 meters of schools that fall inside the Lakeview neighborhood
- Prompt drill: Ask AI to explain which spatial function belongs to each part of the workflow

## Key Learning Goals:
- Use ST_DWithin to test whether features are within a specified distance
- Use ST_Within to test whether one geometry is contained by another
- Use ST_Buffer only when the analysis needs a new zone geometry for mapping, export, or a later overlay
- Use geography or an appropriate projected CRS when the distance rule is measured in meters

## Sample Prompt (Planned):
"Write a PostGIS SQL query to find libraries within 1,000 meters of schools located inside the neighborhood named 'Lakeview'. Tables: schools(id, name, geom), libraries(id, name, geom), and neighborhoods(id, name, geom). The geometry columns use EPSG:4326. Use ST_Within to keep schools inside Lakeview and ST_DWithin with ::geography to find nearby libraries. Return the school name, library name, and distance in meters. Then provide a separate query that creates a 1,000-meter buffer geometry around each Lakeview school for map display. Explain why the two queries use different functions."

## Sample Query (Planned): PostGIS SQL

```
SELECT
 s.name AS school_name,
 l.name AS library_name,
 ST_Distance(
 s.geom::geography,
 l.geom::geography
 ) AS distance_meters
FROM schools AS s
JOIN neighborhoods AS n
 ON ST_Within(s.geom, n.geom)
JOIN libraries AS l
 ON ST_DWithin(
 s.geom::geography,
 l.geom::geography,
 1000
 )
WHERE n.name = 'Lakeview'
ORDER BY school_name, distance_meters;

-- Optional buffer output for mapping:

SELECT
 s.id,
 s.name AS school_name,
 ST_Buffer(s.geom::geography, 1000)::geometry AS buffer_geom
FROM schools AS s
JOIN neighborhoods AS n
 ON ST_Within(s.geom, n.geom)
WHERE n.name = 'Lakeview';

```

## GeoPandas Python:

```
lakeview = neighborhoods.loc[
 neighborhoods["name"] == "Lakeview",
 ["name", "geometry"]
]

schools_projected = schools_in_lakeview.to_crs("EPSG:26916") libraries_projected = libraries.to_crs("EPSG:26916")
nearby_libraries = gpd.sjoin_nearest( schools_projected[["name", "geometry"]], libraries_projected[["name", "geometry"]], how="left", max_distance=1000, distance_col="distance_meters", lsuffix="school", rsuffix="library" )

school_buffers = schools_projected.copy() 
school_buffers["geometry"] = school_buffers.buffer(1000)

```

## Reflection (Planned): 
- ST_Within answers a containment question: is the school inside Lakeview?
- ST_DWithin answers a distance-rule question: is the library within 1,000 meters of the school?
- ST_Buffer creates a new polygon geometry. I use it when I need to map or export the 1,000-meter zone, not just test the distance rule.
- ST_Distance reports the measured distance after the relevant features have been selected.
- Before trusting the result, I will verify the CRS, inspect several matches in QGIS, and confirm that the buffer and nearby-library results make sense visually.

## Code Comments
Plain-English read: select schools inside Lakeview, find libraries no more than 1,000 meters away, then report the measured distance. Create a buffer only when you need to see or reuse the zone itself.

This makes the first 17 days read like a useful progression: relationships, aggregation, proximity, geometry creation, and validation. 

Your 17-day streak is now matched by a clear implementation trail in GitHub.

## 

## 

## 