# Day 25: Spatial Aggregation with ST_Union and ST_Collect

Today I practiced two PostGIS functions for combining multiple geometries:
- ST_Collect
- ST_Union

They sound similar, but they produce different results.

## The Difference

### ST_Collect

ST_Collect puts geometries into one multi-part geometry without changing their boundaries.
In plain English: collect these park polygons into one container, but keep each park shape separate.

```sql
SELECT
 ST_Collect(geom) AS parks_collection
FROM parks;
```
Use ST_Collect when I need to group geometries together but still preserve the individual features.

### ST_Union

ST_Union combines geometries and dissolves overlapping or touching boundaries.
In plain English: merge these park polygons into one cleaned-up shape.

```sql
SELECT
 ST_Union(geom) AS parks_union
FROM parks;

```

Use ST_Union when I need a dissolved boundary for analysis, export, or map display.

## Side-by-Side Comparison
| Function |

| What it does | Boundaries preserved? | Common use |

| ST_Collect | 

| Groups geometries into a multi-part geometry | Yes | Keep individual shapes together | 

| ST_Union | 

| Merges and dissolves geometries | No, overlaps and shared edges are removed | Create one combined area |

## Practice Query

This query merges all park polygons in each neighborhood:

``` sql

SELECT
 neighborhood_name,
 ST_Union(geom) AS merged_parks
FROM parks
GROUP BY neighborhood_name;

```

**Read it like a sentence:**

Select each neighborhood name and merge all park geometries within that neighborhood into one geometry.

## AI Pairing Prompt

I have a PostGIS parks table with neighborhood_name and geom columns. Write one query using ST_Collect and a second query using ST_Union, grouped by neighborhood. Explain when I would use each result in a real GIS workflow.

## What I Learned

ST_Collect is about grouping geometries. ST_Union is about merging geometries.

Choosing the right function depends on whether I need to preserve individual feature boundaries or dissolve them into one analysis-ready shape.

## Next Step

Use both functions on a small real dataset, then compare the output in QGIS or another map viewer.




