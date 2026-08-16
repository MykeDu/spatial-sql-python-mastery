# Day 9 – August 15, 2026 – Daily Practice Log

## Focus:
- Consolidating understanding of SQL JOIN mechanics
- Visualizing how spatial predicates like ST_Within function within the ON clause
- Practicing AI prompt engineering for JOIN explanations

## Activities Planned:
- Micro-lesson: JOIN types and their impact on spatial queries
- Practice problem: Join crime points with police district polygons
- Prompt drill: Draft an AI prompt to explain SQL JOIN code and workflow

## Key Learning Goals:
- Distinguish between INNER JOIN and LEFT JOIN in spatial SQL
- Translate SQL join results into GeoPandas for visual verification
- Refine AI prompts to clarify join logic and expected outputs

## Sample Prompt (Planned):
"Explain how to join crime points to police district polygons in PostGIS SQL. Provide both the SQL query and a GeoPandas workflow for loading and visualizing the results."

## Sample Query (Planned): PostGIS SQL:

**SQL**
```
SELECT c.id, d.district_name
FROM crimes c
JOIN districts d ON ST_Within(c.geom, d.geom);
```
**GeoPandas Python:**
```
joined = gpd.sjoin(crimes, districts, predicate='within')
```
## Reflection (Planned):
- Practicing JOIN logic with real data helped clarify the difference between join types
- Using GeoPandas to visualize join results made the SQL logic more concrete
- Next, I’ll continue testing queries on local datasets and refine my mental model of how the database engine processes joins
