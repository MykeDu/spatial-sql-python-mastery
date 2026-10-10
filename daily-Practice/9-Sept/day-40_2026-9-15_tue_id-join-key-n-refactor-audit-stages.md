# Day 40: Identify the `JOIN_KEY` and Refactor the Audit Stages

## Focus

Move from the coverage-audit idea into a reliable project implementation.

The task is to identify the real `JOIN_KEY`, verify that IDs are safe to use, preserve every subdivision, attach documented review evidence only when IDs match, and count both reviewed and unknown records. Then refactor `nearby_analysis` into named stages so the merge, audit, filter, and nearest-park work can be inspected independently.

## What I Learned

The join key is a data contract, not just a column name. Before merging, I need to confirm:

- which field identifies the same subdivision in both datasets;
- whether the key is present, consistently typed, and normalized;
- whether either side contains duplicate IDs;
- whether the merge preserves the source row count.

The source subdivisions remain the left side of the merge. A matching review row contributes documented evidence; an unmatched row stays in the output with unknown review fields. That distinction lets me count coverage without silently dropping unresolved records.

A duplicate review ID can multiply a subdivision during the merge. An unknown classification is also different from a negative classification: it means the evidence is incomplete, not that the subdivision should be treated as non-residential.

## What I Practiced

I first made the join key explicit and checked it before attaching review evidence:

```python
JOIN_KEY = "subdivision_id"

def check_join_key(df, name):
    if JOIN_KEY not in df.columns:
        raise KeyError(f"{name} is missing {JOIN_KEY!r}")
    if df[JOIN_KEY].isna().any():
        raise ValueError(f"{name} contains null {JOIN_KEY} values")
    if df[JOIN_KEY].duplicated().any():
        duplicates = df.loc[df[JOIN_KEY].duplicated(), JOIN_KEY].unique()
        raise ValueError(f"{name} contains duplicate IDs: {duplicates[:5]}")

check_join_key(subdivisions_gdf, "subdivisions")
check_join_key(classification_review_df, "classification_review")
```

Then I preserved every subdivision with a validated left merge:

```python
source_count = len(subdivisions_gdf)

audited = subdivisions_gdf.merge(
    classification_review_df,
    on=JOIN_KEY,
    how="left",
    validate="one_to_one",
    indicator=True,
)

if len(audited) != source_count:
    raise ValueError("The audit merge changed the source row count.")

reviewed_count = (audited["_merge"] == "both").sum()
unknown_count = (audited["_merge"] == "left_only").sum()

print("Reviewed:", reviewed_count)
print("Unknown:", unknown_count)
```

Finally, I separated `nearby_analysis` into named stages:

```python
def nearby_analysis(subdivisions_gdf, reviews_df, parks_gdf):
    merged = merge_review_evidence(subdivisions_gdf, reviews_df)
    audited = audit_review_coverage(merged)
    residential = filter_reviewed_residential(audited)
    nearest = attach_nearest_park(residential, parks_gdf)
    return nearest
```

Each stage has one job. The residential filter runs after the audit, and nearest-park work runs after the reviewed set is defined.

## The Pattern

```text
identify JOIN_KEY
    -> check nulls, types, and duplicate IDs
    -> preserve every subdivision with a left merge
    -> attach evidence only for matching IDs
    -> count reviewed and unknown records
    -> refactor merge, audit, filter, and nearest-park stages
```

A useful implementation boundary is:

```text
merge_review_evidence(...)
audit_review_coverage(...)
filter_reviewed_residential(...)
attach_nearest_park(...)
```

The names make the data flow visible and make it easier to test each assumption without rerunning the entire analysis.

## AI Pairing Prompt

> Review this geospatial pipeline and identify the real `JOIN_KEY` used by the source and review datasets. Check key presence, nulls, data types, and duplicate IDs before merging. Confirm that the subdivisions layer remains the left side, that the merge preserves its row count, and that only matching IDs receive documented review evidence. Report reviewed and unknown counts separately. Then refactor `nearby_analysis` into named stages for merge, audit, residential filtering, and nearest-park attachment. Do not infer unknown classifications and do not filter before the coverage audit.

## Why This Matters

A readable audit is the evidence behind a defensible ranking. If the join key is wrong, duplicated, or assumed rather than verified, the analysis can attach the wrong review to a subdivision or create phantom rows. Named stages make those failures easier to detect and keep later residential-only and nearest-park decisions from hiding unresolved coverage gaps.

## Next Steps

- Confirm the production datasets use the same `JOIN_KEY` and compatible types.
- Keep duplicate-ID checks and row-count assertions in the workflow.
- Review the audited unknown records before applying residential-only logic.
- Add focused tests for each named stage in `nearby_analysis`.
- Use the validated, reviewed subset as the input to the next ranking step.