# Day 48 - Diagnose Invalid Geometry With SQL

## Focus

Micro-Lesson: Diagnose Invalid Geometry With SQL

Today I focused on the difference between auditing geometry health and repairing geometry. The goal was to identify invalid records and record the reason without changing the source data.

## What I Learned

A geometry audit should answer what is wrong, where it is, and why it is invalid. PostGIS provides `ST_IsValid` for the validation check and `ST_IsValidReason` for a readable explanation of the problem.

For the Marion County source, the observed facts were:

- 3,473 subdivision features
- 1 missing geometry
- 16 invalid geometries

The raw source remains unchanged. `ST_MakeValid` belongs in a separate, deliberate repair stage, not in this read-only audit.

## What I Practiced

I created an invalid-geometry audit log with this query:

```sql
SELECT
  subdivision_id,
  subdivision_name,
  ST_IsValidReason(geom_2236) AS invalid_reason
FROM subdivisions_prepared
WHERE geom_2236 IS NOT NULL
  AND NOT ST_IsValid(geom_2236)
ORDER BY subdivision_id;
```

I wrote the query correctly on the first try. I also completed a prompt drill for requesting a validation review and earned a 90% AI prompt mastery score.

## The Pattern

The pattern is:

1. Exclude NULL geometries when auditing invalid geometry.
2. Use `ST_IsValid` to find invalid records.
3. Use `ST_IsValidReason` to explain each finding.
4. Order the results by a stable identifier.
5. Keep the audit read-only.
6. Plan any repair as a separate, reviewed stage.

This keeps evidence visible and prevents an automatic repair from hiding the original data-quality issue.

## AI Pairing Prompt

Review this read-only PostGIS geometry audit query for correctness and clarity:

- Confirm that NULL geometries are excluded.
- Confirm that only invalid geometries are returned.
- Explain what `ST_IsValidReason` contributes to the audit log.
- Identify any risks or improvements without adding `ST_MakeValid`.
- Suggest a separate follow-up plan for deliberate geometry repair.

## Why This Matters

Geometry validity affects every downstream spatial operation. If invalid records are silently repaired or filtered out, the workflow can lose the evidence needed to understand the source data.

An explicit audit log gives the team a traceable record of geometry health. It also separates diagnosis from repair, making later changes easier to review, test, and explain.

## Next Steps

- Review the invalidity reasons returned for the 16 invalid geometries.
- Group the audit results by reason to identify recurring source issues.
- Preserve the audit output as evidence before any repair work.
- Design a separate repair stage that uses `ST_MakeValid` only after the repair rules are deliberate and documented.
- Re-run the audit after repair and compare the results without overwriting the raw source.