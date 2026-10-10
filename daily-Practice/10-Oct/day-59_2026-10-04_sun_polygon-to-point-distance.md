# Day 59: Polygon-to-point distance

## Focus

Calculate the distance from one subdivision polygon to every park point in the Marion County park-access workflow.

The key setup is simple and deliberate:

1. Isolate one subdivision polygon.
2. Keep the polygon and park points in the projected measurement CRS, EPSG:2236.
3. Use GeoPandas vectorized distance to measure from the polygon to every park point.
4. Sort the results and return the nearest park row.

This is a focused distance exercise before moving into the broader nearest-neighbor workflow.

## What I Learned

A polygon-to-point distance is the shortest distance from the polygon geometry to each park point geometry. GeoPandas calculates that distance across the full geometry column, so there is no need to write a manual row-by-row loop.

The distance units come from the projected CRS. In this workflow, EPSG:2236 provides US survey-foot measurements, so the resulting distance values are in US survey feet.

That is different from the later representative-point workflow. There, a subdivision is reduced to a representative point, and the representative point is compared with the nearest park. Here, the full polygon is measured directly to each park point. Those are related questions, but they are not the same measurement.

## What I Practiced

I practiced making the measurement CRS explicit and keeping the geometry preparation steps separate from the distance calculation.

```python
import geopandas as gpd

subdivision = subdivisions.loc[
    subdivisions["subdivision_name"].eq("Example Subdivision")
].iloc[[0]]

parks_2236 = parks.to_crs("EPSG:2236")
subdivision_2236 = subdivision.to_crs("EPSG:2236")

distances = parks_2236.assign(
    distance_ft=parks_2236.geometry.distance(
        subdivision_2236.geometry.iloc[0]
    )
)

nearest_park = (
    distances.sort_values("distance_ft")
    .iloc[0]
)

nearest_park
```

The important part is `parks_2236.geometry.distance(...)`. GeoPandas applies the distance calculation to the entire geometry series. Sorting by `distance_ft` then puts the closest park first, and `.iloc[0]` returns the nearest park row.

## The Pattern

Prepare geometry first. Measure in a projected CRS. Use a vectorized geometry operation. Sort the results. Return the nearest record.

This keeps three responsibilities visible:

- Geometry preparation and usable-geometry checks happen before measurement.
- The measurement CRS is explicit, rather than assumed.
- The nearest-neighbor method is chosen deliberately instead of hidden inside a manual loop.

That explicitness is becoming a useful habit in the Marion County workflow. It makes it easier to tell whether a problem comes from geometry quality, CRS handling, or the nearest-feature method itself.

## AI Pairing Prompt

Review this polygon-to-point distance code line by line before suggesting any changes.

Check that:

- The subdivision selection returns exactly one polygon.
- Both layers use EPSG:2236 before distance is calculated.
- The code does not confuse setting a CRS with transforming coordinates.
- The distance values are interpreted as US survey feet.
- The calculation stays vectorized and does not introduce `iterrows()`, `apply()`, or another manual row loop.
- The result is clearly distinguished from representative-point-to-nearest-park distance.
- The later workflow should use `sjoin_nearest()` where it is the better fit.

Do not rewrite the code immediately. First explain what each line is doing, identify any CRS or geometry risks, and then propose the smallest safe change.

## Why This Matters

Distance analysis is only trustworthy when the geometry and CRS choices are visible. Calculating distance in EPSG:4326 can produce values in degrees, which are not suitable for interpreting park access in feet. Setting a CRS label is also not the same as transforming coordinates, so the data must be transformed when it is not already in the measurement CRS.

This exercise also reinforces the difference between a direct polygon-to-point measurement and a representative-point approximation. Knowing which question the geometry represents is just as important as knowing which function to call.

## Next Steps

Use this direct distance calculation as a bridge to the full park-access workflow:

1. Keep the geometry-health and usable-geometry gates in place.
2. Build representative points for subdivisions when that is the intended access measure.
3. Use `sjoin_nearest()` to connect representative points to their nearest parks.
4. Compare the resulting access distances and park attributes in a clear, repeatable output.

The goal is not just to find a nearby park. It is to make the geometry preparation, measurement CRS, and nearest-neighbor method explicit enough that the result can be checked and trusted.