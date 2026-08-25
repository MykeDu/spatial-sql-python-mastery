# Day 19 – August 25, 2026 – Daily Practice Log

## Focus

Learning the GeoPandas equivalent of ST_Intersection with gpd.overlay() Using how="intersection" to return the exact shared geometry between two layers Applying the filter-first pattern before an expensive spatial overlay Verifying that both GeoDataFrames use the same CRS before clipping

## Activities Planned

- Micro-lesson: Learn how gpd.overlay() performs vector overlay operations in GeoPandas
- Practice problem: Clip building footprints to the Downtown historic district
- Prompt drill: Ask AI to write an efficient clipping workflow that filters wetlands before running an overlay

## Key Learning Goals

- Use gpd.overlay() to combine two GeoDataFrames based on their shared geometry
- Use how="intersection" to return only the overlapping portions of features
- Filter the boundary or overlay layer before calling gpd.overlay()
- Confirm that both GeoDataFrames have matching CRS values before spatial operations
- Understand that assigning .crs only changes metadata, while .to_crs() transforms coordinates
- Inspect the output with .head() before saving or using it in later analysis

## Sample Prompt (Planned)

"Write a Python script using GeoPandas to clip building footprints to the historic district named 'Downtown'. Files: buildings.geojson and historic_districts.geojson. Read both files with gpd.read_file(). Confirm that both GeoDataFrames use the same CRS before the overlay. First filter the districts layer to name == 'Downtown', then use gpd.overlay() with how='intersection' to return only the portion of each building inside the district. Print the first five rows of the clipped result. Explain why filtering before the overlay is more efficient."

## Sample Script (Planned): GeoPandas Python

```python

import geopandas as gpd

# Load source layers
buildings_gdf = gpd.read_file("data/buildings.geojson") 
historic_districts_gdf = gpd.read_file("data/historic_districts.geojson")

# CRS labels must match before a spatial overlay.
# Use to_crs() if the coordinates actually need to be transformed.
assert buildings_gdf.crs == historic_districts_gdf.crs, "CRS mismatch"

# Filter cheaply first, then run the more expensive geometry operation.
downtown_gdf = historic_districts_gdf.loc[ historic_districts_gdf["name"] == "Downtown", ["name", "geometry"] ]

# GeoPandas equivalent of a spatial intersection clip.
clipped_buildings = gpd.overlay( buildings_gdf, downtown_gdf, how="intersection" )

# Quick validation before saving or using downstream.
print(clipped_buildings.head())

```

## Optional Performance Version: Filter Candidates Before Overlay

```python

import geopandas as gpd
buildings_gdf = gpd.read_file("data/buildings.geojson") 
historic_districts_gdf = gpd.read_file("data/historic_districts.geojson")
assert buildings_gdf.crs == historic_districts_gdf.crs, "CRS mismatch"
downtown_gdf = historic_districts_gdf.loc[ historic_districts_gdf["name"] == "Downtown", ["name", "geometry"] ]

# Identify likely candidates before computing exact clipped geometry.
candidate_buildings = gpd.sjoin( buildings_gdf, downtown_gdf, how="inner", predicate="intersects" )

# Keep only the original building geometry and run the exact overlay.
candidate_buildings = candidate_buildings.loc[:, buildings_gdf.columns]
clipped_buildings = gpd.overlay( candidate_buildings, downtown_gdf, how="intersection" )
print(clipped_buildings.head())

```

## Reflection (Planned)

- gpd.overlay(..., how="intersection") is the GeoPandas tool for returning the geometry shared by two layers.
- The SQL and Python pattern is the same: narrow the candidate features first, then calculate exact geometry only where needed.
- A matching CRS is required for a trustworthy overlay. If the layers are in different coordinate systems, I need .to_crs() rather than simply assigning a CRS label.
- gpd.sjoin(..., predicate="intersects") can identify candidate buildings, while gpd.overlay() creates the final clipped geometry.
- I will inspect the output before saving it, then prefer GeoParquet for larger analysis outputs rather than defaulting to Shapefiles.

## Code Comments

Plain-English read: load the buildings and historic districts, confirm they use the same coordinate system, select only the Downtown district, then return the piece of each building that falls inside that district.

The key pattern is still filter cheaply first, compute geometry second. In SQL, ST_Intersects narrows the candidates before ST_Intersection. In GeoPandas, filtering the target layer, and optionally using sjoin() for candidates, reduces the work before gpd.overlay(..., how="intersection") creates the clipped geometry.
