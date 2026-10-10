## Focus

Start with a spatial data contract before running the nearest-park workflow.

The contract makes the expected schema, geometry, CRS, and working-layer behavior explicit. For this project, analysis stays in EPSG:2236, while EPSG:4326 is reserved for browser copies after analysis is complete.

## What I Learned

A nearest join is not the first step. The first step is proving that the inputs are the inputs the workflow expects.

- Inspect the actual columns in the subdivisions and parks layers.
- Identify stable IDs and readable names before writing the join.
- Verify geometry types and confirm that both layers have a CRS.
- Treat missing required fields as a clear schema error, not as a later join failure.
- Use copied working layers so validation and CRS preparation do not mutate the source data.
- Keep projected analysis in EPSG:2236 and transform to EPSG:4326 only for browser output.

## What I Practiced

I practiced a validation function with this contract:

```python
def validate_park_access_inputs(subdivisions_gdf, parks_gdf):
    required_subdivision_columns = ["subdivision_id", "name", "geometry"]
    required_park_columns = ["park_id", "name", "geometry"]

    missing_subdivision_columns = [
        column for column in required_subdivision_columns
        if column not in subdivisions_gdf.columns
    ]
    missing_park_columns = [
        column for column in required_park_columns
        if column not in parks_gdf.columns
    ]

    if missing_subdivision_columns:
        raise KeyError(
            f"Missing subdivision fields: {missing_subdivision_columns}. "
            f"Actual columns: {list(subdivisions_gdf.columns)}"
        )

    if missing_park_columns:
        raise KeyError(
            f"Missing park fields: {missing_park_columns}. "
            f"Actual columns: {list(parks_gdf.columns)}"
        )

    if subdivisions_gdf.crs is None:
        raise ValueError("Subdivisions must have a CRS before analysis.")
    if parks_gdf.crs is None:
        raise ValueError("Parks must have a CRS before analysis.")

    subdivisions_2236 = subdivisions_gdf.copy().to_crs(2236)
    parks_2236 = parks_gdf.copy().to_crs(2236)

    return subdivisions_2236, parks_2236
```

The important boundary is that this function validates and prepares the inputs only. It does not create representative points or run `sjoin_nearest`.

## The Pattern

```python
subdivisions_2236, parks_2236 = validate_park_access_inputs(
    subdivisions_gdf,
    parks_gdf,
)

# Only after the contract passes:
subdivisions_2236["access_point"] = (
    subdivisions_2236.geometry.representative_point()
)

joined = gpd.sjoin_nearest(
    subdivisions_2236.set_geometry("access_point"),
    parks_2236[["park_id", "name", "geometry"]],
    how="left",
    distance_col="distance_ft",
)
```

The pattern is validate first, transform copied working layers, then run spatial analysis. This keeps schema problems separate from geometry problems, CRS problems, and nearest-join problems.

## AI Pairing Prompt

Help me debug the spatial data contract before rewriting the workflow.

Here is the exact error:

```text
KeyError: Missing park fields: ['park_id', 'name']. Actual columns: ['OBJECTID', 'PARK_NAME', 'geometry']
```

Expected fields:

- Subdivisions: `subdivision_id`, `name`, `geometry`
- Parks: `park_id`, `name`, `geometry`

Constraints:

- Keep analysis in EPSG:2236.
- Reject missing CRS.
- Return copied working layers.
- Do not run `representative_point()` or `sjoin_nearest()` until validation passes.
- Use EPSG:4326 only for browser copies after analysis.

Diagnose the mismatch and suggest the smallest explicit mapping or rename needed. Do not rewrite the whole workflow.

## Why This Matters

A spatial join can be correct and still be based on the wrong fields, geometry type, or CRS.

A data contract catches those problems at the boundary, where the error is easiest to understand. The exact column lists and explicit mapping also make the workflow easier to review, test, and rerun when a source dataset changes.

For `mc-park-access`, this protects the existing analysis and Leaflet proof map without replacing them. The project now has one clear schema inspection stage, one explicit mapping stage, and a deliberate CRS boundary.

## Next Steps

- Add schema inspection and one explicit mapping stage to `mc-park-access`.
- Preserve the existing EPSG:2236 analysis workflow.
- Keep EPSG:4326 limited to copied browser-export layers.
- Add tests for missing fields, missing CRS, geometry types, and returned CRS.
- Run the nearest-park validation only after the contract succeeds.