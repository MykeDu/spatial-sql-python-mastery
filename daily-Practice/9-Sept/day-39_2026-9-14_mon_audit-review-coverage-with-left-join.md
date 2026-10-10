# Day 39: Audit Review Coverage With a LEFT JOIN

## Focus

Make review coverage visible without losing any subdivision from the source layer.

Today's pattern is a left-join coverage audit: start with `subdivisions`, attach documented rows from `classification_review` by `subdivision_id`, and keep unresolved subdivisions in the result instead of filtering them away.

## What I Learned

A `LEFT JOIN` preserves every row from the left, or source, table:

- Every subdivision remains visible, even when no review row exists.
- Review fields are `NULL` when a subdivision has not been reviewed.
- `COUNT(*)` counts rows in the joined result.
- `COUNT(r.subdivision_id)` counts only rows with a matching review ID.
- The difference between those counts exposes unreviewed subdivisions.
- A duplicate `subdivision_id` in the review table can multiply source rows and make coverage counts misleading.

The distinction between `COUNT(*)` and `COUNT(r.subdivision_id)` is the core audit check. I should inspect coverage before applying a residential filter. Unknown review values are unresolved evidence, not a reason to guess a classification.

## What I Practiced

I joined the review table to the full source layer:

```sql
SELECT
    s.subdivision_id,
    s.name,
    r.is_residential,
    r.source,
    r.rationale
FROM subdivisions AS s
LEFT JOIN classification_review AS r
    ON s.subdivision_id = r.subdivision_id;
```

Then I measured total, reviewed, and unreviewed coverage:

```sql
SELECT
    COUNT(*) AS source_subdivisions,
    COUNT(r.subdivision_id) AS reviewed_subdivisions,
    COUNT(*) - COUNT(r.subdivision_id) AS unreviewed_subdivisions
FROM subdivisions AS s
LEFT JOIN classification_review AS r
    ON s.subdivision_id = r.subdivision_id;
```

I did not add a premature condition such as `WHERE r.is_residential = 1`. That would remove unresolved rows from the audit and turn a coverage question into a filtered classification result.

For the Python attachment audit, I used a left merge, validated the expected key relationship, and checked that the source row count survived:

```python
import geopandas as gpd
import pandas as pd

source_row_count = len(subdivisions_gdf)

attached = subdivisions_gdf.merge(
    classification_review_df,
    on="subdivision_id",
    how="left",
    validate="one_to_one",
)

attached = gpd.GeoDataFrame(
    attached,
    geometry="geometry",
    crs=subdivisions_gdf.crs,
)

if len(attached) != source_row_count:
    raise ValueError("The left merge changed the source row count.")

print("Source rows:", len(subdivisions_gdf))
print("Attached rows:", len(attached))
print(attached["is_residential"].value_counts(dropna=False))
```

`validate="one_to_one"` is important because it catches duplicate IDs on either side instead of silently creating extra rows. I also preserved the raw geometry and source layer, leaving classification decisions in the attached audit result.

## The Pattern

```text
source subdivisions
    -> LEFT JOIN review rows by subdivision_id
    -> compare COUNT(*) with COUNT(review_id)
    -> inspect unresolved NULL review values
    -> validate duplicate-ID and row-count risks
    -> filter only after coverage is understood
```

In Python:

```text
subdivisions_gdf
    -> left merge with validate="one_to_one"
    -> check source rows == attached rows
    -> inspect value_counts(dropna=False)
    -> preserve raw geometry and source
```

## AI Pairing Prompt

> Review this left-join coverage audit. Confirm that `subdivisions` is the left/source table, `classification_review` joins by `subdivision_id`, and unresolved rows remain visible. Check the difference between `COUNT(*)` and `COUNT(r.subdivision_id)`, look for duplicate-ID risk, and verify that the Python left merge uses `validate="one_to_one"` and preserves the source row count and geometry. Do not guess missing classifications and do not add a residential filter before coverage is audited.

## Why This Matters

A classification workflow can look complete simply because unresolved rows disappeared. The left join makes the missing work visible, while the count comparison gives a direct, reproducible measure of review coverage.

This also protects downstream spatial analysis. If duplicate review IDs multiply features or an inner join drops subdivisions, later maps and distance calculations can be based on the wrong population. Auditing the join first keeps the source layer intact and makes every later filter explainable.

## Next Steps

- Run the SQL coverage query and record source, reviewed, and unreviewed counts.
- Check `classification_review` for duplicate `subdivision_id` values before trusting the join.
- Run the Python merge with `validate="one_to_one"` and confirm that source and attached row counts match.
- Inspect `value_counts(dropna=False)` so unresolved `NULL` or `NaN` review values remain explicit.
- Create a confirmed-residential subset only after the full coverage audit is complete.