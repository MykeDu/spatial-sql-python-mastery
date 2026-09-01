# Day 11 – August 17, 2026 – Daily Practice Log

## Focus:
- Building a public GitHub portfolio through concise daily practice logs
- Moving from QGIS and MSSQL workflows toward a local PostGIS container
- Applying SQL JOIN mechanics in repeatable spatial analysis workflows

## Activities Planned:
- Micro-lesson: Define a practical project structure for spatial SQL practice
- Practice problem: Use a spatial join to connect features and inspect matched records
- Prompt drill: Draft an AI prompt that requests a minimal, testable PostGIS spatial join

## Key Learning Goals:
- Document spatial SQL practice in a format that shows both code and reasoning
- Explain the difference between INNER JOIN and LEFT JOIN in a spatial workflow
- Build a small, repeatable PostGIS test setup before applying queries to larger datasets

## Sample Prompt (Planned):
"I am moving a spatial workflow from QGIS/MSSQL to PostGIS. Write a minimal - PostGIS example that joins points to polygons using ST_Within. Show both an INNER JOIN and a LEFT JOIN, explain the difference in the returned rows, and keep the query easy to test with small sample tables."

## Sample Query (Planned): PostGIS SQL:

```
SELECT
 p.id AS point_id,
 a.name AS area_name
FROM points AS p
LEFT JOIN areas AS a
 ON ST_Within(p.geom, a.geom);

 ```

## Reflection (Planned):
- Building the log as I practice creates a visible record of real spatial SQL patterns.
- A small PostGIS test database will make JOIN behavior easier to inspect than relying on larger production-style data.
- The key distinction is now clearer: use an INNER JOIN when unmatched features are not needed, and a LEFT JOIN when every feature from the left table must remain in the result.
- Next, I’ll create a minimal local dataset and verify the query output in both PostGIS and QGIS.