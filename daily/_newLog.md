# Day X – Filtering Parcels by Zoning (SQL) & Open Parks (GeoPandas)
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
- Writing the SQL query felt pretty natural today, and 
- I was able to get the GeoPandas filter right on the first try. 
- Describing the task in plain English before coding helped me avoid mistakes. 
- I want to get even faster at combining multiple conditions and handling missing values in both tools.

## Prompt Pattern: 
- I described my filtering task to AI and used its suggestions to double-check my syntax. 
- Pairing with AI made troubleshooting smoother and gave me confidence in my approach.