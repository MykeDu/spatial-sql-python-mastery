# Day 56: Python Geometry-Health Diagnostic Pipeline

## Focus

Before repairing geometries or running nearest-park analysis, diagnose the geometry health of the dataset. The goal is a small, explicit pipeline that tells you what is missing, what is invalid, and what is ready for downstream work.

In Mike's Marion County workflow, the source had 3,473 total features:

- 1 missing geometry
- 16 invalid geometries
- 3,456 usable geometries

## What I Learned

Keep the `raw` or `prepared` GeoDataFrame intact. Create a copied diagnostic frame for inspection and labeling. That separation matters because diagnosis should describe the source data, not quietly mutate it.

The useful labels are explicit:

- `missing`: no geometry is present
- `invalid`: a geometry exists but fails the validity check
- `usable`: a geometry exists and is valid

The final check is a count, not a visual guess:

```python
diagnostic["geometry_health"].value_counts(dropna=False)
```

That gives the workflow a clear checkpoint before any repair decisions or nearest-park calculations.

## What I Practiced

I practiced writing a function that accepts a GeoDataFrame, copies it, assigns a geometry-health label to every feature, and returns both the diagnostic frame and its summary counts.

```python
import geopandas as gpd
import pandas as pd


def diagnose_geometry_health(
    gdf: gpd.GeoDataFrame,
) -> tuple[gpd.GeoDataFrame, pd.Series]:
    diagnostic = gdf.copy()

    diagnostic["geometry_health"] = "usable"

    missing = diagnostic.geometry.isna()
    invalid = diagnostic.geometry.notna() & ~diagnostic.geometry.is_valid

    diagnostic.loc[missing, "geometry_health"] = "missing"
    diagnostic.loc[invalid, "geometry_health"] = "invalid"

    summary = diagnostic["geometry_health"].value_counts(dropna=False)

    return diagnostic, summary


diagnostic, summary = diagnose_geometry_health(prepared_parcels)

print(summary)
```

For Marion County, the expected summary is:

```text
usable     3456
invalid      16
missing       1
Name: geometry_health, dtype: int64
```

## The Pattern

**Preserve, copy, label, summarize, then decide.**

1. Preserve the raw and prepared GeoDataFrames.
2. Copy the prepared data into a diagnostic frame.
3. Label each feature as `missing`, `invalid`, or `usable`.
4. Summarize with `value_counts(dropna=False)`.
5. Keep repair logic in a separate step.
6. Run downstream nearest-park analysis only after the data-quality decision is explicit.

This prevents a common mistake: fixing or dropping rows before you know exactly what the source contained.

## AI Pairing Prompt

> Review this GeoPandas geometry-health diagnostic function. Check whether the labels `missing`, `invalid`, and `usable` are mutually exclusive and collectively complete. Confirm that the function preserves the input GeoDataFrame, handles null geometries safely, and returns counts that reconcile to the total feature count. Do not add repair logic or nearest-park analysis. Suggest only focused improvements to the diagnosis step.

## Why This Matters

Spatial analysis becomes easier to trust when data quality is observable. A nearest-park result built on silently missing or invalid geometries can look precise while excluding the very features you meant to analyze.

This diagnostic pipeline creates an audit point. You can report the source condition, choose a repair policy deliberately, and verify that later analysis uses the intended rows.

## Next Steps

- Add assertions that the summary counts total 3,473 features.
- Inspect the one missing and 16 invalid geometries separately.
- Write a separate repair function with an explicit policy.
- Re-run the health summary after repair.
- Only then connect the cleaned data to nearest-park analysis.