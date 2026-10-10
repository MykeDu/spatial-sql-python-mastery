# Day 45: Inspect the Source Schema Before You Map

## Focus

Make the inputs trustworthy before spatial analysis. Today I reviewed the spatial input contract and inspected the actual source schemas so the project can lock down field mappings, identifiers, CRS, geometry, and basic data quality checks at the boundary.

## What I Learned

A spatial input contract is more than a list of expected column names. It documents how each source field becomes a stable field in the analysis workflow.

For the park source:

- `FACILITYID` is the source technical identifier and maps to `park_id`.
- The park display name should be mapped to the project name field used by downstream analysis.
- The source CRS must be known before representative points or distances are calculated.

For the subdivision source:

- `OBJECTID` may be used as a documented temporary snapshot key.
- `SUB_NAME` is a display candidate, not automatically a stable identifier.
- `SUB_NAME` can become an identifier only after uniqueness and null checks pass.

The analysis CRS stays EPSG:2236. That is the working CRS for representative points and nearest-park distance calculations. EPSG:4326 is for browser inspection and finished Leaflet output only. The existing Leaflet proof map remains unchanged while the input contract is inspected and locked.

## What I Practiced

I practiced a schema inspection helper that checks the evidence needed before writing adapters:

1. CRS and row count.
2. Column names and geometry types.
3. Null counts for candidate identifiers.
4. Duplicate counts for candidate identifiers.
5. Example values for quick source inspection.
6. A clear failure when a required field is unresolved.

I also completed the review with 3/3 first-try correct. The project build was to inspect the real source schemas, document the mappings, and stop rather than guess if a required field could not be resolved.

```python
import geopandas as gpd


def inspect_spatial_input(
    layer: gpd.GeoDataFrame,
    name: str,
    candidate_id_fields: tuple[str, ...] = (),
    required_fields: tuple[str, ...] = (),
) -> None:
    print(f"\n{name}")
    print("-" * len(name))
    print(f"CRS: {layer.crs}")
    print(f"Rows: {len(layer):,}")
    print(f"Columns: {list(layer.columns)}")

    if layer.crs is None:
        raise ValueError(f"{name} has no CRS")

    missing = set(required_fields) - set(layer.columns)
    if missing:
        raise ValueError(
            f"{name} is missing required fields: {sorted(missing)}"
        )

    if "geometry" in layer:
        geometry_types = (
            layer.geometry.geom_type
            .value_counts(dropna=False)
            .to_dict()
        )
        print(f"Geometry types: {geometry_types}")

    for field in candidate_id_fields:
        if field not in layer.columns:
            print(f"{field}: not present")
            continue

        values = layer[field]
        print(f"{field} nulls: {int(values.isna().sum()):,}")
        print(f"{field} duplicates: {int(values.duplicated().sum()):,}")
        print(f"{field} examples: {values.dropna().head(5).tolist()}")


inspect_spatial_input(
    parks_source,
    name="Parks",
    candidate_id_fields=("FACILITYID",),
    required_fields=("FACILITYID", "geometry"),
)

inspect_spatial_input(
    subdivisions_source,
    name="Subdivisions",
    candidate_id_fields=("OBJECTID", "SUB_NAME"),
    required_fields=("OBJECTID", "SUB_NAME", "geometry"),
)
```

The output is evidence for the adapter, not a substitute for the adapter. If `FACILITYID` is missing, or if the subdivision key is unresolved, the workflow should stop and ask for a mapping decision.

## The Pattern

Inspect first, map second, analyze third.

The input contract should document both the source field and the normalized field:

```python
field_map = {
    "parks": {
        "FACILITYID": "park_id",
    },
    "subdivisions": {
        "OBJECTID": "subdivision_snapshot_id",
        "SUB_NAME": "subdivision_name",
    },
}
```

`OBJECTID` is explicitly a temporary snapshot key because it may be unique only within the current source extract. `SUB_NAME` is retained as a display candidate until its uniqueness is verified. The adapter can then return a predictable schema in EPSG:2236:

```python
parks = (
    parks_source
    .rename(columns={"FACILITYID": "park_id"})
    [["park_id", "geometry"]]
    .to_crs("EPSG:2236")
)

subdivisions = (
    subdivisions_source
    .rename(
        columns={
            "OBJECTID": "subdivision_snapshot_id",
            "SUB_NAME": "subdivision_name",
        }
    )
    [["subdivision_snapshot_id", "subdivision_name", "geometry"]]
    .to_crs("EPSG:2236")
)
```

The key rule is to preserve the working CRS during analysis. Convert a validated final copy to EPSG:4326 only for browser inspection or Leaflet export.

## AI Pairing Prompt

> Inspect these GeoPandas source layers and help me lock the input contract. Report the CRS, row count, columns, geometry types, nulls, duplicates, and example values for FACILITYID, OBJECTID, and SUB_NAME. Then propose the smallest documented field mapping to park_id, subdivision_snapshot_id, and subdivision_name. Do not invent a missing field. If a required field or stable identifier is unresolved, stop and explain exactly what decision is needed. Keep EPSG:2236 for analysis and leave the existing Leaflet proof map unchanged.

## Why This Matters

A trustworthy spatial workflow starts before the first spatial join. If identifiers, CRS, or geometry assumptions are wrong, a technically valid nearest-park calculation can still produce misleading results. Inspecting and locking the contract once keeps source-specific names out of later stages, makes failures visible, and protects the working map from silent changes.

## Next Steps

- Record the confirmed source-to-project field mappings.
- Decide whether `OBJECTID` is acceptable as a temporary snapshot key for this extract.
- Check whether `SUB_NAME` is unique before treating it as more than a display field.
- Add the inspection gate before the source adapters run.
- Keep representative-point and nearest-park analysis in EPSG:2236.
- Convert only the finished Leaflet copy to EPSG:4326.
- Preserve the existing Leaflet proof map while validating the new input contract.