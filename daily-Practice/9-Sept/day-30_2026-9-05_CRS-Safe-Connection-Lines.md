# Day 30: CRS-Safe Connection Lines

## Focus

Make the park-access analysis visible without mixing measurement CRS requirements with web-map requirements. The workflow used Marion County subdivisions and parks, measured nearest-park relationships in EPSG:3086, then exported browser-ready copies in EPSG:4326 for Leaflet.

## What I Learned

A nearest-feature result is not enough by itself. The workflow must preserve the relationship between each subdivision and its matched park so the park geometry can be attached safely.

The core pattern was:

- Reproject subdivisions and parks to EPSG:3086 before measuring distance or building lines.
- Use `gpd.sjoin_nearest(..., distance_col="distance_m")` to match each subdivision to its nearest park.
- Preserve `index_right`, which identifies the matched park row.
- Use that index to attach the matched park geometry to the subdivision result.
- Select the ten largest `distance_m` values.
- Build centroid-to-park `LineString` geometries in EPSG:3086.
- Export separate copies to EPSG:4326 for Leaflet.

The browser workflow also mattered. I used `python -m http.server 8000` to serve the map locally instead of double-clicking `index.html`, because browser security rules can block local data requests when a file is opened directly.

## What I Practiced

I completed the knowledge check with 100% first-try success.

I wrote an incomplete implementation of the park-access workflow. It needed corrections and a reusable function before it was dependable, especially around preserving the nearest-park match, attaching the correct park geometry, and keeping projected and web layers separate.

I also wrote an AI review prompt. It earned 40% AI Prompt Mastery because it referenced `test.py` without providing enough technical detail for a useful review. The result made the gap clear: a good pairing prompt should identify the expected inputs, CRS assumptions, join fields, output geometry, and acceptance checks.

The first working map included subdivisions, parks, and ten red connection links. This was a working visual checkpoint, not a claim about final map results.

## The Pattern

```python
import geopandas as gpd
from shapely.geometry import LineString

analysis_crs = "EPSG:3086"

subdivisions_m = subdivisions.to_crs(analysis_crs)
parks_m = parks.to_crs(analysis_crs)

nearest = gpd.sjoin_nearest(
    subdivisions_m,
    parks_m[["geometry"]],
    how="left",
    distance_col="distance_m",
)

matched_parks = parks_m.loc[nearest["index_right"]].copy()
matched_parks.index = nearest.index

nearest["park_geometry"] = matched_parks.geometry
farthest = nearest.nlargest(10, "distance_m").copy()

farthest["geometry"] = [
    LineString([subdivision.centroid, park])
    for subdivision, park in zip(
        farthest.geometry,
        farthest["park_geometry"],
    )
]

links_m = gpd.GeoDataFrame(
    farthest,
    geometry="geometry",
    crs=analysis_crs,
)

# Keep analysis layers in EPSG:3086.
# Export separate copies for Leaflet.
links_web = links_m.to_crs("EPSG:4326")
```

The important boundary is deliberate: use EPSG:3086 for distance and line construction, then use EPSG:4326 only for web delivery.

## AI Pairing Prompt

```text
Review my GeoPandas park-access workflow for correctness and reusability.

Context:
- Input layers are Marion County subdivisions and parks.
- Both layers must be reprojected to EPSG:3086 before nearest-distance analysis.
- I use gpd.sjoin_nearest with distance_col="distance_m".
- The result must preserve index_right so each subdivision can retrieve its matched park geometry.
- I need the ten largest distance_m values.
- For those ten records, build LineString geometries from each subdivision centroid to its matched park geometry.
- Keep analysis and line construction in EPSG:3086.
- Export separate EPSG:4326 copies for Leaflet.
- The local map is served with python -m http.server 8000.

Please:
1. Identify bugs or fragile assumptions in my implementation.
2. Refactor the workflow into a reusable function with clear inputs and outputs.
3. Check that index_right is used safely and that the matched park geometry aligns with the correct subdivision.
4. Check CRS handling, distance units, missing matches, duplicate indexes, and empty inputs.
5. Confirm that the output link layer has LineString geometry and the expected CRS.
6. Suggest focused tests for the top-ten selection and coordinate conversion.

Do not assume that the map is final. Review the code and data-flow contracts first.
```

## Why This Matters

Spatial analysis becomes easier to trust when the measurement layer, relationship keys, geometry construction, and browser layer each have a clear job.

A shared projected CRS prevents distance and line construction from silently using inappropriate units. Preserving `index_right` prevents a common join mistake: drawing a line to the wrong park even though the map still renders. Separate EPSG:4326 exports keep Leaflet compatible without weakening the analysis layer.

The practice results also matter. The knowledge check showed strong conceptual understanding. The incomplete implementation and 40% prompt score identified the next practical target: make the code reusable and make AI requests specific enough to review real spatial logic.

## Next Steps

- Refactor the implementation into a reusable function for nearest-park links.
- Add checks for missing `index_right` values, duplicate indexes, empty layers, and CRS mismatches.
- Verify that the ten selected links are the ten largest `distance_m` values.
- Confirm that all constructed links are valid `LineString` geometries in EPSG:3086.
- Export clearly named EPSG:4326 copies for the Leaflet map.
- Improve the AI prompt by including the relevant function or code excerpt instead of only naming `test.py`.
- Continue using `python -m http.server 8000` when testing the local web map.