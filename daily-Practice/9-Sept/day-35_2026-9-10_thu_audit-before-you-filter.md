# Day 35: Audit Before You Filter

## Focus

Audit the classification field before filtering, then carry only confirmed residential records into the spatial analysis.

Today's project work is the next step in the Marion County park-access workflow:

- Inspect the `1`, `0`, and `NULL` classification values.
- Validate the non-null values before trusting the field.
- Filter confirmed residential records with `.eq(1)`.
- Preserve the source GeoDataFrame with `.copy()`.
- Reproject the working subset and parks to `EPSG:2236`.
- Run `sjoin_nearest`.
- Rank the ten farthest links.
- Export browser copies only after the analysis, in `EPSG:4326`.

## What I Learned

A filter is only as defensible as the field behind it. Before filtering, I need to know what values actually exist, whether the non-null values match the expected rule, and how many records remain unresolved.

The auditable rule is a three-state classification:

- `1` means confirmed residential.
- `0` means confirmed non-residential.
- `NULL` means unresolved and still needs review.

`value_counts(dropna=False)` is important because the default can hide missing values. If I filter first, I can accidentally lose the review queue before I have measured it.

The name-field text-pattern pass is useful as a temporary weak classification input, but it is not the final evidence. The auditable three-state rule is the source of truth for this workflow.

## What I Practiced

I audited the classification values before creating the working subset:

```python
classification_counts = (
    gdf["is_residential"]
    .value_counts(dropna=False)
    .rename_axis("is_residential")
    .rename("record_count")
)

print(classification_counts)

non_null_values = set(gdf["is_residential"].dropna().unique())
allowed_values = {0, 1}

unexpected_values = non_null_values - allowed_values
if unexpected_values:
    raise ValueError(
        f"Unexpected non-null classification values: {unexpected_values}"
    )
```

Then I filtered only confirmed residential records and preserved the source object:

```python
residential = gdf.loc[
    gdf["is_residential"].eq(1)
].copy()

if residential.empty:
    raise ValueError("No confirmed residential records were found.")
```

The `.eq(1)` expression makes the business rule explicit. The `.copy()` matters because the original `gdf` remains available for auditing, comparison, and later corrections.

For the spatial join, I used a projected CRS so nearest-neighbor distances are measured in feet:

```python
residential_2236 = residential.to_crs("EPSG:2236")
parks_2236 = parks.to_crs("EPSG:2236")

nearest_parks = gpd.sjoin_nearest(
    residential_2236,
    parks_2236[["park_id", "park_name", "geometry"]],
    how="left",
    distance_col="distance_ft",
)

farthest_ten = (
    nearest_parks
    .sort_values("distance_ft", ascending=False)
    .head(10)
    .copy()
)

print(
    farthest_ten[
        ["subdivision_id", "name", "park_id", "park_name", "distance_ft"]
    ]
)
```

I kept the analysis in `EPSG:2236`, then created browser-ready copies only after the distance work was complete:

```python
farthest_ten_browser = farthest_ten.to_crs("EPSG:4326")
parks_browser = parks_2236.to_crs("EPSG:4326")

farthest_ten_browser.to_file(
    "farthest-ten-residential-links.geojson",
    driver="GeoJSON",
)

parks_browser.to_file(
    "parks-browser.geojson",
    driver="GeoJSON",
)
```

The projected working data is for analysis. The geographic copies are for browser mapping.

## The Pattern

```text
Audit the classification field
|
v
Confirm the allowed non-null values
|
v
Count 1, 0, and NULL with value_counts(dropna=False)
|
v
Filter with .eq(1)
|
v
Preserve the source with .copy()
|
v
Reproject working data and parks to EPSG:2236
|
v
Run sjoin_nearest and rank distance
|
v
Export browser copies in EPSG:4326
```

The sequence is the point. Classification comes before filtering, distance analysis uses a projected CRS, and browser export happens last.

## AI Pairing Prompt

> Review this GeoPandas workflow as an auditor, not just as a coder. Confirm that the classification field contains only the expected non-null values `0` and `1`, that `NULL` records remain visible in the audit, that the source GeoDataFrame is preserved, and that all nearest-distance calculations happen in `EPSG:2236`. Check the `sjoin_nearest` result for missing matches, verify that the ten farthest links are ranked by `distance_ft`, and confirm that only the final browser copies are exported in `EPSG:4326`. Identify any assumption that should be documented before I publish the result.

## Why This Matters

A clean-looking map can still be built on an unverified filter. Auditing the classification first makes the inclusion rule visible and gives unresolved records a place to go instead of silently dropping them.

The CRS boundary is just as important. `EPSG:2236` keeps the distance calculation in US survey feet, while `EPSG:4326` is the practical format for browser mapping. Keeping those roles separate makes the result easier to explain and harder to misuse.

This is the difference between producing a result and producing a result that another person can inspect.

## Next Steps

- Inspect the full `value_counts(dropna=False)` output and record the counts.
- Review the `NULL` records instead of treating them as non-residential.
- Check the farthest ten links for missing park matches and suspicious geometries.
- Add a short data dictionary entry for the three-state residential rule.
- Publish the browser copies only after the projected analysis passes review.