# Day 29: Index-Aware Proximity Analysis with ST_DWithin

Build a multi-step PostGIS query that counts schools within 500 meters of a fire station, grouped by neighborhood.

The main goal was to combine chained CTEs, proximity filtering, spatial joins, de-duplication, and aggregation into one readable analysis.

## What I Learned

ST_DWithin is the better default for proximity filtering when I only need a yes-or-no distance test.
Instead of building a buffer polygon around each fire station with ST_Buffer, ST_DWithin directly checks whether a school falls within the specified distance. In PostGIS, this pattern can take advantage of spatial indexing.

I also reinforced that the distance value passed to ST_DWithin uses the units of the geometry column’s CRS. For a 500-meter search, the data needs to use a projected CRS with meter-based units, or a deliberate geography workflow.

## What I Practiced

- Chaining CTEs to separate spatial-analysis steps.

- Using ST_DWithin to identify schools near fire stations.

- Using DISTINCT to prevent schools near multiple stations from being counted more than once.

- Using a LEFT JOIN so neighborhoods with zero qualifying schools remain in the output.

- Using ST_Covers to include schools located directly on a neighborhood boundary.

- Aggregating results with COUNT() and sorting neighborhoods by school count.

## The Pattern

```sql

WITH schools_near_stations AS (
 -- Find schools within 500m of any fire station.
 -- DISTINCT prevents duplicate schools when multiple stations qualify.
 SELECT DISTINCT
 s.school_id,
 s.name,
 s.geom
 FROM schools AS s
 JOIN fire_stations AS f
 ON ST_DWithin(s.geom, f.geom, 500)
),
schools_with_neighborhood AS (
 -- Attach each qualifying school to a neighborhood.
 -- LEFT JOIN retains neighborhoods with zero qualifying schools.
 SELECT
 n.name AS neighborhood_name,
 sns.school_id,
 sns.name AS school_name
 FROM neighborhoods AS n
 LEFT JOIN schools_near_stations AS sns
 ON ST_Covers(n.geom, sns.geom)
)
SELECT
 neighborhood_name,
 COUNT(school_id) AS school_count
FROM schools_with_neighborhood
GROUP BY neighborhood_name
ORDER BY school_count DESC;

```

Read the query like a sentence:

1. Find every school within 500 meters of a fire station.
2. Keep each school only once.
3. Assign qualifying schools to neighborhoods.
4. Count the schools in each neighborhood.
5. Return the highest counts first.

Use ST_Buffer when the buffer polygon itself is needed for mapping or another overlay operation. Use ST_DWithin when the question is simply whether two features are within a specified distance.

For larger datasets, add GiST indexes to the geometry columns:

```sql

CREATE INDEX schools_geom_gix ON schools USING GIST (geom);
CREATE INDEX fire_stations_geom_gix ON fire_stations USING GIST (geom);
CREATE INDEX neighborhoods_geom_gix ON neighborhoods USING GIST (geom);



```

## AI Pairing Prompt

Review this PostGIS proximity query for production readiness.

The analysis should count schools within 500 meters of any fire station, grouped by neighborhood. 

Check the query for:
1. Correct use of ST_DWithin versus ST_Buffer.

2. CRS and distance-unit assumptions.

3. Duplicate-counting risks when a school is near multiple stations.

4. Whether neighborhoods with zero qualifying schools remain in the result.

5. Boundary behavior for schools on a neighborhood edge.

6. Spatial-index recommendations.

Explain each recommendation in plain English, then provide a revised query only where a change is necessary.


## Why This Matters

This is a real-world spatial SQL pattern, not a one-off exercise.

Proximity analysis appears in emergency response, service coverage, site selection, accessibility studies, and public-facility planning. 

The professional move is choosing the lightest function that answers the question, then structuring the SQL so someone else can read, test, and extend it.

The win today was not just writing a working query. It was recognizing that ST_DWithin expresses the proximity question more directly and is the stronger production pattern for an index-aware filter.


## Next Steps

- Run the query with a real dataset using a meter-based projected CRS.

- Inspect the output for neighborhoods with zero counts and boundary cases.

- Compare EXPLAIN ANALYZE results before and after adding GiST indexes.

- Add a map, result screenshot, data-source notes, and this writeup to a GitHub portfolio repository.

- Extend the pattern by calculating the nearest fire station for each school using a PostGIS KNN query.



