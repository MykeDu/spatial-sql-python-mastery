# Day 23: Nearest Neighbor Searches in GeoPandas

## Focus

Today’s Python pattern is the nearest neighbor join.
In SQL, you learned the pattern as: find the closest feature for each row.
In GeoPandas, the equivalent move is usually:
```
gpd.sjoin_nearest()
```
This is the clean Python version of the same spatial idea. 
You have one GeoDataFrame of source features, one GeoDataFrame of target features, and you want the nearest match for each source row.

**Real world mindset**
```
For each sinkhole, find the closest fire station and record the distance.

```

## Why This Matters

This is one of the core professional patterns in spatial work.
You will use it when you need to answer questions like:
- What is the nearest fire station to each sinkhole?
- What is the closest hospital to each incident?
- Which asset is nearest to each inspection point?

The important point is that the pattern stays the same across tools.
- In PostGIS, you may use ORDER BY <-> with a nearest-neighbor pattern.
- In GeoPandas, you will usually use sjoin_nearest().

Different syntax. Same spatial logic.


## The Common Mistake

A lot of people try to do this with a manual loop or apply().

That usually looks something like this:
```Python

sinkholes["nearest_station"] = sinkholes.geometry.apply(
 lambda geom: fire_stations.distance(geom).idxmin()
)

```

That works for tiny examples, but it is not the pattern you want to build into your workflow.

Why?
- It is harder to read.
- It is slower on larger data.
- It hides the real spatial pattern.

The better move is to use the built-in nearest join.


## The Correct Pattern

```Python

import geopandas as gpd
sinkholes = gpd.read_file("data/florida_sinkholes.geojson") 
fire_stations = gpd.read_file("data/florida_fire_stations.geojson")
sinkholes = sinkholes.to_crs("EPSG:26917") 
fire_stations = fire_stations.to_crs("EPSG:26917")
nearest = gpd.sjoin_nearest( sinkholes, fire_stations, how="left", distance_col="distance_m" )
print(nearest.head())

```

Read that code like a sentence:

Take each sinkhole, find the nearest fire station, keep every sinkhole, and store the distance in meters in a new column called distance_m.

That is the pattern.


## Key Implementation Note: CRS First

This is the same unit gotcha you have already seen in spatial work.

If your data is still in EPSG:4326, distance values are in degrees, not meters.

So before using nearest-neighbor distance logic, project both layers into a CRS that makes sense for the study area.

Example:
```Python

sinkholes = sinkholes.to_crs("EPSG:26917")
fire_stations = fire_stations.to_crs("EPSG:26917")

```

For practice, the rule is simple:
```
If distance matters, projection matters.
```


## Breaking Down sjoin_nearest()

Here is the same pattern again:

```Python

nearest = gpd.sjoin_nearest(
 sinkholes,
 fire_stations,
 how="left",
 distance_col="distance_m"
)

```


### What each part does

**sinkholes**

This is the left GeoDataFrame. These are the rows you want to keep.

**fire_stations**

This is the right GeoDataFrame. These are the features you want to search against.

**how="left"**

Keep every sinkhole row, even if a nearby match is missing.

**distance_col="distance_m"**

Write the computed nearest distance into a new column.

```
This is a good habit because it turns the join into something you can inspect, sort, filter, and publish.
```


## Minimal Working Example

```Python

import geopandas as gpd
from shapely.geometry import Point
sinkholes = gpd.GeoDataFrame( { "sinkhole_id": [1, 2, 3] }, geometry=[ Point(-82.4572, 27.9506), Point(-82.4800, 27.9700), Point(-82.4300, 27.9400) ], crs="EPSG:4326" )
fire_stations = gpd.GeoDataFrame( { "station_id": [101, 102], "station_name": ["Station A", "Station B"] }, geometry=[ Point(-82.4600, 27.9550), Point(-82.4200, 27.9350) ], crs="EPSG:4326" )
sinkholes = sinkholes.to_crs("EPSG:26917") 
fire_stations = fire_stations.to_crs("EPSG:26917")
nearest = gpd.sjoin_nearest( sinkholes, fire_stations, how="left", distance_col="distance_m" )
print(nearest[["sinkhole_id", "station_id", "station_name", "distance_m"]])

```

## What I Practiced Today

- Using gpd.sjoin_nearest() to solve a nearest neighbor problem
- Translating a known SQL spatial pattern into Python
- Projecting data before measuring distance
- Keeping all source rows with how="left"
- Writing distance results into a reusable field


## Portfolio Connection
This is a strong pattern for your Florida sinkhole and fire-station project.

A clean portfolio version of this analysis could answer:
- Which fire station is closest to each sinkhole?
- How far is each sinkhole from its nearest station?
- Which sinkholes are relatively far from emergency response coverage?

That turns a practice exercise into a real analysis artifact.


## AI Pairing Prompt

Use a prompt like this when you want AI help without losing the spatial logic:

```Prompt

I have two GeoDataFrames in Python:

1. sinkholes: point features with sinkhole_id
2. fire_stations: point features with station_id and station_name

Write a GeoPandas solution using sjoin_nearest to find the nearest fire station for each sinkhole. 
Reproject both layers to an appropriate projected CRS first so distance is measured in meters. 
Keep all sinkholes in the result and store the distance in a column called distance_m. 
Then explain the code in plain English.

```

This is a good prompt because it tells the AI:

- the inputs
- the spatial task
- the CRS requirement
- the output shape
- the naming you want


## Practice Problem

I used an old Florida project to frame this example.

**Task**

Wrote Python code that:

1. Reads a sinkhole dataset
2. Reads a fire station dataset
3. Reprojects both layers into a projected CRS
4. Uses sjoin_nearest() to find the closest fire station for each sinkhole
5. Stores the distance in meters
6. Prints a table with:
	- sinkhole_id
	- station_id
	- station_name
	- distance_m

**Plain-English plan first**

Before writing the code, describe the workflow in one or two sentences.

**Example:**

I will load both GeoDataFrames, project them into a 
CRS with meter units, then use sjoin_nearest() to 
attach the nearest fire station to each sinkhole and 
store the distance in a new column.

```
This step matters. Spatial thinking first. Syntax second.
```


## Quick Check
1. What is the main GeoPandas function for nearest neighbor joins?
```
gpd.sjoin_nearest()
```
2. Why should you avoid measuring distance in EPSG:4326?
```
Because the units are degrees, not meters.
```
3. What does how="left" do in this pattern?
```
It keeps every row from the left GeoDataFrame.
```

## Key Takeaway

The big win today is not just learning a function name.

It is seeing that the nearest neighbor pattern survives the move from SQL to Python.

- In SQL, you learned the logic.
- In Python, you apply the same logic with GeoPandas.

That is how fluency builds. Not by memorizing isolated commands, but by recognizing the same spatial pattern across tools.

