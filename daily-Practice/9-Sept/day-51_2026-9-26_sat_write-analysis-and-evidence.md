# Day 51: Write Analysis and Evidence

## Focus

Use a usable-geometry gate in GeoPandas so the analysis is explicit, traceable, and safe.

The workflow should:

1. Preserve the raw GeoDataFrame unchanged.
2. Identify missing and invalid geometry.
3. Create an analysis subset containing only usable geometry.
4. Return a summary report that documents what happened.

The analysis subset answers: "Which features can continue?"
The summary report answers: "What happened to the full source layer?"

## What I Learned

A GeoDataFrame can contain rows with missing or invalid geometry. Those rows should not silently flow into spatial operations.

A boolean mask makes the gate visible:

- Missing geometry: `gdf.geometry.isna()`
- Valid geometry: `gdf.geometry.is_valid`
- Usable geometry: geometry is present and valid

The raw GeoDataFrame should remain available for auditing and reporting. The filtered subset is a separate object used for downstream analysis.

Using `.loc[mask].copy()` is important because it makes the selected analysis data explicit and avoids accidental edits to a view of the source data.

## What I Practiced

I practiced writing a preparation function that:

- Accepts a raw GeoDataFrame.
- Preserves the source object.
- Counts total, missing, invalid, and usable geometries.
- Returns a copied analysis subset.
- Returns a summary report with evidence about the gate.

```python
import geopandas as gpd


def prepare_usable_subdivisions(
    subdivisions: gpd.GeoDataFrame,
) -> tuple[gpd.GeoDataFrame, dict[str, int]]:
    """Return usable geometry and an evidence summary.

    The input GeoDataFrame is not modified.
    """
    total_count = len(subdivisions)

    missing_mask = subdivisions.geometry.isna()
    valid_mask = subdivisions.geometry.notna() & subdivisions.geometry.is_valid
    invalid_mask = subdivisions.geometry.notna() & ~subdivisions.geometry.is_valid

    usable_mask = valid_mask

    usable_subdivisions = subdivisions.loc[usable_mask].copy()

    summary = {
        "total_count": total_count,
        "missing_geometry_count": int(missing_mask.sum()),
        "invalid_geometry_count": int(invalid_mask.sum()),
        "usable_geometry_count": int(usable_mask.sum()),
    }

    return usable_subdivisions, summary
```

Example use:

```python
usable_subdivisions, report = prepare_usable_subdivisions(
    raw_subdivisions
)

print(report)
```

The report should make the relationship between the counts easy to check:

```text
total_count = missing_geometry_count
             + invalid_geometry_count
             + usable_geometry_count
```

This assumes that geometry is either missing, invalid, or usable. If the project defines additional geometry states, the report should document those categories too.

## The Pattern

The reusable pattern is:

```python
raw_gdf = load_source_layer()

missing_mask = raw_gdf.geometry.isna()
valid_mask = raw_gdf.geometry.notna() & raw_gdf.geometry.is_valid

analysis_gdf = raw_gdf.loc[valid_mask].copy()

summary = {
    "total_count": len(raw_gdf),
    "missing_geometry_count": int(missing_mask.sum()),
    "invalid_geometry_count": int(
        (raw_gdf.geometry.notna() & ~raw_gdf.geometry.is_valid).sum()
    ),
    "usable_geometry_count": int(valid_mask.sum()),
}
```

Keep the raw layer for evidence. Use the copied subset for analysis. Return both the usable data and the report so later code does not have to reconstruct what the gate did.

## AI Pairing Prompt

> Review this GeoPandas preparation function as a data-quality gate. Verify that it preserves the raw GeoDataFrame, creates an independent analysis subset with `.loc[mask].copy()`, and correctly counts missing, invalid, and usable geometry. Check whether the categories are mutually exclusive and collectively complete. Suggest tests for an empty layer, all usable geometry, all missing geometry, invalid geometry, and mixed input. Use plain ASCII punctuation in the explanation and code comments.

## Why This Matters

A filtered result without evidence is difficult to audit. A report without a clearly defined analysis subset is difficult to use.

Returning both objects creates a clean boundary:

- The raw GeoDataFrame supports traceability.
- The analysis subset supports spatial computation.
- The summary report supports communication and testing.

This makes the geometry gate observable instead of hidden inside a later failure.

## Next Steps

- Add tests for missing, invalid, and usable geometry.
- Confirm that the raw GeoDataFrame is unchanged after preparation.
- Add an assertion that the reported counts reconcile with the total.
- Decide whether invalid geometries should be repaired, excluded, or reported for manual review.
- Carry the summary report into the next analysis or export step.