# Day 27: Validating and Diagnosing Broken Geometries with PostGIS

## Focus

Today was a Spatial SQL day focused on data validation and avoiding a common trap: assuming the source data is clean.
Before running a spatial join, buffer, union, area calculation, or export, check the geometry first. Invalid geometries can create misleading results or break a workflow farther downstream.

## What I Learned

### 1. The Diagnostic Tool: ST_IsValid

ST_IsValid checks whether a geometry is topologically valid.

```SQL

SELECT
 id,
 ST_IsValid(geom) AS is_valid
FROM land_parcels;

```

This gives each feature a true or false result.

### 2. The Repair Tool: ST_MakeValid

ST_MakeValid repairs invalid geometry when a repair makes sense for the data and analysis.

```SQL	

SELECT
 id,
 ST_MakeValid(geom) AS geom
FROM land_parcels;

```

A repair is not the end of the workflow. The repaired geometry needs to be checked again before it is used in analysis.

## What I Practiced

I wrote a query that identifies invalid parcel geometries and explains why each one fails.

```SQL

SELECT
 id,
 ST_IsValidReason(geom) AS invalid_reason
FROM land_parcels
WHERE NOT ST_IsValid(geom);

```

**The query reads like a sentence:**

Select each parcel ID and the reason its geometry is invalid from land_parcels, where the geometry is not valid.

ST_IsValidReason turns a simple true-or-false check into a usable data-quality report. It can reveal issues such as self-intersections or rings that are not closed.

## The Pattern

Do not assume source geometries are clean.

1. Audit the geometry with ST_IsValid.

2. Diagnose failures with ST_IsValidReason.

3. Repair only when appropriate, often with ST_MakeValid.

4. Validate the output again before using it in analysis.

- Validate first.
- Repair when needed.
- Proceed with analysis confidently.

## AI Pairing Prompt

I practiced using AI as a coding partner for data-cleaning work.

```Prompt

I have a PostGIS table called zoning_boundaries with an id column and a geometry column named geom. 
Write a query that creates a new table called zoning_boundaries_clean using ST_MakeValid to repair invalid geometries. 
Then show me how to validate the cleaned output before using it in analysis.

```
The useful prompt pattern is:
1. Name the source table.

2. Name the relevant columns.

3. State the outcome you need.

4. Specify the spatial function or workflow.

5. Ask for a validation step, not only a repair.

## Why This Matters

This is a production habit, not just a PostGIS function to memorize. Good spatial analysis starts with data you can trust.

Today’s practice moved from reading a validity audit, to explaining the logic, to writing a clean diagnostic query. 

This is a learning pattern I find helpful.

## Next Steps

When I have a real dataset available, I want to run this pattern against parcel, zoning, boundary, or building-footprint data.

Finding usable data can be the hard part, so the practical first move is simple:

- Start with a dataset already used in a work project or personal GIS exercise.

- Run ST_IsValid first and count valid versus invalid geometries.

- Use ST_IsValidReason to inspect the failures.

- Create a separate cleaned output only if the analysis requires repair.

- Validate the output again before using it in a spatial join, area calculation, or export.

The goal is not to repair data automatically. 

It is to understand the condition of the data before trusting the analysis.

