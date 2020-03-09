---
title: "Assignment 2: Mapping Fire Incidents and FDNY Response Times"
author: Thomas Brambor
date: 2020-03-07
always_allow_html: yes
output: 
  html_document:
    keep_md: true
---

Fires in NYC and FDNY Response
================================

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Overview

For this assignment, we are going to investigate fires requiring the fire department to respond. Using data about the locations of firehouses and fires occurring in New York City, we want to know whether response times to fires differ across the city. Second, we will try to focus on one possible variable that could affect response times -- the distance from the firehouse -- and see whether we find the (expected) effect.

To keep this homework manageable, I am leaving out another part of the investigation: What is the effect of demographic and/or income characteristics of the neighborhood on response times. This is likely a bit more sensitive but also relevant from a public policy perspective.  

## Data

We rely on two data sets.

#### Incidents responded to by fire companies

NYC Open Data has data on all [incidents responded to by fire companies](https://data.cityofnewyork.us/Public-Safety/Incidents-Responded-to-by-Fire-Companies/tm6d-hbzd). I have included the variable description file in the exercise folder. The following variables are available:

  - IM_INCIDENT_KEY:	Unique identifier for each incident which serves
  - INCIDENT_TYPE_DESC	The code and description of the incident category type
  - INCIDENT_DATE_TIME	The date and time that the incident was logged into the Computer Aided Dispatch system
  - ARRIVAL_DATE_TIME	The date and time that the first unit arrived on scene
  - UNITS_ONSCENE	Total number of units that arrived on scene
  - LAST_UNIT_CLEARED_DATETIME	The date and time that the incident was completed and the last unit cleared the scene
  - HIGHEST_LEVEL_DESC	The highest alarm level that the incident received
  - TOTAL_INCIDENT_DURATION	The total number of seconds from when then incident was created to when the incident was closed
  - ACTION_TAKEN1_DESC	The code and description of the first action taken
  - ACTION_TAKEN2_DESC	The code and description of the second action taken
  - ACTION_TAKEN3_DESC	The code and description of the third action taken
  - PROPERTY_USE_DESC	The code and description of the type of street or building where the incident took place
  - STREET_HIGHWAY	The name of the street where the incident_took place
  - ZIP_CODE	The postal zip code where the incident took place
  - BOROUGH_DESC	The borough where the incident took place
  - FLOOR	The floor of the building where the incident took place
  - CO_DETECTOR_PRESENT_DESC	Indicator for when a CO detector was present
  - FIRE_ORIGIN_BELOW_GRADE_FLAG	Indicator for when the fire originated below grade
  - STORY_FIRE_ORIGIN_COUNT	Story in which the fire originated
  - FIRE_SPREAD_DESC	How far the fire spread from the object of origin
  - DETECTOR_PRESENCE_DESC	Indicator for when a  detector was present
  - AES_PRESENCE_DESC	Indicator for when an Automatic Extinguishing System is present
  - STANDPIPE_SYS_PRESENT_FLAG	Indicator for when a standpipe was present in the area of origin of a fire

This dataset is only updated annually, and thus far only data from 2013 to 2018 is contained. The full dataset is also somewhat too large for an exercise (2.5M rows), so I suggest to limit yourself to a subset. I have added a file containing the subset of of only building fires (`INCIDENT_TYPE_DESC == "111 - Building fire"`) for 2013 to 2018 only which yields about 14,000 incidents.

```{r, eval=FALSE}
library(tidyverse)
fire_all <- read_csv("no_upload/Incidents_Responded_to_by_Fire_Companies.csv")
fire <- fire_all %>%
  filter(INCIDENT_TYPE_DESC == "111 - Building fire")
```

Unfortunately, the addresses of the incidents were not geocoded yet. Ideally, I would like you to know how to do this but am mindful about the hour or so required to get this done. So, here is the code. The geocodes (as far as they were returned successfully) are part of the data (as variables `lat` and `lon`).

```{r, eval=FALSE}
library(ggmap)

# Register Google API Key
register_google(key = Sys.getenv("GOOGLE_MAPS_API_KEY"))

# Create and geocode addresses
fire <- fire %>%
  mutate(address = str_c( str_to_title(fire$STREET_HIGHWAY),
                  "New York, NY",
                  fire$ZIP_CODE,
                  sep=", ")) %>%
  filter(is.na(address)==FALSE) %>%
  mutate_geocode(address)

# Save File
write_csv(fire, "building_fires.csv")
```

#### FDNY Firehouse Listing

NYC Open Data also provides data on the [location of all 218 firehouses in NYC](https://data.cityofnewyork.us/Public-Safety/FDNY-Firehouse-Listing/hc8x-tcnd). Relevant for our analysis are the following variables: `FacilityName`, `Borough`, `Latitude`, `Longitude`

```{r, eval=FALSE}
library(tidyverse)
firehouses <- read_csv("FDNY_Firehouse_Listing.csv") %>%
  dplyr::filter(!is.na(Latitude))
```

_Note:_ 5 entries contain missing information, including on the spatial coordinates. We can exclude these for the exercise. 

## Tasks

#### 1. Location of Severe Fires

Provide a `leaflet` map of the highest severity fires (i.e. subset to the highest category in `HIGHEST_LEVEL_DESC`)  contained in the file `buiding_fires.csv`. Ignore locations that fall outside the five boroughs of New York City. Provide at least three pieces of information on the incident in a popup. 

#### 2. Layers and Clusters

##### a) Color by Type of Property

Start with the previous map. Now, distinguish the markers of the fire locations by `PROPERTY_USE_DESC`, i.e. what kind of property was affected. If there are too many categories, collapse some categories. Choose an appropriate coloring scheme to map the locations by type of affected property. Add a legend informing the user about the color scheme. Also make sure that the information about the type of affected property is now contained in the popup information. Show this map.

##### b) Cluster

Add marker clustering, so that zooming in will reveal the individual locations but the zoomed out map only shows the clusters. Show the map with clusters.

#### 3. Fire Houses

The second data file contains the locations of the 218 firehouses in New York City. Start with the non-clustered map (2b) and now adjust the size of the circle markers by severity (`TOTAL_INCIDENT_DURATION` or `UNITS_ONSCENE` seem plausible options). More severe incidents should have larger circles on the map. On the map, also add the locations of the fire houses. Add two layers ("Incidents", "Firehouses") that allow the user to select which information to show. 

#### 4. Distance from Firehouse and Response Time

We now want to investigate whether the distance of the incident from the nearest firehouse varies across the city. 

##### a) Calculate Distance

For all incident locations (independent of severity), identify the nearest firehouse and calculate the distance between the firehouse and the incident location. Provide a scatter plot showing the time until the first engine arrived (the variables `INCIDENT_DATE_TIME`  and `ARRIVAL_DATE_TIME`) will be helpful. 

Now also visualize the patterns separately for severe and non-severe incidents (use `HIGHEST_LEVEL_DESC` but feel free to reduce the number of categories). What do you find?

##### b) Map of Response Times

Provide a map visualization of response times. Investigate whether the type of property affected (`PROPERTY_USE_DESC`) or fire severity (`HIGHEST_LEVEL_DESC`) play a role here.

Show a faceted choropleth map indicating how response times have developed over the years. What do you find?

