# Day 57: Explicit Geometry-Health Labels

## Focus

Turn geometry diagnostics into an explicit, reviewable classification step. For the Marion County park-access workflow, the goal is to label every feature as `missing`, `invalid`, or `usable` without changing the source data.

## What I Learned

A diagnostic pipeline should make geometry health visible in the data itself. With 3,473 total features, the current facts are:

- 1 feature has missing geometry.
- 16 features have invalid geometry.
- 3,456 features are usable.

The labels are more useful than a set of temporary boolean checks because they create one clear status field for summaries, filtering, and later review.

## What I Practiced

I copied the source GeoDataFrame before adding diagnostic columns. Then I assigned explicit labels in a deliberate order: missing first, invalid second, and usable last. The summary uses `value_counts(dropna=False)` so every label is counted and unexpected null labels are not silently hidden.

```python
import geopandas as gpd

diagnostic = parks.copy()

is_missing = diagnostic.geometry.isna()
is_invalid = diagnostic.geometry.notna() & ~diagnostic.geometry.is_valid

diagnostic["geometry_health"] = "usable"
diagnostic.loc[is_invalid, "geometry_health"] = "invalid"
diagnostic.loc[is_missing, "geometry_health"] = "missing"

health_summary = diagnostic["geometry_health"].value_counts(
    dropna=False
)

print(health_summary)
```

Expected counts are `usable: 3456`, `invalid: 16`, and `missing: 1`, assuming the source facts have not changed.

## The Pattern

Preserve the raw GeoDataFrame. Create a copied diagnostic GeoDataFrame. Assign explicit labels that cover the full geometry-health state space. Summarize with `value_counts(dropna=False)`. Keep diagnosis separate from repair.

That last boundary matters. This step identifies and describes problems. It should not call `make_valid()`, drop records, fill missing geometries, or overwrite the source layer. Repair is a separate decision with its own checks and documentation.

## AI Pairing Prompt

> Review this GeoPandas geometry-health diagnostic pattern. Confirm that missing geometries are classified before invalid geometries, that the source GeoDataFrame remains unchanged, and that every feature receives exactly one label. Then suggest two assertions that would catch unexpected labels or a mismatch between the summary counts and the total feature count. Do not add repair logic.

## Why This Matters

Explicit labels make data quality operational. They support reproducible counts, targeted inspection, and clear handoffs before spatial joins or network analysis. They also reduce the risk of quietly treating missing geometry as invalid geometry, or of mixing diagnostic results with repairs that change the evidence.

## Next Steps

- Add assertions for the expected labels and total count.
- Inspect the 1 missing and 16 invalid features by ID.
- Save the diagnostic output separately from the raw layer.
- Decide on repair rules only after reviewing the flagged records.