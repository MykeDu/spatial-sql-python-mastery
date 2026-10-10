# Day 63: Review the Parameter Boundary

## Focus

Review a PostGIS nearest-park screening query by keeping two decisions separate: select the nearest park first, then apply the access-gap threshold to the completed distance result.

The work stayed focused on the Marion County park-access workflow, with source geometry in EPSG:4326 and analysis geometry prepared in EPSG:2236.

## What I Learned

The nearest-match stage answers one spatial question:

> For each subdivision representative point, which park is nearest?

The outer filter answers a different reporting question:

> Which completed nearest-park matches are farther than 10,560 feet?

Those decisions should not be mixed together. `CROSS JOIN LATERAL` lets the nearest-park lookup reference the current subdivision row. The KNN operator, `<->`, orders park candidates by proximity, and `LIMIT 1` keeps the nearest candidate.

`ST_Distance` then calculates `distance_ft` between the subdivision access point and the selected park geometry. Because the analysis geometries are prepared in EPSG:2236, the result is measured in US survey feet.

## What I Practiced

I worked through the query in three stages:

1. Prepare subdivision representative points with `ST_PointOnSurface(ST_Transform(geom, 2236))`.
2. Transform park geometries to EPSG:2236 and select one nearest park per subdivision with `CROSS JOIN LATERAL`, `<->`, and `LIMIT 1`.
3. Filter the completed `distance_ft` values with `WHERE distance_ft > 10560` and sort the largest gaps first.

I also wrote the complete PostGIS query and explained how the lateral lookup, distance calculation, and outer filter work together.

The SQL was correct on the first try. It included the representative point, projected analysis geometry, nearest-neighbor lookup, straight-line distance calculation, post-match threshold, and descending sort.

## The Pattern

```text
Prepare the geometry.
Create one representative point per subdivision.
Select the nearest park.
Calculate the completed distance.
Apply the threshold after matching.
Sort the retained evidence.
```

In plain English:

> Select each subdivision's nearest park and calculate that distance. Then return only the subdivisions whose completed nearest-park distance is greater than 10,560 feet.

The important boundary is between method and policy. The nearest-park method stays stable. The 10,560-foot value is a reporting rule applied after the evidence exists.

## AI Pairing Prompt

```text
Explain my PostGIS nearest-park screening query line by line before suggesting changes.

Both source tables are EPSG:4326. The query must prepare EPSG:2236 geometry, use ST_PointOnSurface for each subdivision, and select one nearest park with CROSS JOIN LATERAL, <->, and LIMIT 1.

I need distance_ft to mean straight-line US-survey-foot distance from the representative point to the selected nearest park. Review whether the 10560-foot threshold is applied only after each nearest match and distance are calculated. Keep road-network distance, geometry repair, and a different nearest-neighbor method out of scope.
```

The prompt names the source CRS, analysis CRS, representative-point method, nearest-neighbor method, distance meaning, and threshold boundary. It also tells the AI partner to explain the query before proposing edits.

## Why This Matters

A query can return plausible results while still asking the wrong spatial question. If the threshold is pushed inside the nearest lookup, candidate parks can be removed before the actual nearest park is selected.

Keeping the stages separate makes the query easier to inspect, test, and reuse. The completed nearest-match result becomes evidence that can support a thresholded report, a map, or a future planning control without changing the underlying method.

This is also a strong AI-pairing habit. State the exact method you want to preserve, define the measurement, and name the changes that are out of scope. That gives the AI a narrow review boundary instead of inviting it to rewrite the workflow.

## Next Steps

- Keep nearest-park selection and threshold screening as separate query stages.
- Continue using EPSG:2236 for straight-line distance measurements in US survey feet.
- Preserve `ST_PointOnSurface`, `CROSS JOIN LATERAL`, `<->`, and `LIMIT 1` in the workflow.
- Use the completed `distance_ft` evidence for later reports and map outputs.
- Continue the next Spatial SQL practice with the method and policy boundary clearly named.