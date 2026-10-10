# Day 62: Parameterized access-gap screening

## Focus

Turn the nearest-park output into a ranked access-gap screen.

The workflow uses GeoPandas with already-projected `subdivisions_2236` and `parks_2236` layers in EPSG:2236. The job today was to create representative points, preserve the full nearest-match result, and sort the completed matches so the largest gaps are easy to inspect.

## What I Learned

`representative_point()` creates a point inside each subdivision geometry for the nearest-park comparison. It is a practical stand-in for the subdivision, not a claim about where every resident lives.

`gpd.sjoin_nearest()` attaches the nearest park to each representative point and can write the measured value into `distance_ft` with `distance_col='distance_ft'`.

The reporting step stays separate from the matching step:

```python
subdivision_points = subdivisions_2236.copy()
subdivision_points["representative_point"] = (
    subdivision_points.geometry.representative_point()
)

nearest_matches = gpd.sjoin_nearest(
    subdivision_points.set_geometry("representative_point"),
    parks_2236,
    how="left",
    distance_col="distance_ft",
)

ranked_matches = nearest_matches.sort_values(
    "distance_ft",
    ascending=False,
).copy()

top_10_access_gaps = ranked_matches.head(10).copy()
```

The full `ranked_matches` output matters. It keeps ties, repeated subdivision IDs, and the complete nearest-match evidence available for inspection instead of hiding those cases inside a top-ten result.

## What I Practiced

I completed the practice by building the GeoPandas workflow around the existing projected layers:

1. Copy `subdivisions_2236` before adding the representative point geometry.
2. Create representative points with `representative_point()`.
3. Run `gpd.sjoin_nearest()` against `parks_2236`.
4. Store the distance in `distance_ft`.
5. Sort completed matches descending by `distance_ft`.
6. Return both `ranked_matches` and `top_10_access_gaps`.

The result is a parameterized screen, not a new distance model. It ranks the nearest-match distances already produced by the spatial join.

## The Pattern

Preserve the evidence, then rank the result.

1. Start with projected source layers whose CRS and units are known.
2. Create a representative point for each subdivision.
3. Run one nearest join for every representative point.
4. Keep the full output before applying `head(10)` or another reporting limit.
5. Sort by the measured distance only after the matching step is complete.
6. Treat the top ten as a view of the evidence, not a replacement for the evidence.

These distances are straight-line measurements in US survey feet because the workflow uses EPSG:2236. They do not measure road-network accessibility, walking distance, travel time, barriers, or the distance from every home to a park.

## AI Pairing Prompt

Review this GeoPandas workflow as a coding partner before editing it.

First, explain what each step does and identify any assumptions about CRS, geometry columns, join direction, distance units, ties, and repeated subdivision IDs. Confirm that `representative_point()` is used, that `gpd.sjoin_nearest()` writes `distance_ft`, and that the full nearest-match output is preserved before creating `ranked_matches` and `top_10_access_gaps`.

Do not edit the code yet. After the explanation, list only the changes you recommend and why. Wait for my approval before rewriting anything.

## Why This Matters

A ranked access-gap screen is useful only when the ranking can be traced back to the full match output. Keeping `ranked_matches` separate from `top_10_access_gaps` makes the result auditable and keeps reporting choices from changing the underlying evidence.

The method also keeps the claim honest. A large `distance_ft` value identifies a subdivision whose representative point is far from its nearest park in straight-line space. It does not prove poor real-world access.

The practice was completed, and the AI pairing prompt reached 100% mastery. That score reflects prompt mastery, not first-try code success. The stored result was later reanalyzed as 0%, so I am not treating this as a first-pass success claim.

## Next Steps

- Inspect the full `ranked_matches` output for ties and repeated subdivision IDs.
- Verify the CRS and units before sharing the ranking.
- Compare the straight-line screen with a road-network or walkability analysis before making access claims.
- Keep the explain-before-edit guardrail in future AI reviews.