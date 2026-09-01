## Day 1 – August 7, 2026 – Kickoff: Spatial SQL & Python Mastery

Summary: Started my journey to master Spatial SQL (DuckDB, PostGIS) and Python for spatial analysis (GeoPandas, Rasterio). Today’s focus was on setup, understanding the core philosophy (implementation over theory), and running my first basic queries in both SQL and Python.

## Key Code Snippets:
```
-- SQL: Select all parcels
SELECT * FROM parcels;
```

```
# GeoPandas: Load and preview parcels
import geopandas as gpd
parcels = gpd.read_file('parcels.geojson')
parcels.head()

```

## Reflection: 
- Excited to start this hands-on journey. 
- The biggest challenge is moving from point-and-click GIS to code-first workflows, but I’m ready to build real fluency. 
- Looking forward to building daily momentum and sharing progress here.

## Workspace: 
Set up my project folders and installed DuckDB, GeoPandas, and JupyterLab.