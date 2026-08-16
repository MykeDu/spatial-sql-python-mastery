# Day 8 – August 14, 2026 – Daily Practice Log

## Focus:
- Mastering proximity queries with ST_DWithin and ST_Within
- Understanding why spatial joins are needed for comparing features across tables
- Improving AI prompt specificity for spatial join tasks

## Activities Planned:
- Micro-lesson: Efficient proximity filtering with ST_DWithin vs ST_Distance
- Practice problem: Find all schools within 500 meters of any park
- Prompt drill: Draft an AI prompt to generate a spatial join query using ST_DWithin

## Key Learning Goals:
- Know when to use ST_DWithin for fast, index-friendly proximity checks
- Understand why spatial joins are required for cross-table spatial relationships
- Refine AI prompts to specify table names, geometry columns, and performance intent

## Sample Prompt (Planned):
"Write a PostGIS query to find all schools within 500 meters of any park. Tables: schools(id, geom), parks(id, geom). Both in a projected CRS using meters. Use ST_DWithin for efficiency."

Sample Query (Planned): PostGIS SQL:
```
SELECT DISTINCT s.name
FROM schools s
JOIN parks p ON ST_DWithin(s.geom, p.geom, 500);

```
GeoPandas Python:
schools_in_range = gpd.sjoin_nearest(schools, parks, max_distance=500)

## Reflection (Planned):
- Today clarified why a join is needed for proximity queries between two tables—SQL needs to compare every school to every park
- Practicing the plain-English read of the query helped cement the logic
- Next, I’ll keep refining my AI prompts to specify spatial intent and performance needs, and test these patterns on real datasets
