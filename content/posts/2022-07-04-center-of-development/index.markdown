---
title: Center of Development
author: Rich Posert
date: '2022-07-04'
slug: []
categories: []
tags: []
---



In line with [last week's building footprint plots](/posts/2022-06-25-portland-building-footprints/)
I want to keep working with the Portland GIS data. I had an idea to try to
plot the movement of the center of new development every so often (year/decade?)
for Portland.

So, let's load up some data:


```r
border <- st_read('../2022-06-25-portland-building-footprints/big_data/City_Boundaries/') |> 
  filter(CITYNAME == 'Portland')
```

```
## Reading layer `City_Boundaries' from data source 
##   `C:\Users\rich\Documents\blog\content\posts\2022-06-25-portland-building-footprints\big_data\City_Boundaries' 
##   using driver `ESRI Shapefile'
## Simple feature collection with 57 features and 5 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -13747930 ymin: 5632401 xmax: -13606340 ymax: 5800163
## Projected CRS: WGS 84 / Pseudo-Mercator
```

```r
footprints <- st_read('../2022-06-25-portland-building-footprints/big_data/Building_Footprints/') |> 
  st_filter(border, .predicate = st_intersects)
```

```
## Reading layer `Building_Footprints' from data source 
##   `C:\Users\rich\Documents\blog\content\posts\2022-06-25-portland-building-footprints\big_data\Building_Footprints' 
##   using driver `ESRI Shapefile'
## Simple feature collection with 727134 features and 42 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -13734820 ymin: 5605674 xmax: -13548510 ymax: 5745079
## Projected CRS: WGS 84 / Pseudo-Mercator
```

"It should be easy", he said, certainly cursing himself:


```r
building_centroids <- footprints |> 
  mutate(Decade = YEAR_BUILT - (YEAR_BUILT %% 10)) |> 
  st_centroid()

# stolen from https://stackoverflow.com/questions/70561598/calculate-the-average-of-a-cloud-of-points-based-on-a-grouping-variable-with-sf
decade_centroids <- aggregate(building_centroids, by = list(building_centroids$Decade), FUN = function(x) x = x[1]) |> 
  st_centroid()

decade_centroids |> 
  filter(Decade > 1800 & Decade <= 2022) |> 
  ggplot() +
  theme_void() +
  geom_sf(data = border, fill = NA) +
  geom_sf(aes(color = Decade)) +
  scale_color_viridis_c()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/initial-plot-1.png" width="672" />

Hmm...less interesting than I expected! Oh well, here's your slop.
