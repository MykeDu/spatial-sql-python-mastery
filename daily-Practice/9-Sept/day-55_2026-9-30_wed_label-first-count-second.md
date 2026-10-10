# Day 55: Label First, Count Second

## Focus

Turn a geometry-health audit into a visible classification before summarizing it. The goal is to keep the raw GeoDataFrame unchanged, create a copied analysis subset, assign explicit geometry-health labels, and only then count those labels.

## What I Learned

A count is easier to trust when the categories behind it are visible. Instead of jumping directly to separate boolean counts, create one clear label field such as `missing`, `invalid`, or `usable` on a copied GeoDataFrame. The labeled rows become evidence that can be inspected, filtered, mapped, or used in later analysis.

This preserves the read-only nature of the audit. The source data remains the source of truth, while the copy provides an explicit analysis record. In the known Marion County results, the source contains 3,473 features, including 1 missing geometry and 16 invalid geometries. Those established results can be represented as labels without changing the source.

## What I Practiced

I practiced building the geometry-health label with an ordered decision rule:

1. Identify missing geometry first.
2. Check validity only for geometry that is present.
3. Label the remaining features as usable.
4. Summarize the labels with `value_counts(dropna=False)`.

This separates classification from summarization. If the counts look unexpected, the labeled rows can be inspected before any downstream spatial operation is attempted.

## The Pattern

```python
import geopandas as gpd

# Keep the raw GeoDataFrame unchanged.
raw_gdf = parcels

# Create a copied analysis subset for geometry health.
health_gdf = raw_gdf.copy()

missing = health_gdf.geometry.isna()
present = ~missing
invalid = present & ~health_gdf.geometry.is_valid

health_gdf["geometry_health"] = "usable"
health_gdf.loc[missing, "geometry_health"] = "missing"
health_gdf.loc[invalid, "geometry_health"] = "invalid"

health_counts = health_gdf["geometry_health"].value_counts(dropna=False)

print(health_counts)

# Use only usable features for geometry-dependent analysis.
usable_gdf = health_gdf.loc[
    health_gdf["geometry_health"] == "usable"
].copy()
```

The important sequence is label first, count second. The label column makes the classification inspectable and reusable. The count is a summary of that evidence, not a replacement for it.

## AI Pairing Prompt

Review this GeoPandas geometry-health workflow for correctness and clarity. Keep the raw GeoDataFrame unchanged. On a copied analysis GeoDataFrame, classify each feature as `missing`, `invalid`, or `usable`, with missing geometry checked before validity. Then summarize the labels with `value_counts(dropna=False)`. Explain how to inspect the labeled rows and confirm that only usable geometries enter downstream spatial analysis. Do not invent measurements or repair geometries unless explicitly requested.

## Why This Matters

Explicit labels make data quality decisions reproducible. They also make later debugging easier because a feature can be traced from its original row to its health category and then to the downstream filter that included or excluded it. This supports a safer GeoPandas and PostGIS workflow where the raw source is preserved, categories are visible, and analysis is based on documented evidence.

## Next Steps

- Add a small inspection view for each health label.
- Compare the labeled GeoDataFrame with the known Marion County results: 3,473 total features, 1 missing geometry, and 16 invalid geometries.
- Keep geometry repair as a separate, explicit step rather than mixing it into the audit.
- Reuse the label field when preparing safe inputs for spatial joins, overlays, or exports.