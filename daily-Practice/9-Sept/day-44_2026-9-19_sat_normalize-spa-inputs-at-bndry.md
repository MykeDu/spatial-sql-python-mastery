# Day 44 - Saturday, September 19, 2026

## Focus

Normalize spatial inputs at the boundary so the rest of the workflow can use a small, stable data contract. The goal is to make the workflow readable: inspect real schemas, map source fields once, validate CRS and required fields, and reject bad inputs before spatial work begins.

## What I Learned

A source adapter is the Python-first preparation boundary between messy source files and reusable analysis stages.

For the park layer, the adapter should:

- Require `FACILITYID`, `PARK_NAME`, and `geometry`.
- Reject a missing CRS before analysis.
- Map `FACILITYID` to `park_id`.
- Map `PARK_NAME` to `park_name`.
- Copy the data and return only the standardized fields.
- Return the prepared layer in EPSG:2236.

For subdivisions, the adapter should accept `source_id_field` and `source_name_field`, validate missing IDs and duplicate IDs, and return standardized fields in EPSG:2236.

EPSG:2236 remains the analysis CRS for representative-point work and nearest-park distance calculations. EPSG:4326 should be used only after validated outputs are ready for the Leaflet map.

## What I Practiced

I practiced building a source adapter for the park-access workflow:

1. Print the source columns and CRS before changing anything.
2. Identify the actual source fields.
3. Map `FACILITYID` and `PARK_NAME` to `park_id` and `park_name`.
4. Validate required fields, geometry, CRS, missing IDs, and duplicate IDs.
5. Return a clean copy with the expected schema in EPSG:2236.
6. Keep representative-point creation and `sjoin_nearest` out of the adapter.
7. Preserve the existing Leaflet proof map and representative-point metric.

The adapter prepares inputs. Later stages perform analysis, create representative points, find nearest parks, and export validated results for web mapping.

## The Pattern

```python
import geopandas as gpd


def prepare_parks(source: gpd.GeoDataFrame) -> gpd.GeoDataFrame:
    required = {"FACILITYID", "PARK_NAME", "geometry"}
    missing = required - set(source.columns)
    if missing:
        raise ValueError(f"Missing park fields: {sorted(missing)}")

    if source.crs is None:
        raise ValueError("Park source must have a CRS")

    parks = source.rename(
        columns={
            "FACILITYID": "park_id",
            "PARK_NAME": "park_name",
        }
    ).copy()

    parks = parks[["park_id", "park_name", "geometry"]]

    if parks["park_id"].isna().any():
        raise ValueError("Park IDs cannot be missing")

    return parks.to_crs("EPSG:2236")


def prepare_subdivisions(
    source: gpd.GeoDataFrame,
    source_id_field: str,
    source_name_field: str,
) -> gpd.GeoDataFrame:
    required = {source_id_field, source_name_field, "geometry"}
    missing = required - set(source.columns)
    if missing:
        raise ValueError(f"Missing subdivision fields: {sorted(missing)}")

    if source.crs is None:
        raise ValueError("Subdivision source must have a CRS")

    subdivisions = source.rename(
        columns={
            source_id_field: "subdivision_id",
            source_name_field: "subdivision_name",
        }
    ).copy()

    if subdivisions["subdivision_id"].isna().any():
        raise ValueError("Subdivision IDs cannot be missing")

    if subdivisions["subdivision_id"].duplicated().any():
        raise ValueError("Subdivision IDs must be unique")

    subdivisions = subdivisions[
        ["subdivision_id", "subdivision_name", "geometry"]
    ]

    return subdivisions.to_crs("EPSG:2236")
```

The adapter establishes names, types, and CRS. It does not create representative points, call `sjoin_nearest`, calculate distances, or export Leaflet layers.

## AI Pairing Prompt

> Diagnose the smallest schema-mapping fix. First identify the actual source fields and the fields the downstream workflow expects. If the source uses `FACILITYID` and `PARK_NAME` while the workflow expects `park_id` and `park_name`, propose only the adapter rename and validation changes needed. Preserve the existing representative-point, nearest-neighbor, and Leaflet stages. Do not rewrite the whole workflow; diagnose before changing code.

## Why This Matters

Source-specific field names should be handled once, at the boundary. That keeps downstream functions readable and makes failures local and understandable.

CRS validation is equally important. A missing or incorrect CRS can make later distance results meaningless. Keeping EPSG:2236 for analysis preserves the intended units and measurement behavior, while using EPSG:4326 only for validated web outputs keeps the Leaflet map compatible without mixing display concerns into analysis logic.

Separating preparation from spatial work also protects the existing representative-point metric. The workflow can continue to answer the same question - distance from a subdivision representative point to its nearest park - without hiding that metric inside an input-cleaning function.

## Next Steps

- Finish the park and subdivision source adapters.
- Print source columns and CRS before preparation.
- Identify and document the stable subdivision ID field.
- Add focused tests for missing fields, missing CRS, missing IDs, and duplicate IDs.
- Keep representative-point creation and nearest-park work in later analysis stages.
- Preserve the existing Leaflet proof map as a validation artifact.
- Export EPSG:4326 only after the analysis outputs have passed validation.