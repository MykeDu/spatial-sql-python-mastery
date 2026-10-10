# Day 53: Geometry Gate in Practice

## Focus

Today's focus is translating the usable-geometry gate between SQL and GeoPandas.

The rule is simple: keep records where geometry is present and valid. Exclude everything else.

## What I Learned

In SQL, the gate is expressed directly in the `WHERE` clause:

```sql
WHERE geom IS NOT NULL AND ST_IsValid(geom)
```

The same logic in GeoPandas uses element-by-element boolean operators:

```python
usable_mask = (
    subdivisions["geometry"].notna()
    & subdivisions["geometry"].is_valid
)

usable_subdivisions = subdivisions.loc[usable_mask].copy()
```

The important detail is using `&`, not Python's `and`, because the condition is evaluated across the entire geometry column.

## What I Practiced

I audited the Marion County subdivisions layer:

- Total features: 3,473
- Missing geometries: 1
- Invalid geometries: 16
- Usable geometries: 3,456

The prompt drill was completed with 100% mastery.

## The Pattern

Use the same three-step pattern every time:

1. Preserve the source layer with `.copy()`.
2. Build an explicit boolean mask for every inclusion rule.
3. Filter with `.loc[mask].copy()`.

This makes the working set intentional, reviewable, and safe to use downstream.

## AI Pairing Prompt

> Check this GeoPandas filtering step against the SQL geometry gate. Confirm that it keeps only records with non-null, valid geometry, uses element-by-element boolean operators correctly, preserves the source layer, and reports source, rejected, and usable counts.

## Why This Matters

A geometry gate prevents bad records from quietly reaching spatial joins, overlays, or area calculations.

The filter is not just cleanup. It defines the trustworthy working set for everything that follows. Keeping the counts visible also makes the audit boundary clear.

## Next Steps

- Apply the geometry gate before spatial joins.
- Apply it before overlays and area calculations.
- Keep source, rejected, and usable counts visible in audit notes.
- Continue translating patterns between SQL and GeoPandas.