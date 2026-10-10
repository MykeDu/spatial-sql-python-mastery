# Day 65: Named Threshold Policy

## Focus
Parameterize the park-access screening query without weakening its spatial method. The goal was to make the threshold explicit, reviewable, and easy to change while keeping every geometry and distance decision in the right place.

## What I Learned
A `params` CTE gives the policy a name and a clear home. The analysis geometry stays in EPSG:2236, where the distance result is measured in feet. Subdivision representatives come from `ST_PointOnSurface`, and prepared park geometry remains available for the nearest-match step. The 10,560-foot value is not part of the nearest-park calculation. It is a screening policy applied only after the nearest park has been found.

## What I Practiced
I built a first-try successful query pattern that uses `CROSS JOIN LATERAL`, KNN ordering with `<->`, and `LIMIT 1` to return one nearest park for each subdivision. After that match exists, the outer query calculates `distance_ft` and applies the named threshold from `params`. This keeps nearest-neighbor evidence separate from the access-gap decision.

## The Pattern
Use this sequence:

1. Define the screening value in a `params` CTE.
2. Keep subdivision and park analysis geometry in EPSG:2236.
3. Create one representative point per subdivision with `ST_PointOnSurface`.
4. Prepare the park geometry used by the nearest search.
5. Use `CROSS JOIN LATERAL` with `<->` and `LIMIT 1` for one nearest park.
6. Calculate `distance_ft` after the match.
7. Apply the threshold in the outer query.

The query should answer two different questions in order: Which park is nearest? Does that distance exceed the named policy threshold?

## AI Pairing Prompt
Review this PostGIS query as a spatial SQL educator. Explain the query's method before suggesting any edits. Confirm that the analysis geometry remains in EPSG:2236, that `ST_PointOnSurface` creates subdivision representatives, that `CROSS JOIN LATERAL` with `<->` and `LIMIT 1` returns one nearest park per subdivision, that `distance_ft` is calculated after the match, and that the 10,560-foot threshold is applied only in the outer query. Protect this method. Do not rewrite the query, change the CRS, move the threshold into the nearest-neighbor step, or introduce a different distance workflow unless you identify a specific correctness problem and explain it first.

## Why This Matters
Named policy values make spatial screening easier to audit and maintain. The nearest-park result is evidence, while the threshold is a decision rule. Keeping those layers separate prevents a policy change from accidentally changing the spatial method. Reaching Day 65 with a first-try success also shows that a repeatable query structure can carry the work forward without unnecessary experimentation.

## Next Steps
Reuse this pattern for other access screens with a named parameter, a projected analysis CRS, representative features, nearest-neighbor evidence, and an outer policy filter. Add a small validation query that reports the threshold used, the distance units, the nearest park identifier, and the final pass or fail status.