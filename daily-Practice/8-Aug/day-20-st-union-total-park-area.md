# Day 20: Merging Geometries with ST_Union

## Today’s focus

Today was a Spatial SQL day focused on ST_Union.
The goal was simple: learn how to take multiple geometries and merge them into a single clean geometry by dissolving interior boundaries. In plain English, this is the pattern for turning many overlapping shapes into one usable result.

## What I learned

1. What ST_Union does
ST_Union merges geometries into a single geometry.
This is different from just collecting shapes together. The important behavior is that ST_Union dissolves shared boundaries and returns a cleaned, combined spatial result.

2. ST_Union vs ST_Collect
A useful distinction:
- ST_Collect groups geometries together without dissolving them
- ST_Union merges geometries and dissolves interior overlaps/boundaries

That makes ST_Union the right tool when the goal is one real merged output geometry instead of just a grouped container of shapes.

3. The grouped dissolve pattern

I also worked with the pattern of using ST_Union alongside GROUP BY.

That pattern is useful when I want one merged geometry per category, such as:
- one geometry per zoning type
- one geometry per district
- one geometry per region

## Practice problem

Today’s practice problem was to create a single merged geometry representing all parkland in a target area.

## Problem setup
I had:
- a parks table with polygon geometries
- a districts table with polygon geometries

The task was to produce one merged geometry for parks in the Downtown district.

## My planning logic

Before writing the SQL, I broke the workflow into steps:

1. Start with the parks table
2. Spatially join parks to districts
3. Filter to the district where d.name = 'Downtown'
4. Aggregate the matching park geometries with ST_Union
5. Return one final merged geometry

That planning step mattered because it made the SQL feel like a sentence instead of random syntax.

## Query pattern in plain English

Select all park geometries that intersect the Downtown district, then merge those park geometries into one single output geometry.

That is the core pattern I want to keep building: join → filter → aggregate.

## Example SQL

```sql 

SELECT ST_Union(p.geom) AS downtown_parks_geom
FROM parks p
JOIN districts d
 ON ST_Intersects(p.geom, d.geom)
WHERE d.name = 'Downtown';

```

## Performance note

I also called out the indexing pattern that supports this kind of workflow in PostGIS:

```sql

CREATE INDEX idx_parks_geom ON parks USING GIST (geom);
CREATE INDEX idx_districts_geom ON districts USING GIST (geom);

```

That is a good reminder that spatial SQL is not just about writing a correct query. It is also about writing queries that will still work when the data gets bigger.

## Prompt drill

The prompt drill focused on asking AI to build a multi-step spatial workflow clearly.

My prompt was:

```prompt

Write a PostGIS query. 
The tables are: subway_stations (points) and a table of parks (polygons). 
First, find all parks that are within 500 meters of any subway station. 
Then, then merge all of those selected parks into a single unioned geometry. Make sure the tables are in a projected CRS using meters.

```

## Why that prompt works

What worked well in that prompt:
- it named the input tables
- it specified the geometry types
- it described the workflow in sequence
- it included the distance constraint
- it called out the CRS requirement so the units make sense

That is a useful reminder that good spatial prompting starts with spatial thinking. Describe the process clearly first. Then verify the generated SQL.

## Key takeaway

- Today reinforced a very practical spatial SQL pattern:
- use a spatial predicate to identify the right features
- filter to the target subset
- aggregate the result with ST_Union

That pattern is reusable far beyond parks. It applies to service areas, land ownership aggregation, zoning dissolves, habitat grouping, and many other GIS workflows.

## Win from today

A specific win from today was recognizing that this was not just an ST_Union lesson. It was a broader pattern lesson about composing spatial SQL in logical steps.

I practiced the structure:
join → filter → aggregate
That is the kind of repeatable pattern that turns one lesson into long-term fluency.

## If I extend this tomorrow

A good follow-up would be to run the same workflow in Python and compare the logic side by side:
- SQL version using ST_Intersects + ST_Union
- GeoPandas version using spatial filtering/join logic plus a union step

That would make the cross-tool pattern even more concrete.

## Tags

postgis spatial-sql st-union gis sql portfolio-project daily-practice