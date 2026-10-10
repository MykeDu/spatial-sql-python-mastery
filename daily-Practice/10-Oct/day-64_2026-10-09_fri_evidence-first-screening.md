# Day 64: Evidence-First Screening

## Focus

Today I am applying an evidence-first pattern to a PostGIS park-access workflow.

The goal is to answer two separate questions in the right order:

1. What is the nearest park for each subdivision?
2. Which subdivisions should appear in the report based on a distance threshold?

The first question is spatial method. The second question is reporting policy. Keeping them separate protects the meaning of the result.

## What I Learned

A nearest-match query should establish the evidence before applying the reporting screen.

For each subdivision, I first:

- Create one representative point with `ST_PointOnSurface`.
- Transform the source geometry from EPSG:4326 to EPSG:2236.
- Use `CROSS JOIN LATERAL` with the KNN operator `<->`.
- Use `LIMIT 1` to select exactly one nearest park.
- Calculate `distance_ft` after the winning park match exists.

Only after that do I apply the threshold in the outer query.

If I filter parks or candidate matches before the nearest-neighbor step, I am no longer asking, "What is the nearest park?" I am asking a different question, such as, "What is the nearest park that survives this filter?" That may be a valid policy, but it is not the same spatial evidence.

## What I Practiced

Here is the complete pattern:

```sql
WITH params AS (
    SELECT 2640.0::double precision AS threshold_ft
),
subdivision_points AS (
    SELECT
        s.subdivision_id,
        s.subdivision_name,
        ST_Transform(
            ST_PointOnSurface(s.geom),
            2236
        ) AS subdivision_point
    FROM mc_subdivisions_wgs84 AS s
),
nearest_parks AS (
    SELECT
        sp.subdivision_id,
        sp.subdivision_name,
        p.park_id,
        p.park_name,
        sp.subdivision_point,
        p.park_geom_2236
    FROM subdivision_points AS sp
    CROSS JOIN LATERAL (
        SELECT
            park_id,
            park_name,
            ST_Transform(geom, 2236) AS park_geom_2236
        FROM mc_parks_wgs84
        ORDER BY ST_Transform(geom, 2236) <-> sp.subdivision_point
        LIMIT 1
    ) AS p
),
matches AS (
    SELECT
        np.subdivision_id,
        np.subdivision_name,
        np.park_id,
        np.park_name,
        ST_Distance(
            np.subdivision_point,
            np.park_geom_2236
        ) AS distance_ft
    FROM nearest_parks AS np
)
SELECT
    m.subdivision_id,
    m.subdivision_name,
    m.park_id,
    m.park_name,
    m.distance_ft,
    p.threshold_ft
FROM matches AS m
CROSS JOIN params AS p
WHERE m.distance_ft > p.threshold_ft
ORDER BY m.distance_ft DESC;
```

Read it like a sentence: build one point per subdivision, find one nearest park for each point, measure the winning pair in feet, then report only matches beyond the configured threshold.

The recent win was keeping the threshold filter after the nearest-match stage. That preserved the evidence first and the reporting policy second.

## The Pattern

Use this boundary:

```text
spatial method -> winning nearest match -> measured distance -> reporting screen
```

Do not move the reporting screen into the lateral subquery unless the business question explicitly changes.

Evidence tells me what happened. Policy tells me what to show. The SQL becomes easier to review when those jobs have visible boundaries in the CTEs and the final `WHERE` clause.

## AI Pairing Prompt

Review this PostGIS query as a bounded SQL reviewer.

Check only these requirements:

1. Each subdivision gets one representative point with `ST_PointOnSurface`.
2. Source geometries are transformed from EPSG:4326 to EPSG:2236 before distance interpretation.
3. `CROSS JOIN LATERAL`, KNN `<->`, and `LIMIT 1` select one nearest park per subdivision.
4. `ST_Distance` is calculated after the nearest park has been selected.
5. `threshold_ft` is defined in a `params` CTE.
6. The threshold filter appears only in the outer reporting query.
7. No early filter changes the meaning from "nearest park" to "nearest qualifying park."

Return:

- PASS or NEEDS REVISION for each requirement
- One short explanation for each result
- Any corrected SQL only if a requirement fails

Do not redesign the query, add indexes, or introduce a new policy.

## Why This Matters

Nearest-neighbor work can look correct while answering the wrong question.

If threshold logic appears too early, a query may hide the actual nearest park and replace it with the nearest park that passed a screen. That makes the reported distance difficult to interpret and weakens the audit trail.

Separating method from policy gives me a result I can explain: this was the nearest park, this was the measured distance, and this is why the record was included in the report.

## Next Steps

- Run the query with the current threshold from the Day 63 parameter-boundary review.
- Inspect a few subdivisions that fall just below and just above the threshold.
- Confirm that every subdivision has exactly one nearest park before the outer screen.
- Keep the evidence CTEs stable when changing reporting thresholds.