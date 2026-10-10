# Day 47: Report Geometry Health Without Mutating the Source

## Focus

Check geometry health before spatial analysis without changing the source GeoDataFrame. Today the boundary is simple: report missing and invalid geometry separately, then return a clean working subset only when the caller asks for one.

## What I Learned

Missing geometry and invalid geometry are different data-quality conditions.

- Missing geometry means the geometry value is null.
- Invalid geometry means a geometry is present but fails the geometry validity check.
- A missing geometry should not be counted as invalid.
- A health report should describe the input, not silently repair it, reproject it, or overwrite it.

The safest pattern is to preserve the input object, select the rows with geometry, and call `is_valid` only on that present subset. That keeps the counts explicit and makes the boundary behavior easy to test.

## What I Practiced

I practiced the Geometry-Health Boundary lesson and the Report Geometry Health exercise against the spatial inputs from the park-access workflow.

The review checked:

1. Total row count.
2. Missing geometry count.
3. Invalid geometry count among rows with geometry.
4. A copied subset containing only present and valid geometry.
5. No mutation, repair, or reprojection of the input.

The output is useful for an audit log and safe for a downstream step that requires usable geometry. It does not pretend that a bad geometry is fixed just because the pipeline continued.

```python
import geopandas as gpd


def report_geometry_health(
    layer: gpd.GeoDataFrame,
) -> tuple[dict[str, int], gpd.GeoDataFrame]:
    """Report geometry health and return a copied present-valid subset."""
    if "geometry" not in layer:
        raise ValueError("The GeoDataFrame must have a geometry column")

    present = ~layer.geometry.isna()
    present_geometry = layer.geometry.loc[present]
    invalid = ~present_geometry.is_valid

    report = {
        "rows": int(len(layer)),
        "missing_geometry": int((~present).sum()),
        "invalid_geometry": int(invalid.sum()),
        "present_and_valid": int((~invalid).sum()),
    }

    present_valid = present.copy()
    present_valid.loc[present] = ~invalid

    clean_subset = layer.loc[present_valid].copy()
    return report, clean_subset
```

The important details are that `layer` is never modified, `isna()` identifies missing geometry first, and `is_valid` is evaluated only for rows where geometry is present. The returned subset is a copy, so downstream work has a separate object to inspect or transform.

## The Pattern

Think of geometry health as a small, explicit contract:

```text
source GeoDataFrame
        |
        +-- missing geometry count
        |
        +-- present geometry
                |
                +-- invalid geometry count
                |
                +-- copied present-and-valid subset
```

Do not repair with `make_valid()`, drop rows in place, or call `to_crs()` inside this health check. Those are separate decisions with separate evidence. First report the source condition. Then let the next step decide whether to stop, review, repair, or continue.

## AI Pairing Prompt

> Review this GeoPandas geometry-health function as a data-quality boundary. Confirm that it preserves the input GeoDataFrame, counts missing and invalid geometries separately, evaluates `is_valid` only where geometry is present, and returns a copied present-and-valid subset. Do not add geometry repair, reprojection, or in-place mutation. Suggest focused tests for all-missing, all-valid, mixed, and invalid-input cases.

## Why This Matters

Spatial failures are often discovered too late, after a join, distance calculation, or map export has already mixed good and bad rows together. A small health report makes the condition visible at the boundary and keeps the decision traceable.

This also protects the source data. A diagnostic function should not change the evidence it is reporting. Preserving the input makes repeated checks predictable and keeps repair or reprojection as deliberate, reviewable steps.

## Next Steps

- Add tests for missing, valid, and invalid geometry.
- Save the health report with the input-stage audit output.
- Decide explicitly what the pipeline should do when invalid geometry is present.
- Keep repair and reprojection in separate, named steps.