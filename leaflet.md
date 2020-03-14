## Introduction
### 基本绘图
```r
m <- leaflet()
m <- addTiles(m)
m <- addMarkers(m, lng=174.768, lat=-36.852, popup="The birthplace of R")
m
```

```r
m <- leaflet() %>%
  addTiles() %>%  
  addMarkers(lng=174.768, lat=-36.852, popup="The birthplace of R")
m  
```

创建地图的步骤如下
- 用``leaflet()``创建地图  
- 添加涂层layer(``addTiles``,``addMarkers``,``addPolygons``)

## The Map Widget 地图零部件
### Map methods
- ``setView()``: 设置中心点和放大倍数
- ``fitBounds()``: 设置界限[lng1, lat1] – [lng2, lat2]
- ``clearBounds()``


### The data object 数据映射
```r
df = data.frame(Lat = 1:10, Long = rnorm(10))
leaflet(df) %>% addCircles()
leaflet() %>% addCircles(data = df)
leaflet() %>% addCircles(data = df, lat = ~ Lat, lng = ~ Long)
```

```r
m = leaflet() %>% addTiles()
df = data.frame(
  lat = rnorm(100),
  lng = rnorm(100),
  size = runif(100, 5, 20),
  color = sample(colors(), 100)
)
m = leaflet(df) %>% addTiles()
m %>% addCircleMarkers(radius = ~size, color = ~color, fill = FALSE)
m %>% addCircleMarkers(radius = runif(100, 4, 10), color = c('red'))
```

## Use basemaps
### 地图类型

```r
m <- leaflet() %>% setView(lng = -71.0589, lat = 42.3601, zoom = 12)
m %>% addTiles()
```
``
m %>% addProviderTiles(providers$Stamen.Toner)  
``

``
m %>% addProviderTiles(providers$CartoDB.Positron)
``

``
m %>% addProviderTiles(providers$Esri.NatGeoWorldMap)
``

### Custom Title URL template
```r
leaflet() %>% addTiles() %>% setView(-93.65, 42.0285, zoom = 4) %>%
  addWMSTiles(
    "http://mesonet.agron.iastate.edu/cgi-bin/wms/nexrad/n0r.cgi",
    layers = "nexrad-n0r-900913",
    options = WMSTileOptions(format = "image/png", transparent = TRUE),
    attribution = "Weather data © 2012 IEM Nexrad"
  )
```

### Combing  Title layers

```r
m %>% addProviderTiles(providers$MtbMap) %>%
  addProviderTiles(providers$Stamen.TonerLines,
    options = providerTileOptions(opacity = 0.35)) %>%
  addProviderTiles(providers$Stamen.TonerLabels)
```

## Markers

### Icon Markers
```r
data(quakes)

# Show first 20 rows from the `quakes` dataset
leaflet(data = quakes[1:20,]) %>% addTiles() %>%
  addMarkers(~long, ~lat, popup = ~as.character(mag), label = ~as.character(mag))
```

#### Awesome Icons
```r
df.20 <- quakes[1:20,]

getColor <- function(quakes) {
  sapply(quakes$mag, function(mag) {
  if(mag <= 4) {
    "green"
  } else if(mag <= 5) {
    "orange"
  } else {
    "red"
  } })
}

icons <- awesomeIcons(
  icon = 'ios-close',
  iconColor = 'black',
  library = 'ion',
  markerColor = getColor(df.20)
)

leaflet(df.20) %>% addTiles() %>%
  addAwesomeMarkers(~long, ~lat, icon=icons, label=~as.character(mag))
```

#### Marker Clusters

```r
leaflet(quakes) %>% addTiles() %>% addMarkers(
  clusterOptions = markerClusterOptions()
)
```
``markerClusterOptions(freezeAtZoom = 5)``固定倍率

#### Circle Markers
```
leaflet(df) %>% addTiles() %>% addCircleMarkers()
```

```r
df <- sp::SpatialPointsDataFrame(
  cbind(
    (runif(20) - .5) * 10 - 90.620130,  # lng
    (runif(20) - .5) * 3.8 + 25.638077  # lat
  ),
  data.frame(type = factor(
    ifelse(runif(20) > 0.75, "pirate", "ship"),
    c("ship", "pirate")
  ))
)
# Create a palette that maps factor levels to colors
pal <- colorFactor(c("navy", "red"), domain = c("ship", "pirate"))

leaflet(df) %>% addTiles() %>%
  addCircleMarkers(
    radius = ~ifelse(type == "ship", 6, 10),
    color = ~pal(type),
    stroke = FALSE, fillOpacity = 0.5
  )
```

## Popups
``addPopups()``

```r
content <- paste(sep = "<br/>",
  "<b><a href='http://www.samurainoodle.com'>Samurai Noodle</a></b>",
  "606 5th Ave. S",
  "Seattle, WA 98138"
)

leaflet() %>% addTiles() %>%
  addPopups(-122.327298, 47.597131, content,
    options = popupOptions(closeButton = FALSE)
  )
```

### Labels
