## Day 2 – August 8, 2026 – Filtering Parcels by Zoning (SQL) & Open Parks (GeoPandas)
Summary: Focused on two core filtering patterns: 
- selecting parcels by zoning in SQL (DuckDB) and 
- filtering open parks in Python (GeoPandas). 

Practiced translating spatial questions into code and checked results for accuracy.

## Key Code Snippets:

```
-- SQL: Filter parcels by zoning
SELECT * FROM parcels WHERE zoning = 'Residential';
```

```
# GeoPandas: Filter open parks
gdf_open = parks[parks['status'] == 'Open']
```

## Reflection: 
- Felt confident writing both queries from scratch. 
- The pattern of describing the spatial question first, then writing the code, is helping. 
- Next, I want to practice combining multiple conditions and handling missing values.

## Prompt Pattern: 
Described the filtering task to AI and used its suggestions to refine my queries. 

Noticed that pairing with AI speeds up troubleshooting and helps catch small syntax errors.