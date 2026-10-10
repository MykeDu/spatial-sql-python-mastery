## Focus

Validate the nearest-park result in GeoPandas and make the matching rule inspectable.

The lesson used subdivision representative points and park geometries in EPSG:2236, then applied `gpd.sjoin_nearest()` to reproduce the nearest-park relationship in Python. The validation focused on distance units, missing matches, duplicate subdivision matches caused by tied nearest parks, and a deterministic rule for keeping one result per subdivision before ranking the farthest ten.

## What I Learned

A spatial join can be technically correct and still need an audit step before its output is trusted.

- EPSG:2236 keeps the distance calculation in a projected coordinate system appropriate for local measurements.
- `representative_point()` creates one point inside each subdivision for a consistent point-to-park comparison.
- `gpd.sjoin_nearest()` adds the nearest park relationship and a distance column that can be named `distance_ft`.
- With EPSG:2236, `distance_ft` is measured in US survey feet, not degrees.
- Missing matches and duplicate `subdivision_id` values are useful signals to inspect rather than silently discard.
- Duplicate rows can occur when two parks are tied for the nearest distance, so "nearest" does not always mean "one row."

## What I Practiced

I validated the nearest-park join with a staged GeoPandas workflow:

1. Reprojected the subdivision and park layers to EPSG:2236.
2. Created one `access_point` per subdivision with `representative_point()`.
3. Ran `gpd.sjoin_nearest()` from subdivision access points to parks and stored the result in a table with `distance_ft`.
4. Inspected rows with missing park matches.
5. Grouped by `subdivision_id` to find duplicate matches created by tied nearest parks.
6. Applied a deterministic tie rule by sorting on `distance_ft` and then `park_id`.
7. Dropped duplicate `subdivision_id` values after the sort so the smallest park ID wins an exact distance tie.
8. Sorted the cleaned result by `distance_ft` and reviewed the farthest ten subdivisions.

I also refactored the project into named stages while preserving the existing Leaflet proof map. The analysis stays in a projected CRS, and EPSG:4326 is used only at the browser-export boundary where Leaflet expects longitude and latitude.

## The Pattern

```python
subdivision_points = subdivisions.to_crs(2236).copy()
parks_2236 = parks.to_crs(2236).copy()

subdivision_points["access_point"] = subdivision_points.geometry.representative_point()

joined = gpd.sjoin_nearest(
    subdivision_points.set_geometry("access_point"),
    parks_2236[["park_id", "geometry"]],
    how="left",
    distance_col="distance_ft",
)

missing_matches = joined[joined["park_id"].isna()]

duplicate_matches = (
    joined.groupby("subdivision_id")
    .size()
    .loc[lambda counts: counts > 1]
)

cleaned = (
    joined
    .sort_values(["subdivision_id", "distance_ft", "park_id"])
    .drop_duplicates("subdivision_id", keep="first")
)

farthest_ten = cleaned.sort_values("distance_ft", ascending=False).head(10)
```

The important pattern is to treat the nearest-neighbor result as an auditable intermediate table. First measure and inspect it. Then make the tie rule explicit. Only after one row remains per subdivision should the result be ranked or exported.

## AI Pairing Prompt

Review this GeoPandas nearest-park validation workflow. Confirm that EPSG:2236 is preserved for the distance calculation, that `distance_ft` is interpreted as US survey feet, and that missing matches and tied nearest parks are audited before deduplication. Check whether sorting by `subdivision_id`, `distance_ft`, and `park_id` creates a deterministic one-row-per-subdivision result. Also identify any places where the Leaflet proof map might accidentally receive projected coordinates instead of EPSG:4326 browser coordinates.

## Why This Matters

A ranking is only as trustworthy as the thing it measures.

A validation pass catches assumptions that can hide inside a convenient spatial join. It shows whether every subdivision found a park, whether ties created multiple candidate rows, and whether the final ten are based on a stable one-row-per-subdivision table. The explicit tie rule makes reruns reproducible instead of dependent on the source row order.

The refactor matters for the same reason. Named stages make it possible to inspect whether a problem came from CRS handling, representative points, nearest-neighbor selection, tie resolution, ranking, connection lines, or browser export. Keeping EPSG:4326 limited to the final Leaflet export also protects the distance calculation from accidental use of geographic degrees.

## Next Steps

- Compare the GeoPandas results with the previous SQL nearest-park ranking.
- Record the count of missing matches and tied matches as validation checks.
- Keep the deterministic `park_id` tie rule documented beside the ranking code.
- Preserve the named stages as the project grows.
- Add a small regression check that asserts one row per `subdivision_id` before the farthest-ten ranking.
- Continue using EPSG:4326 only when preparing data for the Leaflet browser map.