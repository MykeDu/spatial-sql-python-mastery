# Day 36: Review-First Residential Classification

## Focus

Build a review-first residential classification for the Marion County park-access workflow.

The project rule is deliberately conservative:

- Do not infer residential status from subdivision names.
- Store review decisions in `data/classification_review.csv`.
- Key the CSV by `subdivision_id`.
- Use `1` for confirmed residential, `0` for confirmed non-residential, and a blank value when no decision has been made.
- Keep `reason` alongside each reviewed decision.
- Do not rerun residential-only ranking until enough records have been reviewed.

## What I Learned

A classification field should represent an evidence-backed decision, not a guess based on a convenient text pattern. Subdivision names can help identify records for review, but they should not silently determine whether a subdivision is residential.

The review CSV creates an explicit decision layer:

```text
subdivision_id,is_residential,reason
101,1,Residential subdivision confirmed by review
102,0,Commercial or institutional land use confirmed
103,,Needs manual review
```

Blank values are meaningful. They preserve uncertainty instead of turning an unresolved record into a false positive or false negative.

The join also needs validation. A one-to-one join protects the workflow from duplicated review rows or duplicated source keys that could silently multiply records.

## What I Practiced

I created the review file with the required columns:

```python
review = pd.read_csv("data/classification_review.csv")

required_columns = {
    "subdivision_id",
    "is_residential",
    "reason",
}
missing_columns = required_columns - set(review.columns)

if missing_columns:
    raise ValueError(f"Missing required columns: {missing_columns}")
```

I validated the non-missing classification values before joining:

```python
non_null_values = set(
    review["is_residential"].dropna().unique()
)
allowed_values = {0, 1}

unexpected_values = non_null_values - allowed_values
if unexpected_values:
    raise ValueError(
        f"Unexpected non-null classification values: {unexpected_values}"
    )
```

Then I copied the source GeoDataFrame before attaching the review decisions:

```python
classified = subdivisions.copy()

classified = classified.merge(
    review[
        ["subdivision_id", "is_residential", "reason"]
    ],
    on="subdivision_id",
    how="left",
    validate="one_to_one",
)
```

The left join preserves every source subdivision. Records without a review decision remain `NULL` in `is_residential`, so the unresolved queue stays visible.

I reported all three states before deciding whether the downstream analysis was ready:

```python
classification_counts = (
    classified["is_residential"]
    .value_counts(dropna=False)
    .rename_axis("is_residential")
    .rename("record_count")
)

print(classification_counts)
```

The checkpoint is intentional: if the `NULL` count is still substantial, I stop and review more records rather than rerunning the residential-only park-access ranking.

I also completed the `NOT EXISTS` practice correctly on the first try. That reinforced the SQL foundation, but today's main build was the classification-review layer that makes the later analysis defensible.

## The Pattern

```text
Create review CSV
->
Validate columns and values
->
Copy the source GeoDataFrame
->
Left join by subdivision_id
->
Validate one-to-one keys
->
Preserve unmatched records as NULL
->
Report counts for 1, 0, and NULL
->
Checkpoint review coverage
->
Only then rerun residential-only ranking
```

Keep `EPSG:2236` for distance analysis. Create `EPSG:4326` copies only for Leaflet or other browser exports.

## AI Pairing Prompt

```text
Act as a careful data-quality partner for my Marion County park-access project.

Review this classification workflow and check that it:
- Reads data/classification_review.csv with subdivision_id, is_residential, and reason.
- Treats only 0 and 1 as valid non-null classifications.
- Uses a copied source GeoDataFrame.
- Performs a validated one-to-one left join on subdivision_id.
- Preserves unmatched source records as NULL.
- Reports counts for 1, 0, and NULL.
- Stops before rerunning residential-only ranking if review coverage is insufficient.
- Keeps EPSG:2236 for analysis and uses EPSG:4326 only for Leaflet export copies.

Do not infer residential status from subdivision names. Identify the smallest implementation change needed to make the workflow auditable.
```

## Why This Matters

This build separates evidence from analysis. The review CSV records what was decided and why, while the left join preserves the full source layer and keeps unresolved records visible.

That separation prevents a guessed classification from becoming a hidden filter. It also creates a repeatable checkpoint: review coverage can be measured before the project produces a new residential-only ranking or map.

## Next Steps

- Review enough subdivision records to reduce the unresolved `NULL` queue.
- Add concise, evidence-based reasons for every `1` and `0`.
- Re-run the three-state audit after each review batch.
- Proceed to residential-only park-access ranking only after the checkpoint shows adequate coverage.
- Keep the raw source, review CSV, joined working copy, and browser export as distinct project artifacts.