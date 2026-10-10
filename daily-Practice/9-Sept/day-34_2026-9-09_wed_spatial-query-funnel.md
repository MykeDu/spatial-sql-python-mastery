# Day 34: The Spatial Query Funnel

## Focus

Make a spatial query do less work before it does expensive geometry work.

Today's pattern is to funnel the data through a small, intentional workflow:

- Filter to confirmed residential subdivisions early.
- Retain only the columns needed for the result.
- Use `ST_DWithin` in `EPSG:2236`, where one mile is `5,280` US survey feet.
- Use `NOT EXISTS` to find residential subdivisions with no nearby park.
- Aggregate early when possible, then rejoin geometry only when the map or final output needs it.

## What I Learned

Query performance is shaped by the order of operations, not just by the final spatial predicate.

The first step is to reduce the candidate set. Filtering `is_residential = 1` in the initial CTE prevents non-residential subdivisions from entering the rest of the workflow. Selecting only `subdivision_id`, `name`, and `geom_2236` keeps the working row small and makes the intent clear.

The distance value must match the CRS units. In `EPSG:2236`, the geometry uses US survey feet, so one mile is `5,280` feet. That is different from the meter-based threshold used in other projected CRSs.

`NOT EXISTS` expresses the business question directly: keep a residential subdivision only when no park satisfies the nearby-distance test.

## What I Practiced

I completed the Funnel a Park-Access Query exercise with this working query:

```sql
WITH residential_subdivisions AS (
  SELECT subdivision_id, name, geom_2236
  FROM subdivisions
  WHERE is_residential = 1
)
SELECT s.subdivision_id, s.name
FROM residential_subdivisions AS s
WHERE NOT EXISTS (
  SELECT 1
  FROM parks AS p
  WHERE ST_DWithin(s.geom_2236, p.geom_2236, 5280)
);
```

The query filters confirmed residential subdivisions before the spatial search, carries only the needed fields, and returns only subdivisions with no park within one mile.

A specific win: I completed the implementation correctly on the first try. The knowledge check also confirmed that `5,280` is the correct one-mile distance for `EPSG:2236`.

For the broader project workflow, I kept `EPSG:2236` for analysis and `EPSG:4326` for browser copies. When a later map or output needs geometry, the aggregate or filtered result can be rejoined to the geometry only at that point.

## The Pattern

```text
Filter confirmed residential subdivisions
 |
 v
Keep only needed columns
 |
 v
Use EPSG:2236 distance units
 |
 v
Apply ST_DWithin(..., 5280)
 |
 v
Use NOT EXISTS for "no nearby park"
 |
 v
Aggregate early when possible
 |
 v
Rejoin geometry for the final map or output
```

In plain English:

> Reduce the rows, reduce the columns, do the spatial test in the correct units, and delay geometry work until it earns its cost.

## AI Pairing Prompt

```text
Review this park-access query as an implementation-focused SQL partner.

Requirements:
- Filter to confirmed residential subdivisions with is_residential = 1.
- Keep only subdivision_id, name, and geom_2236 in the working CTE.
- Use EPSG:2236 for analysis.
- Treat 5,280 US survey feet as one mile.
- Use ST_DWithin to test the nearby-park distance.
- Use NOT EXISTS to return residential subdivisions with no nearby park.
- Aggregate early when an intermediate count or summary can reduce the working set.
- Rejoin geometry only when the final map or output needs it.

Confirm the query logic first. Then identify only the narrowest implementation improvement that preserves the same result and keeps the funnel readable.
```

I earned a 100% prompt mastery score on this drill.

## Why This Matters

The spatial query funnel turns performance guidance into a repeatable implementation pattern.

Filtering early reduces the number of rows that reach the spatial predicate. Selecting only needed columns reduces the amount of data carried through each step. Using the correct projected CRS keeps the distance threshold meaningful, while `NOT EXISTS` keeps the no-nearby-park rule easy to verify.

The pattern also supports the project handoff: analysis can stay in `EPSG:2236`, while geometry is rejoined and copied to `EPSG:4326` only when the browser needs it.

## Next Steps

- Compare the funnelled query with a version that filters and selects later.
- Inspect the execution plan before and after the spatial predicate.
- Add an aggregate summary when the next project question needs counts rather than features.
- Rejoin geometry only for the final map layer or exported result.
- Keep the one-mile unit conversion documented beside the query.