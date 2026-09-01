# Day 26: Inspecting Raster Metadata with Rasterio

Today I practiced the core Rasterio pattern for safely opening a GeoTIFF and inspecting its metadata.

## What I built

A small Python script that opens a raster file and prints:

- Coordinate Reference System (CRS)
- Number of bands
- Raster width and height
- Bounds
- Data type

## Code

**A few scripts I wrote**

```python

import rasterio
with rasterio.open("landcover.tif") as src: 
    print(f"CRS: {src.crs}") 
    print(f"Band count: {src.count}") 
    print(f"Width: {src.width}") 
    print(f"Height: {src.height}") 
    print(f"Bounds: {src.bounds}") 
    print(f"Data types: {src.dtypes}")

```

```Python

import rasterio
with rasterio.open('landcover.tif') as src:
    crs = src.crs
    band_count = src.count
    print(f"CRS: {crs}")
    print(f"Band count: {band_count}")

```

## What I learned

with rasterio.open(...) as src: safely opens and closes the raster file.

In plain English, this script opens landcover.tif, reads its metadata, prints the key properties, then closes the file cleanly.

## AI pairing prompt

Write a Python script using Rasterio to open landcover.tif with a context manager. Print the CRS, band count, width, height, bounds, and data types. Add short comments explaining each property.

## Next step
Run this against a real GeoTIFF and add one sentence describing what the raster represents.


