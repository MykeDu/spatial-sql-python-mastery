# Day 52, September 27, 2026

## Focus

**Boolean Filters in SQL and GeoPandas**

Today's focus was translating the usable-geometry gate between SQL and GeoPandas. The goal was not to memorize two different solutions. The goal was to recognize the same inclusion rule in two tools and carry it across without changing the meaning.

The gate is simple:

- keep records where the geometry is present
- keep records where the geometry is valid
- exclude everything else from the analysis layer

## What I Learned

A geometry gate is an explicit boolean filter, not a vague cleanup step.

In SQL, the logic reads directly:

```sql
WHERE geom IS NOT NULL
  AND ST_IsValid(geom)
```

In GeoPandas, the same rule becomes a boolean mask:

```python
usable_mask = (
    subdivisions["geometry"].notna()
    & subdivisions["geometry"].is_valid
)

usable_subdivisions = subdivisions.loc[usable_mask].copy()
```

The important detail is the `&`. Each condition returns a boolean Series, and the parentheses make the combined rule clear. In pandas and GeoPandas, `&` combines element-by-element conditions. Python's `and` is not the right tool for Series-level filtering.

The other important detail is `.copy()`. The source layer stays intact, while the filtered result becomes a separate working object.

## What I Practiced

I translated the geometry gate from SQL into GeoPandas and implemented the same present-and-valid rule in both places.

The source-preserving pattern is:

```python
source_subdivisions = subdivisions.copy()

usable_mask = (
    source_subdivisions["geometry"].notna()
    & source_subdivisions["geometry"].is_valid
)

usable_subdivisions = source_subdivisions.loc[usable_mask].copy()
```

For Marion County, the audit counts were:

- **3,473 total features**
- **1 feature with missing geometry**
- **16 features with invalid geometry**
- **3,456 features passing the usable-geometry gate**

That last number is the working set for downstream spatial analysis. The rejected records still matter because they explain the difference between the full source layer and the analysis-ready layer.

## The Pattern

Use the same three-step pattern every time:

1. Preserve the source with `.copy()`.
2. Build an explicit boolean mask for every inclusion rule.
3. Filter with `.loc[mask].copy()`.

```python
source = gdf.copy()

usable_mask = (
    source["geometry"].notna()
    & source["geometry"].is_valid
)

usable = source.loc[usable_mask].copy()
```

The SQL translation is equally direct:

```sql
SELECT *
FROM subdivisions
WHERE geom IS NOT NULL
  AND ST_IsValid(geom);
```

This is the geometry gate: present and valid. Keep the rule visible in the code. Do not hide it inside an unexplained helper or silently overwrite the source data.

## AI Pairing Prompt

> Review the audit documentation boundary for this spatial layer. Confirm that the usable-geometry gate is expressed consistently in SQL and GeoPandas, that the source layer is preserved with `copy()`, and that the documentation distinguishes the full source count from the present-and-valid analysis count. Report any mismatch in the boolean inclusion rules, missing-geometry count, invalid-geometry count, or downstream working-set count.

I completed the prompt drill with a **100% mastery score**. The prompt asked for a review of the audit documentation boundary, and the response kept the technical rule, source preservation, and evidence counts connected.

## Why This Matters

SQL and GeoPandas are different interfaces for the same data decision. If the usable-geometry gate changes during translation, the analysis can change without anyone noticing.

An explicit mask makes the decision inspectable. `.copy()` protects the source. The counts make the boundary auditable. Together, those habits produce a working layer that can be trusted and a source layer that can still be explained.

## Next Steps

- Carry the same boolean-mask discipline into other inclusion rules.
- Keep source counts, rejected counts, and usable counts visible in audit notes.
- Use the geometry gate before spatial joins, overlays, and area calculations.
- Continue translating patterns between SQL and GeoPandas instead of treating them as separate skills.