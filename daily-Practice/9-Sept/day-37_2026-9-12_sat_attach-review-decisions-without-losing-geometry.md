# Day 37: Attach Review Decisions Without Losing Geometry

## Focus

Attach reviewed residential decisions to `subdivisions_gdf` without changing the source row count, dropping geometry, or losing the coordinate reference system.

Today's implementation uses `classification_review.csv` as a separate decision table. The join key is `subdivision_id`, and the review table documents four things:

- `subdivision_id`
- `is_residential`
- `source`
- `rationale`

The goal is to attach evidence to the spatial data, not to filter the map yet.

## What I Learned

A GeoPandas merge should preserve the source layer when the source GeoDataFrame is on the left and the join uses `how="left"`. That keeps every subdivision row and its geometry, including subdivisions with no review decision.

The review table needs strict validation before it touches the spatial data:

- `subdivision_id` must not be duplicated in the review table.
- Non-missing `is_residential` values must be only `0` or `1`.
- Missing values must remain missing because they mean the record is unresolved.
- `source` and `rationale` make the decision traceable.
- The merged GeoDataFrame should retain its original CRS.

Most importantly, attaching review decisions and filtering residential records are separate steps. The merge builds a complete, auditable layer. A later filter can use that layer when the review coverage is ready.

## What I Practiced

I loaded the review table, checked its structure, rejected duplicate review IDs, and validated the classification values:

```python
import pandas as pd
import geopandas as gpd

review = pd.read_csv("data/classification_review.csv")

required_columns = {
    "subdivision_id",
    "is_residential",
    "source",
    "rationale",
}
missing_columns = required_columns - set(review.columns)

if missing_columns:
    raise ValueError(f"Missing required columns: {sorted(missing_columns)}")

duplicate_ids = review.loc[
    review["subdivision_id"].duplicated(keep=False),
    "subdivision_id",
].unique()

if len(duplicate_ids):
    raise ValueError(
        f"Duplicate review subdivision_id values: {duplicate_ids.tolist()}"
    )

allowed_values = {0, 1}
non_missing_values = set(review["is_residential"].dropna().unique())
unexpected_values = non_missing_values - allowed_values

if unexpected_values:
    raise ValueError(
        f"Unexpected is_residential values: {unexpected_values}"
    )
```

Then I saved the source CRS, made a copy, and performed a validated left merge:

```python
source_crs = subdivisions_gdf.crs
source_row_count = len(subdivisions_gdf)

attached = subdivisions_gdf.copy()

attached = attached.merge(
    review[
        [
            "subdivision_id",
            "is_residential",
            "source",
            "rationale",
        ]
    ],
    on="subdivision_id",
    how="left",
    validate="one_to_one",
)

if len(attached) != source_row_count:
    raise ValueError("The merge changed the source row count.")

if attached.crs != source_crs:
    raise ValueError("The merge changed the source CRS.")

if attached.geometry.isna().any():
    raise ValueError("The merge introduced missing geometry.")
```

I checked the result before applying any residential filter:

```python
review_counts = (
    attached["is_residential"]
    .value_counts(dropna=False)
    .rename_axis("is_residential")
    .rename("record_count")
)

print(review_counts)
print(attached[[
    "subdivision_id",
    "is_residential",
    "source",
    "rationale",
    "geometry",
]].head())
```

Only after the attachment step is stable should filtering happen separately:

```python
residential_only = attached.loc[
    attached["is_residential"].eq(1)
].copy()
```

This keeps the complete attached layer available for audit and future review.

## The Pattern

```text
Read the review table
->
Check required columns
->
Reject duplicate subdivision_id values
->
Validate is_residential as 0, 1, or missing
->
Save source row count and CRS
->
Copy the source GeoDataFrame
->
Left merge the review decisions
->
Validate row count, geometry, and CRS
->
Report reviewed and unresolved records
->
Filter separately only when analysis is ready
```

The geometry stays with the left-side GeoDataFrame. The CRS belongs to the GeoDataFrame, so a normal attribute merge should preserve it, but checking it makes the workflow explicit and defensible.

## AI Pairing Prompt

```text
Act as a data-quality partner for my GeoPandas workflow.

Review this task: merge data/classification_review.csv onto subdivisions_gdf using subdivision_id.

The implementation must:
- Preserve every source row and every source geometry.
- Use a left merge with validate="one_to_one".
- Reject duplicate subdivision_id values in the review table before merging.
- Accept only 0, 1, or missing values in is_residential.
- Retain source and rationale for each decision.
- Verify that the merged GeoDataFrame keeps the original CRS.
- Keep filtering residential records separate from the attachment step.
- Report the counts of confirmed residential, confirmed non-residential, and unresolved records.

Do not infer classifications from names or geometry. Identify any silent data-loss risk, and suggest the smallest practical fix.
```

## Why This Matters

A classification decision is only useful if it can be traced back to evidence and safely attached to the spatial records it describes. This pattern preserves the full source layer, keeps unresolved cases visible, and prevents a bad review row from multiplying features or silently changing the map.

It also creates a clean boundary between data preparation and analysis. The attached GeoDataFrame is the auditable source for later work. A residential-only view is a deliberate downstream product, not an accidental side effect of the merge.

## Next Steps

- Add more reviewed records while keeping `source` and `rationale` complete.
- Track how many subdivisions remain unresolved.
- Review the attached layer for unexpected nulls or unmatched IDs.
- Use `residential_only` only after review coverage is sufficient.
- Feed the validated residential subset into the next park-access analysis without replacing the complete attached layer.