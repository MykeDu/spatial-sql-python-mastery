# Day 50: Trace the Usable-Geometry Gate

## Focus

Trace how a GeoPandas geometry-health decision moves from boolean masks to a named analysis subset.

The date for this practice session was September 25, 2026. The goal was to make the Marion County workflow easy to inspect by preserving the raw prepared GeoDataFrame and using aligned masks, `.loc`, and `.copy()` to create a separate usable layer.

## What I Learned

Boolean masks are row-aligned conditions. When a mask is built from a GeoDataFrame, its index identifies which rows satisfy the condition.

For the geometry-health workflow:

- `missing` identifies rows where geometry is absent.
- `present` identifies rows that have geometry.
- `invalid` checks validity only for present geometries.
- `usable` identifies rows that are both present and valid.

The masks can be combined safely because they use the same index as the raw GeoDataFrame. `.loc[usable]` selects the rows whose mask value is `True`. Adding `.copy()` creates an independent GeoDataFrame for analysis instead of a view into the source.

The raw prepared layer remains available for audit and comparison. The usable subset becomes the explicit input for later representative-point and nearest-park operations.

## What I Practiced

I traced the Marion County geometry gate from the source layer to the analysis layer.

```python
def trace_usable_geometry_gate(subdivisions_prepared):
    # Preserve the prepared source for inspection.
    raw = subdivisions_prepared.copy()

    # Build masks with the same index as raw.
    missing = raw.geometry.isna()
    present = ~missing

    # Validate only geometries that are present.
    invalid = present & ~raw.geometry.is_valid
    usable = present & ~invalid

    # Select rows by the aligned boolean mask.
    subdivisions_analysis = raw.loc[usable].copy()

    report = {
        "total": len(raw),
        "missing": int(missing.sum()),
        "invalid": int(invalid.sum()),
        "usable": int(usable.sum()),
        "analysis_rows": len(subdivisions_analysis),
    }

    return raw, subdivisions_analysis, report


subdivisions_raw, subdivisions_analysis, geometry_report = (
    trace_usable_geometry_gate(subdivisions_prepared)
)

print(geometry_report)
```

The expected Marion County source-quality counts were:

- 3,473 total subdivision features
- 1 missing geometry
- 16 invalid geometries
- 3,456 usable features

The raw and analysis layers now have different roles:

```python
# Audit the original prepared layer.
subdivisions_raw.geometry.isna().sum()

# Run later spatial work only on the gated subset.
subdivisions_analysis
```

## The Pattern

```text
Copy the prepared source.
Build masks from the source index.
Separate missing geometry from invalid geometry.
Combine conditions into a usable mask.
Select with .loc.
Copy the selected rows.
Keep the raw layer unchanged.
Pass only the named analysis subset forward.
```

In plain English:

> Make the inclusion rule visible, use the aligned mask to select rows, and copy the result before spatial analysis.

For the Marion County workflow, `subdivisions_analysis` is the controlled input for later representative-point creation and nearest-park analysis in EPSG:2236. The geometry gate does not repair or reproject data. It only defines which existing features are eligible for the next stage.

## AI Pairing Prompt

```text
Review this GeoPandas geometry gate for a Marion County nearest-park workflow.

Check whether the boolean masks remain aligned with the raw GeoDataFrame, whether .loc is used clearly for row selection, and whether .copy() preserves a separate analysis subset.

The raw layer must remain unchanged. The usable subset should include only present and valid geometries. Do not add automatic repair or reprojecting. Explain any concern before suggesting the smallest possible revision.

missing = raw.geometry.isna()
present = ~missing
invalid = present & ~raw.geometry.is_valid
usable = present & ~invalid

subdivisions_analysis = raw.loc[usable].copy()
```

## Why This Matters

A geometry gate is more trustworthy when each step can be traced.

The workflow shows:

- Which GeoDataFrame is the source
- Which index-aligned conditions were evaluated
- How missing and invalid geometry were treated
- Which rows entered the analysis subset
- That the prepared source was not silently modified
- Why later spatial operations use `subdivisions_analysis`

This separation makes debugging easier. If the usable count changes, the report and masks provide a direct path back to the source-quality decision.

## Next Steps

Use `subdivisions_analysis` for the next spatial stage.

1. Create representative points without changing the source polygons.
2. Confirm the point layer uses EPSG:2236.
3. Run the nearest-park workflow on the gated features.
4. Record the usable count alongside later spatial results.
5. Keep geometry repair and other transformations as separate, documented decisions.