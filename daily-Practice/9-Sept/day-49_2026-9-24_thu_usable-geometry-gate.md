# Day 49: Create an Explicit Usable-Geometry Gate

## Focus

Create a clear GeoPandas analysis gate that separates source inspection from spatial computation.

The goal was to preserve the prepared subdivision layer, document its geometry health, and create a separate working subset containing only features with present and valid geometry.

## What I Learned

A geometry audit and an analysis subset serve different purposes.

The audit records what arrived in the source data:

- 3,473 total subdivision features
- 1 missing geometry
- 16 invalid geometries

The analysis subset defines what is allowed into later spatial operations.

The raw prepared layer remains unchanged. A separate copied layer, `subdivisions_analysis`, contains only features that pass the inclusion rule.

Missing and invalid geometry are also separate conditions. A missing geometry has no shape to validate, so validity should only be evaluated for rows where geometry is present.

## What I Practiced

I wrote a reusable function that:

1. Preserves the prepared input layer.
2. Identifies missing geometry.
3. Identifies present geometry.
4. Identifies invalid geometry only among present features.
5. Creates a report with total, missing, invalid, and usable counts.
6. Returns a copied analysis subset and the report.

```python
def prepare_usable_subdivisions(subdivisions_prepared):
    raw = subdivisions_prepared.copy()

    missing = raw.geometry.isna()
    present = ~missing
    invalid = present & ~raw.geometry.is_valid
    usable = present & ~invalid

    report = {
        "total": len(raw),
        "missing": int(missing.sum()),
        "invalid": int(invalid.sum()),
        "usable": int(usable.sum()),
    }

    analysis = raw.loc[usable].copy()
    return analysis, report
```

The function returns two values:

- `analysis`, the copied present-and-valid GeoDataFrame
- `report`, the evidence needed to document the source-quality results

I also practiced writing an AI review prompt that named the observed counts, preserved the raw source, stated the inclusion rule, and kept automatic repair out of scope.

## The Pattern

```text
Audit the source.
Create explicit boolean masks.
Select present and valid features.
Copy the analysis subset.
Return the subset and the evidence report.
Run spatial analysis only after the gate.
```

In plain English:

> Keep the original prepared data available for inspection, identify the records that are safe for analysis, and pass only that named subset to the next spatial stage.

For the Marion County workflow, later representative-point and nearest-park calculations should use the usable subset in EPSG:2236. Geometry repair, reprojection, representative-point creation, and nearest-neighbor analysis remain separate stages.

## AI Pairing Prompt

```text
Review my GeoPandas inclusion rule for a Marion County nearest-park workflow.

The raw prepared subdivision layer has 3,473 features, including 1 missing geometry and 16 invalid geometries. I am preserving that layer unchanged. A separate copied analysis layer includes only geometries that are present and valid.

Review whether this rule and its documentation are clear for later representative-point and sjoin_nearest work in EPSG:2236. Do not propose automatic repair. Explain any concern before suggesting a minimal code change.

def prepare_usable_subdivisions(subdivisions_prepared):
    raw = subdivisions_prepared.copy()

    missing = raw.geometry.isna()
    present = ~missing
    invalid = present & ~raw.geometry.is_valid
    usable = present & ~invalid

    report = {
        "total": len(raw),
        "missing": int(missing.sum()),
        "invalid": int(invalid.sum()),
        "usable": int(usable.sum()),
    }

    analysis = raw.loc[usable].copy()
    return analysis, report
```

## Why This Matters

Spatial analysis is easier to trust when the inclusion rule is visible.

A reviewer can see:

- What entered the workflow
- What was excluded
- Why it was excluded
- Which layer was used for later calculations
- That the raw source was not silently changed

This prevents geometry validation, repair, coordinate transformation, and nearest-park analysis from becoming one opaque operation.

The usable-geometry gate creates a clear boundary between evidence and computation.

## Next Steps

- Keep the raw prepared subdivision layer unchanged.
- Retain the read-only geometry-health report.
- Pass `subdivisions_analysis` into the representative-point stage.
- Continue using EPSG:2236 for measurement-based spatial analysis.
- Keep geometry repair as a separate, explicitly documented stage.
- Wire the gate into the existing `mc-park-access` workflow before resuming nearest-park and map work.
- Record the observed usable count and inclusion rule in the next GitHub field note.