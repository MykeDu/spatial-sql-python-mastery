
# Day 24: Nearest Neighbor Pattern Review

## What I Did Today
Today was a review and maintenance day.
Instead of adding a new spatial function, I reinforced a pattern I practiced this week: finding the nearest feature efficiently.
I also organized my project files, planned the coming week, and committed my work to GitHub.

## Pattern Reviewed: Nearest Neighbor
The spatial question is simple:
For each source feature, what is the closest target feature?

**Examples:**
- Find the nearest hospital for every incident.
- Find the nearest fire station for every sinkhole.
- Find the nearest road, parcel, or service location for each point.

The syntax changes by tool, but the spatial logic stays the same.

## Spatial SQL Pattern

For a large SQL dataset, the efficient pattern is a CROSS JOIN LATERAL query with nearest-neighbor ordering.

```sql
SELECT
 s.sinkhole_id,
 fs.station_id,
 ST_Distance(s.geom, fs.geom) AS distance_m
FROM sinkholes AS s
CROSS JOIN LATERAL (
 SELECT
 station_id,
 geom
 FROM fire_stations
 ORDER BY s.geom <-> geom
 LIMIT 1
) AS fs;

```

Read the query like a sentence:

For each sinkhole, find the single fire station whose geometry is closest, then return that station and the distance.

**Why this pattern matters**

ORDER BY <-> can use a spatial index, which makes a major difference when the source and target tables are large.

The important implementation pattern is:

```
CROSS JOIN LATERAL
ORDER BY source_geom <-> target_geom
LIMIT 1

```

## GeoPandas Pattern

In GeoPandas, the equivalent operation is:

```python

gpd.sjoin_nearest()
nearest = gpd.sjoin_nearest(
 sinkholes,
 fire_stations,
 how="left",
 distance_col="distance_m"
)

```

Read the code like a sentence:

For each sinkhole, attach the nearest fire station, keep every sinkhole, and store the distance in meters.

Before calculating distance, both layers need a projected CRS with useful distance units.

```
sinkholes = sinkholes.to_crs("EPSG:26917")
fire_stations = fire_stations.to_crs("EPSG:26917")
```

## What I Confirmed

- The SQL nearest-neighbor pattern uses CROSS JOIN LATERAL, ORDER BY <->, and LIMIT 1.
- This approach is efficient because it can use a spatial index.
- The GeoPandas equivalent is gpd.sjoin_nearest().
- Distance calculations require projected data when I need results in meters or feet.
- The core spatial reasoning is the same in SQL and Python, even when the syntax is different.

## Workspace Maintenance

I completed my weekly workspace tidy-up:

- Organized local practice files.
- Planned the coming week of practice.
- Committed my current work to GitHub.

This is part of building a reproducible workflow, not just completing exercises. 
Clean files, clear project structure, and regular commits make it easier to inspect work, catch errors, and show progress over time.

## Portfolio Connection

This nearest-neighbor pattern fits directly into my Florida sinkhole and fire-station project.

A practical next analysis is to:
1. Load sinkhole incidents and fire-station locations.
2. Find the nearest fire station for each sinkhole.
3. Store the nearest-station distance.
4. Review sinkholes that are farthest from station coverage.
5. Publish the analysis, code, and a short methods note.

## Key Takeaway

Today reinforced a useful professional pattern:

```
Describe the spatial relationship first, then use the tool-specific implementation.
```

In SQL, that means CROSS JOIN LATERAL with ORDER BY <->.

In GeoPandas, that means sjoin_nearest().

The function names are different. The spatial problem is the same.




