## Day 5 – August 11, 2026 – Daily Practice Log

### Focus:
- Mastering Coordinate Reference System (CRS) management in GeoPandas
- Understanding the difference between setting CRS metadata and performing coordinate transformations

### Activities Completed:
- Micro-lesson: CRS concepts in spatial analysis
- Knowledge check: Solved 3 CRS-related problems (100% first-try success)
- Practice: Performed CRS transformation on a sample dataset
- Prompt drill: Drafted an AI prompt for resolving GDAL-related transformation errors

### Key Takeaways:
- Setting a CRS only updates metadata; transforming coordinates actually changes the geometry values.
- Always check the CRS before performing spatial operations to avoid mismatches and errors.
- A clear, context-rich AI prompt leads to better troubleshooting and code generation.

### Sample Prompt Used:
"I’m working with a GeoPandas DataFrame in EPSG:4326 and need to reproject it to EPSG:3857. What’s the correct method, and how do I handle any GDAL errors that might come up?"

### Sample Code:

```
import geopandas as gpd
# Assume gdf is your GeoDataFrame in EPSG:4326
gdf_3857 = gdf.to_crs(epsg=3857)

```

### Reflection:
- Practicing with both code and prompt-writing clarified the difference between metadata and true transformation.
- Next, I’ll apply CRS transformations to local datasets and continue refining my AI prompting for technical troubleshooting.
