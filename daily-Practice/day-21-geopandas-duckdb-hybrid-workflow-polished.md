
# Day 20 - Running SQL on a GeoDataFrame with DuckDB

## Focus

Using DuckDB to query a GeoPandas-loaded dataset 

**Technical win:** 
- Built a clean local raw/ data workflow that loads reliably on Windows, 
- runs a grouped SQL aggregation, and exits cleanly with Python exit code: 0.

## What I practiced

Today I practiced a hybrid workflow that combines GeoPandas for data loading with DuckDB for SQL-based aggregation.

**The goal was simple:**

- load the Natural Earth countries dataset with GeoPandas
- remove the geometry column for a non-spatial summary
- register the table in DuckDB
- write SQL to group countries by continent
- return the result as a pandas DataFrame

This is a useful modern GIS pattern because it lets Python handle file access and geospatial data structures, while SQL handles the filtering, grouping, and aggregation.

## Why this pattern matters

This is not just a Python script. 

It is a practical workflow pattern.

GeoPandas is good at reading spatial files and managing geometry

DuckDB is good at clear, readable SQL on tabular data

pandas is the clean handoff format for inspecting results

That separation matters. Each tool is doing the job it is best at.

**Plain-English version:**

Load the countries data, treat the attribute table like a SQL table, group by continent, sum population, and sort from largest to smallest.
That is the whole move.

## Working script

``` Python
from pathlib import Path
from urllib.request import urlretrieve
import geopandas as gpd
import duckdb

url = "https://naciscdn.org/naturalearth/110m/cultural/ne_110m_admin_0_countries.zip"
raw_folder = Path("raw") raw_folder.mkdir(exist_ok=True)
zip_path = raw_folder / "ne_110m_admin_0_countries.zip"

if not zip_path.exists(): 
    print("starting download...") 
	urlretrieve(url, zip_path) 
	print("download complete") 
else: 
    print("using local zip")

print("reading local file...") 
world = gpd.read_file(f"zip://{zip_path.as_posix()}") 
print("read complete")
world_table = world.drop(columns="geometry") 
on = duckdb.connect(database=":memory:")
try: 
    con.register("world", world_table)
	continent_population = con.sql(""" SELECT CONTINENT, SUM(POP_EST) AS total_population FROM world GROUP BY CONTINENT ORDER BY total_population DESC """).df()
    print(continent_population.head()) 
	print("EOF")
finally: 
    con.unregister("world") 
con.close()
```

## The SQL, explained like a sentence

```SQL

SELECT
 CONTINENT,
 SUM(POP_EST) AS total_population
FROM world
GROUP BY CONTINENT
ORDER BY total_population DESC

```

**Read it like this:**
Select each continent, add up the estimated population for that continent, group the rows by continent, then sort from the largest total population to the smallest.

That translation step is worth practicing. When I can describe the query clearly in plain English, the syntax gets easier to write and verify.

## Output

 CONTINENT total_population
0 Asia 4.550277e+09
1 Africa 1.306370e+09
2 Europe 7.454125e+08
3 North America 5.837560e+08
4 South America 4.270667e+08
EOF
Python exit code: 0

## What went wrong, and what fixed it

The first version of this workflow used:

```python

gpd.read_file(url)

```

That did load the data, but on my Windows setup the process would finish the work and still hang before returning to the PowerShell prompt.

The fix was to separate data acquisition from analysis:

1. download the ZIP once into a local raw/ folder
2. read the local ZIP with GeoPandas
3. run the DuckDB query against that local table

That change gave me a clean result and a clean exit.

This is a good reminder that reliability is part of implementation. A script is not really done until it runs cleanly from start to finish.

## Pattern to reuse

I can reuse this same pattern any time I want SQL clarity inside a Python spatial workflow.

**Examples:""
count features by category
- sum values by region
- filter records with SQL instead of pandas boolean indexing
- compute grouped summaries before joining results back to a spatial layer

**General structure:**
- gdf = gpd.read_file("some_source")
- table = gdf.drop(columns="geometry")
- con = duckdb.connect(database=":memory:") con.register("my_table", table)
- result = con.sql("SELECT ...").df()

That is a strong pattern for hybrid geospatial analysis.

## AI pairing pattern

A good prompt for this kind of workflow is:
- Write a Python script that uses GeoPandas to load a spatial dataset, 
- drops the geometry column for a non-spatial analysis, 
- registers the result as a DuckDB table, and 
- runs a SQL query to aggregate results by a named attribute. 
- Return the final result as a pandas DataFrame and 
- print the first five rows.

**Why this works:**

- it assigns each tool a clear role
- it defines the workflow in order
- it specifies the output format
- it reduces ambiguity for the AI

That is the bigger lesson. Describe the spatial process clearly, then let AI help write or refine the syntax.

## Reflection

This was a good practice rep because it connected tools instead of isolating them.

Day 19 focused on GeoPandas geometry operations with overlay and intersection. 

Day 20 focused on using SQL inside a Python spatial workflow. That is a real skill move. Not just learning one function, but learning which tool should do which job.

Today’s win was specific:
- I loaded spatial data with GeoPandas
- converted it into a clean tabular handoff
- used DuckDB to write a readable aggregation query
- fixed a local execution issue by moving the source data into raw/
- got a clean return to the terminal prompt

That is implementation. That is the kind of pattern worth keeping in a portfolio.

## Next variation to practice

A clean next rep would be filtering large-population countries before sorting:

```sql
SELECT
 NAME,
 CONTINENT,
 POP_EST
FROM world
WHERE POP_EST > 200000000
ORDER BY POP_EST DESC

```

That would keep the same GeoPandas → DuckDB pattern while adding a WHERE clause and returning named records instead of grouped totals.

## Takeaway
Today’s main lesson was not just how to write SUM(POP_EST).

**It was learning a repeatable hybrid pattern:**
- store source data in raw/
- load with GeoPandas
- drop geometry when the task is non-spatial
- query with DuckDB
- return a pandas DataFrame
- make sure the script exits cleanly

That is a practical workflow I can use again.