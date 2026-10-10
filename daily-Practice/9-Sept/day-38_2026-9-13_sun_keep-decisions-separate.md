# Day 38: The Audit Boundary Before Spatial Analysis

## Focus

Attach a documented residential-review table to the full `subdivisions_gdf` with a validated left merge, then create a separate confirmed-residential subset.

This is the audit boundary: first preserve and inspect the complete source layer; only then make a downstream analysis subset. Unreviewed subdivisions must remain visible, and missing review statuses must stay missing.

## What I Learned

A left merge keeps the source GeoDataFrame as the governing table:

- Every source subdivision remains in the result.
- Source geometry is preserved.
- Unreviewed subdivisions remain present with a missing `is_residential` value.
- `validate="one_to_one"` catches unexpected duplicate keys instead of silently multiplying rows.
- A separate `residential_gdf` makes the analytical filter explicit and reversible.

The review table is evidence, not a replacement for the source layer. I should preserve the raw source, validate the review columns and keys before joining, and audit all classification states after the merge. In particular, I must not fill missing statuses with `1`: unresolved is not the same as confirmed residential.

## What I Practiced

I validated the review table before attaching it:

```python
import pandas as pd
import geopandas as gpd

required_columns = {
    "subdivision_id",
    "is_residential",
    "source",
    "rationale",
}
missing_columns = required_columns - set(review_df.columns)

if missing_columns:
    raise ValueError(
        f"Missing required review columns: {sorted(missing_columns)}"
    )

duplicate_ids = review_df.loc[
    review_df["subdivision_id"].duplicated(keep=False),
    "subdivision_id",
].unique()

if len(duplicate_ids):
    raise ValueError(
        f"Duplicate subdivision_id values: {duplicate_ids.tolist()}"
    )

allowed_values = {0, 1}
non_missing_values = set(
    review_df["is_residential"].dropna().unique()
)
unexpected_values = non_missing_values - allowed_values

if unexpected_values:
    raise ValueError(
        f"Unexpected is_residential values: {unexpected_values}"
    )
```

I kept the raw source untouched, attached the review decisions with a left merge, and restored the GeoDataFrame metadata explicitly:

```python
source_crs = subdivisions_gdf.crs
source_row_count = len(subdivisions_gdf)

attached = subdivisions_gdf.merge(
    review_df,
    on="subdivision_id",
    how="left",
    validate="one_to_one"
)
attached = gpd.GeoDataFrame(
    attached,
    geometry="geometry",
    crs=subdivisions_gdf.crs
)

if len(attached) != source_row_count:
    raise ValueError("The merge changed the source row count.")

if attached.crs != source_crs:
    raise ValueError("The merge changed the source CRS.")
```

Then I audited the full set, including missing values, before creating the confirmed-residential subset:

```python
print(attached["is_residential"].value_counts(dropna=False))

residential_gdf = attached.loc[
    attached["is_residential"].eq(1)
].copy()
```

The audit should also make the merge coverage visible:

```python
reviewed_count = attached["is_residential"].notna().sum()
unreviewed_count = attached["is_residential"].isna().sum()

print(f"Source rows: {len(subdivisions_gdf)}")
print(f"Reviewed rows: {reviewed_count}")
print(f"Unreviewed rows: {unreviewed_count}")
print(f"Confirmed residential rows: {len(residential_gdf)}")
```

## The Pattern

```text
raw source GeoDataFrame
    -> validate review table
    -> left merge by subdivision_id
    -> audit 0, 1, and missing values
    -> create confirmed-residential subset
    -> reproject only before later distance analysis
```

The merge is the boundary between source evidence and downstream interpretation. Keep `subdivisions_gdf` as the raw source, keep `attached` as the auditable joined layer, and use `residential_gdf` only for confirmed-residential analysis.

## AI Pairing Prompt

> Review this GeoPandas workflow as an audit-sensitive data pipeline. Check that the review table has the required columns, reject duplicate `subdivision_id` values, preserve the raw source and source geometry, use a left merge with `validate="one_to_one"`, and report `0`, `1`, and missing `is_residential` values separately. Do not fill missing statuses with `1`. Explain any row-count, CRS, or join-coverage risk before suggesting changes.

## Why This Matters

Spatial analysis is only as defensible as the boundary between evidence and filtering. If a merge drops unreviewed rows, duplicates features, loses geometry, or converts missing decisions into positive classifications, the map can look precise while answering the wrong question.

This pattern makes the decision trail inspectable. It also keeps the workflow flexible: I can review more subdivisions later and rebuild the attached layer without rewriting the source data or hiding unresolved records.

For later distance analysis, reproject the analysis copy to `EPSG:2236` only at the point where projected measurements are needed. Do not change the source CRS merely to perform the review attachment.

## Next Steps

- Inspect the `value_counts(dropna=False)` output and record the reviewed, unreviewed, residential, and non-residential counts.
- Review the missing-status queue before trusting any residential-only result.
- Visualize `attached` with review status visible so unresolved subdivisions remain on the map.
- Use `residential_gdf` for confirmed-residential analysis only.
- Reproject a downstream analysis copy to `EPSG:2236` immediately before distance calculations.