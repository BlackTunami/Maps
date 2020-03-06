---
title: "Mapping in R - selected exercises"
output: github_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

# World Map of Measles Vaccinations
## World Development Indicators (WDI)

For this exercise we will work with data from the World Development Indicators (WDI). Vincent Arel-Bundock provides a nice package for R that makes it easy to import the data. 

```{r, message=FALSE}
# install.packages("WDI")
library(WDI)
library(dplyr)
```

## Select the data

Let's get some data on measles vaccinations. `SH.IMM.MEAS` seems like a good fit. But feel free to use another variable you find interesting.

```{r}
WDIsearch('measles')
df = WDI(indicator = "SH.IMM.MEAS" ,
         start = 2017, end = 2017, extra = F)
df = df %>% filter(!is.na(SH.IMM.MEAS))
```

## Questions

1. Use the map package and the measles data to make a world map of the share of infants vaccinated against measles.

2. Install the package `countrycode()` and use the countrycode function to add a region indicator to the dataset. Create a world map faceted by your region indicator.

# Locations of Fortune 500 companies

## Where are the Fortune 500 headquarters

For this exercise, we want to create a map of where the Fortune 500 companies - that is the five hundred largest U.S. corporations by total revenue - have their headquarters. 


## Addresses

Let's get the addresses here: https://www.geolounge.com/fortune-500-list-by-state-for-2015/

```{r, eval=FALSE}
library(XML)
library(RCurl)
fortune500_url <- getURL("https://www.geographyrealm.com/fortune-500-list-by-state-for-2015/",.opts = list(ssl.verifypeer = FALSE) )  # We needs this because the site is https
fortune500 = readHTMLTable(fortune500_url, header = TRUE, which = 1)
colnames(fortune500) <- tolower(colnames(fortune500))
fortune500 <- subset(fortune500, select=c("company","streetadd","place","state","zip"))
write.csv(fortune500, "fortune500.csv")
```


## Load the associated file of addresses 

Load the list of Fortune 500 companies (in 2015).

```{r}
library(readr)
fortune500 <- read_csv("fortune500.csv")
```

## Task 1: Make a map by state

**Task**: 
Aggregate the number of headquarters by state. Make a map in which the states are shaded by their number of Fortune 500 companies

Use `dplyr()` for the aggregation and the `maps()` package for a map of the U.S. with state boundaries. 

## Task 2: Tax Rates by State

**Task**: 
We also got some info on top corporate income tax rates by state. Import sheet 2 of the file `State_Corporate_Income_Tax_Rates_2015.xlsx` and make a map shaded by top corporate income tax rates. 

## Task 3: Scatter plot of income tax rates and # of headquarters

Is there evidence of companies being headquartered in low tax states? 

**Task**: 
1. Make a scatter plot with a loess function estimating the relationship between corporate income tax rates (x-axis) and # of headquarters (y-axis). 
2. What happens if we account for state population (i.e use HQs per capita)? Import the data on state populations from Wikipedia: https://en.wikipedia.org/wiki/List_of_U.S._states_and_territories_by_population. Use the function `readHTMLTable()` from the package `XML` to import the data and merge it on. Re-do the scatter plot from part 1. 

## Geocoding

Now, let's get the Latitude and Longitude coordinates of all these addresses of the HQ addresses. Fortunately, ggmap() does this for us nicely.

```{r}
library(ggmap)
# This is Walmart's HQ address:
geocode("702 S.W. Eighth St. Bentonville Arkansas 72716", output = "latlon" , source = "google")
```

## Task 4: Geocode headquarter locations and add to the plot

**Task**: 
1. Geocode the locations of all headuarters in the sample. Add the locations of the headquarters as points to the map from part 2 (U.S. state map shaded by top corporate income tax rates).
2. Add a label with the company names. Use ggrepel() if the labels overlap too much.
3. Size the points by the ranking on the Fortune 500 list (we don't have revenue here, so the rank will have to do). 
