# Day X – MM DD, 2026 – Filtering Parcels by Zoning (SQL) & Open Parks (GeoPandas)

Summary: Today I focused on two core filtering patterns: selecting parcels by zoning in SQL (DuckDB) and filtering open parks in Python (GeoPandas). I practiced translating spatial questions into code and checked my results for accuracy. I’m starting to see how these patterns show up in real spatial analysis.

## Key Code Snippets:
```
-- SQL: Filter parcels by zoning
SELECT * FROM parcels WHERE zoning = 'Residential';
```
```
# GeoPandas: Filter open parks
gdf_open = parks[parks['status'] == 'Open']
```
Optional data path
```
# config.py
DATA_PATH = r"C:/tmp"

```

## Reflection: 
- 

## Prompt Pattern:
- 