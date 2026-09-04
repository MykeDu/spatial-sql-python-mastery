# Day 28: GeoPandas Geometry Validation

## Focus
Apply the professional geometry-validation pattern in Python with GeoPandas:
1. Audit the source data.
2. Isolate invalid geometries.
3. Repair a copied GeoDataFrame.
4. Verify the repair before using the data in analysis.

## What I Learned
GeoPandas provides two direct tools for geometry quality control:

- .is_valid returns a Boolean Series showing whether each geometry is well-formed.
- .geometry.make_valid() repairs invalid geometries when possible.

The Python pattern matches yesterday's PostGIS workflow:
```
PostGIS: ST_IsValid() → ST_MakeValid()
GeoPandas: .is_valid → .make_valid()
```

The spatial logic stays the same. Only the syntax changes.

## What I Practiced

I wrote a GeoPandas workflow that:

- counted valid and invalid parcel geometries,
- created an invalid_mask with ~parcels_gdf.is_valid,
- preserved the original data with .copy(),
- repaired only the geometry column,
- re-ran the validation check after cleaning,
- reported any geometries that still need manual review.

```Python
import geopandas as gpd

Assumes parcels_gdf is already loaded
# 1. Audit the source data
invalid_mask = ~parcels_gdf.is_valid invalid_count = invalid_mask.sum()
print("Data Quality Audit:") print(parcels_gdf.is_valid.value_counts()) print(f"Found {invalid_count} invalid geometries out of {len(parcels_gdf)} total parcels.")

# 2. Preserve the original and repair the geometry column
parcels_clean_gdf = parcels_gdf.copy() parcels_clean_gdf["geometry"] = parcels_clean_gdf.geometry.make_valid()

# 3. Verify the repaired output
still_invalid = (~parcels_clean_gdf.is_valid).sum()
print(" Post-Cleaning Audit:") print(parcels_clean_gdf.is_valid.value_counts()) print(parcels_clean_gdf.geom_type.value_counts())
if still_invalid == 0: print(" Cleaning complete: all geometries are valid.") else: print(f" {still_invalid} geometries still need manual review.")

```

## The Pattern
Audit → isolate → copy → repair → verify

In plain English:

Check the incoming data before analysis. Keep the original dataset for comparison. Repair geometry problems in a clean copy. Then prove the output is valid before trusting it.

This is not a one-off cleanup step. It is a reusable data-quality pattern for any GeoPandas workflow.

## AI Pairing Prompt

I have a GeoDataFrame named `parcels_gdf`.

Write a GeoPandas data-quality script that:
1. checks every geometry with .is_valid,
2. reports the count of valid and invalid features,
3. saves invalid features to invalid_parcels_gdf for review,
4. creates a copied GeoDataFrame named parcels_clean_gdf,
5. repairs only its geometry column using .geometry.make_valid(),
6. verifies that the cleaned data is valid,
7. compares geometry types before and after repair, and
8. clearly reports any features that still need manual inspection.

Do not overwrite the original GeoDataFrame. Explain each step briefly.


## Why This Matters

Spatial analysis can fail quietly when source geometries are broken. Invalid polygons can create incorrect overlay results, failed exports, topology errors, or misleading area calculations.

A professional GIS workflow does not assume the data is clean. It validates first, repairs deliberately, and verifies the result.

This pattern now exists in both parts of my toolkit:
- PostGIS for database workflows.
- GeoPandas for Python scripts and notebooks.


## Next Steps

Apply this validation script to a real public dataset for a portfolio project.

A strong next artifact would be:

1. load a parcel, zoning, stormwater, or boundary dataset,
2. identify invalid features,
3. export the invalid subset to GeoJSON,
4. build a small Leaflet map that styles invalid features in red,
5. publish the map with GitHub Pages,
6. document the before-and-after data-quality results in the project README.


