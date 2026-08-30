# Day 22: PostGIS KNN Nearest Neighbor Queries

## Focus
Using PostGIS KNN nearest-neighbor ordering to find the closest spatial features efficiently. Today’s pattern was finding the five fire hydrants nearest to one incident.

What I practiced
I wrote a CTE-based query that isolates the target incident geometry, then orders fire hydrants with the PostGIS <-> operator.

```sql

WITH target AS (
    SELECT geom
    FROM incidents
    WHERE id = 123
)
SELECT h.id
FROM fire_hydrants h, target
ORDER BY h.geom <-> target.geom
LIMIT 5;

```
Read like a sentence: select hydrants, compare each one to the target incident, order them from nearest to farthest, and return five rows.

I also practiced identifying the performance trap:

```sql

ORDER BY ST_Distance(h.geom, target.geom)

```

ST_Distance() is useful when I need an exact distance value. For nearest-neighbor ordering, <-> is the better first pattern because it supports an index-friendly KNN query.

I caught an important CRS detail too. With EPSG:4326 geometry, distance values are in degrees, not meters. I need a suitable projected CRS or geography when reporting or filtering by meters.

## Why this pattern matters

Nearest-feature analysis shows up constantly in GIS work: closest hydrant to an incident, nearest park to a building, nearest station to an address, or nearest parcel to a point.

This pattern matters because it keeps the query clear and helps PostGIS use spatial indexes instead of calculating and sorting every possible distance. The implementation rule I want to remember is simple: use <-> to find the nearest rows, then use ST_Distance() only when I also need the exact distance.

## Pattern

Use PostGIS's <-> operator to find the nearest features efficiently.
Plain English: select the five hydrants closest to incident 123.

## The performance trap

Avoid sorting a large table with ST_Distance() when the goal is simply to find the nearest rows.

```sql

-- Slow pattern on large tables
SELECT h.id
FROM fire_hydrants AS h
CROSS JOIN incidents AS i
WHERE i.id = 123
ORDER BY ST_Distance(h.geom, i.geom)
LIMIT 5;

```

ST_Distance() calculates a distance for every candidate before sorting. That can become expensive as the table grows.

## The KNN pattern

Use <-> in ORDER BY for an index-friendly nearest-neighbor query.

```sql

WITH target_incident AS (
 SELECT geom
 FROM incidents
 WHERE id = 123
)
SELECT h.id,
 h.geom
FROM target_incident AS i
CROSS JOIN fire_hydrants AS h
ORDER BY h.geom <-> i.geom
LIMIT 5;

```

## Read the query like a sentence

Select hydrant IDs and geometries from fire_hydrants, pair them with the one target incident, order hydrants by proximity to that incident, and return the first five.

## Why the CTE and ORDER BY?

- CTE (target_incident): isolates the one geometry I am measuring from.
- ORDER BY <->: expresses nearest-neighbor ordering and can use a spatial index.

## CRS and units caveat

If geom uses EPSG:4326, coordinates are longitude and latitude. Distance values are in degrees, not meters.

For meter-based distance reporting or filtering, use an appropriate projected CRS or cast geometries to geography when that fits the workflow.

```sql

-- Example: report distance in meters with geography
WITH target_incident AS (
 SELECT geom
 FROM incidents
 WHERE id = 123
)
SELECT h.id,
 ST_Distance(h.geom::geography, i.geom::geography) AS distance_meters
FROM target_incident AS i
CROSS JOIN fire_hydrants AS h
ORDER BY h.geom <-> i.geom
LIMIT 5;

```

## AI pairing prompt

Please refactor this slow PostGIS query to be more performant for a nearest neighbor search. 
It should use the index-friendly `<->` operator instead of `ST_Distance` in the `ORDER BY` clause.

SELECT b.id
FROM buildings b, parks p
WHERE p.id = 42
ORDER BY ST_Distance(b.geom, p.geom)
LIMIT 10;

## Takeaways

1. For nearest-feature searches in PostGIS, reach for <-> first
2. Use ST_Distance() when you need an exact distance value
3. Check the CRS before interpreting that value.