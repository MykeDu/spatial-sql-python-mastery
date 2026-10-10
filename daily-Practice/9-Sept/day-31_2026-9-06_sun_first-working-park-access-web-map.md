# Day 31: First Working Park-Access Web Map

## Focus

Today I reviewed the handoff from the analysis CRS, EPSG:3086, to the web export CRS, EPSG:4326. The main goal was to keep each job in the CRS where it makes sense, then publish a clean set of files that a browser can use.

The analysis stays in EPSG:3086 for measurement, ranking, and line construction. The web copies move to EPSG:4326 for Leaflet.

## What I Learned

A CRS boundary is part of the workflow, not a cleanup step at the end.

EPSG:3086 is the right place for distance-based work because the units support meaningful measurement. That is where I rank the farthest nearest-park distances and construct the connection lines.

EPSG:4326 is the right export format for the browser map. Leaflet expects longitude and latitude coordinates, so the web copies need to be reprojected before they are written to `web/data`.

I also reviewed how `index_right` from `sjoin_nearest` points back to the matched park geometry. That index lets me retrieve the correct park feature before building the connection line.

## What I Practiced

I reviewed the export workflow and verified that these files are written into `web/data`:

- `subdivisions.geojson`
- `parks.geojson`
- `farthest_links.geojson`

I checked that every web copy is EPSG:4326 and confirmed that `farthest_links.geojson` contains 10 farthest links.

I also rendered a minimal Leaflet map using:

- `fetch`
- `L.geoJSON`
- `fitBounds`
- layer controls

The final step was to document the first browser result, whether the map rendered correctly or returned an error that needed investigation.

## The Pattern

The pattern is to separate analytical geometry from display geometry.

1. Load and prepare the data in a projected CRS.
2. Use EPSG:3086 for measurement, ranking, nearest-neighbor results, and line construction.
3. Use `index_right` from `sjoin_nearest` to retrieve each matched park geometry.
4. Build the farthest connection lines while the data is still in EPSG:3086.
5. Reproject copies to EPSG:4326 only for web delivery.
6. Export the GeoJSON files into `web/data`.
7. Verify the CRS and feature counts before opening the browser.
8. Render the layers with Leaflet and record the first result.

This keeps the analysis honest and gives the browser the coordinate format it needs.

## AI Pairing Prompt

Review the boundary between my EPSG:3086 analysis and EPSG:4326 web export. Check that measurement, ranking, nearest-park geometry retrieval through `index_right`, and line construction happen in EPSG:3086; check that the three GeoJSON files in `web/data` are exported in EPSG:4326; verify that there are 10 farthest links; and review the minimal Leaflet browser QA using `fetch`, `L.geoJSON`, `fitBounds`, and layer controls. Do not rebuild the nearest-neighbor analysis. Focus on export correctness, CRS handling, file paths, feature counts, and the first browser result.

## Why This Matters

This is where the analysis became a visible portfolio artifact.

The work is no longer only a script or a table of results. It now has a clear handoff from reproducible spatial analysis to a browser map that someone else can inspect. That handoff makes the project easier to explain, easier to debug, and easier to share.

It also reinforces a practical rule: use the CRS that supports the job at hand, and make the conversion to web coordinates explicit.

## Next Steps

- Keep the browser QA notes with the project files.
- Add a short README section explaining the CRS boundary.
- Capture a screenshot or link for the first working map.
- Continue improving the map without changing the validated nearest-park analysis.