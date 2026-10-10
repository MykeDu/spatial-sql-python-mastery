# Day 33: Filter With an Audit Trail

## Focus

Make a data filter defensible by preserving the source, recording uncertainty, and separating confirmed results from records that need review.

Today's pattern is transparent residential classification:

- `1` = confirmed residential
- `0` = confirmed non-residential
- `NULL` = needs review

The related project work continues the proof map debugging from yesterday: inspect the actual HTTP response and file paths before changing map code.

## What I Learned

A filter is not just a yes-or-no operation. It is also a decision record.

Using three states keeps uncertainty visible:

```python
gdf["is_residential"].value_counts(dropna=False)
```

This shows the counts for confirmed residential, confirmed non-residential, and unresolved records. The `dropna=False` argument matters because otherwise the review queue can disappear from the summary.

I also learned to preserve the source GeoDataFrame. The original data should remain available for auditing, comparison, and later corrections. Analysis transformations should happen on a copied subset.

For the spatial workflow:

- Use `EPSG:3086` for distance analysis and nearest-neighbor ranking.
- Use `EPSG:4326` only when exporting GeoJSON for Leaflet.
- Do not reproject the source object just to prepare a map export.

## What I Practiced

For the Residential Park-Access Filter, I first inspected the classification:

```python
gdf["is_residential"].value_counts(dropna=False)
```

Then I created a separate confirmed-residential subset:

```python
residential = gdf.loc[
    gdf["is_residential"].eq(1)
].copy()

review_queue = gdf.loc[
    gdf["is_residential"].isna()
].copy()
```

The `.copy()` call makes the workflow explicit and avoids accidentally modifying a view of the source data. The `review_queue` remains separate so unresolved records are not silently treated as either residential or non-residential.

For the spatial ranking step, the copied subset stays in the analysis CRS:

```python
residential_3086 = residential.to_crs("EPSG:3086")

nearest = residential_3086.sjoin_nearest(
    parks_3086,
    distance_col="distance_m"
)
```

Only the final browser export should use WGS84:

```python
residential_4326 = residential_3086.to_crs("EPSG:4326")
residential_4326.to_file(
    "web/data/residential.geojson",
    driver="GeoJSON"
)
```

For the proof map, I diagnosed the `JSON.parse` error by checking the response before changing Leaflet code:

```javascript
const response = await fetch("data/subdivisions.geojson");

if (!response.ok) {
  throw new Error(`HTTP ${response.status}: ${response.statusText}`);
}

const body = await response.text();
console.log(body.slice(0, 200));

const subdivisions = JSON.parse(body);
```

The local server must run from the web folder, and the requested paths must match the actual files:

```text
data/subdivisions.geojson
data/parks.geojson
data/links.geojson
```

An HTML 404 response is not valid GeoJSON. The response body and Network tab reveal that problem faster than rewriting the map.

## The Pattern

```text
Preserve the source GeoDataFrame
 |
 v
Classify as 1, 0, or NULL
 |
 v
Inspect value_counts(dropna=False)
 |
 v
Copy confirmed residential records
 |
 v
Analyze and rank in EPSG:3086
 |
 v
Export Leaflet data in EPSG:4326
 |
 v
Check HTTP status, response body, and file paths before changing map code
```

In plain English:

> Make every filtering decision visible, keep uncertain records in a review queue, and separate analysis CRS from display CRS.

## AI Pairing Prompt

```text
Review this residential park-access workflow.

Requirements:
- is_residential = 1 means confirmed residential.
- is_residential = 0 means confirmed non-residential.
- is_residential = NULL means needs review.
- Preserve the source GeoDataFrame.
- Create a copied confirmed-residential subset before reprojection.
- Use EPSG:3086 for analysis and nearest-neighbor ranking.
- Use EPSG:4326 only for Leaflet GeoJSON exports.
- Keep the NULL records in a separate review queue.

First, check whether the audit trail is preserved.
Then identify any step that could silently drop or misclassify records.
Do not rewrite the whole workflow.
```

For the map debugging:

```text
The browser reports JSON.parse: unexpected character at line 1 column 1.

Help me diagnose the narrowest likely cause.
Ask me to check:
- response.ok and the HTTP status
- response.text() before response.json()
- the browser Network response body
- the local server working directory
- these exact paths:
  data/subdivisions.geojson
  data/parks.geojson
  data/links.geojson

Do not change the Leaflet map code until the response and path are verified.
```

## Why This Matters

An audit trail makes the result explainable.

If a record is included, the workflow can show that it was confirmed residential. If it is excluded, the workflow can distinguish confirmed non-residential from unresolved data. That difference matters when the output informs a map, a neighborhood comparison, or a portfolio project that someone else may review.

The CRS separation also prevents a common spatial mistake: using longitude and latitude as if they were meter-based analysis coordinates. Projected analysis in `EPSG:3086` keeps distance calculations meaningful, while `EPSG:4326` keeps the browser export compatible with Leaflet.

Finally, checking the real HTTP response makes debugging evidence-based. A JSON parse error often means the browser received the wrong file, an HTML error page, or a request from the wrong server directory.

## Next Steps

- Review every `NULL` record and document the classification decision.
- Compare the confirmed-residential count with the full source count.
- Complete nearest-neighbor ranking in `EPSG:3086`.
- Export only browser-facing GeoJSON in `EPSG:4326`.
- Run the local server from the web folder.
- Verify the three GeoJSON paths in the Network tab.
- Save the corrected proof map and its audit notes.