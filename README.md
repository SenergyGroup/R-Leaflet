# R-Leaflet

Creating leaflet objects in R using Census data.

## Prerequisite

Before you begin. You will need an API to access the Census data. You
can obtain one here: <https://api.census.gov/data/key_signup.html>.

Once you have acquired an API key you are ready to go!

## Step 1: The setup

Start by installing the `tidycensus` package and `tidyverse`.

    install.packages("tidycensus")
    install.packages("tidyverse")
    library(tidycensus)
    library(tidycensus)

Before pulling data, I like to use the following option for pulling
geometry data. This makes it so the data is cached, so pulling data
multiple times will be much quicker.

    options(tigris_use_cache = TRUE)

Now, setup you API key by running the following code.

    census_api_key("YOUR API KEY GOES HERE")

## Step 2: Census API Variables

Census data is pulled using the `tidycensus` package. There are two main
calls that we will use, `get_decennial` and `get_acs`.

Before you pull data, you will need to find which variables you want to
pull. To do this, use the function `load_variables` on the year and data
set you want to analyze.

The code below pulls the variables for the 2020 5-year ACS data.

    vars <- tidycensus::load_variables(year = 2020, dataset = "acs5")

For this tutorial I want to use population counts for Hispanic and
non-Hispanic population.

To make things easier on myself, I created a list variable that contains
each of the API codes I want to analyze.

    acs2020 <-   c(
        Black_or_AA = "B03002_004"
      )

## Step 3: Pulling Census Data

Now we can start pulling Census data using the API.

Thankfully, `tidycensus` makes using the API a painless process. The
code below shows the general structure of the API call.

    df_name <- get_acs(
      geography = "geography type you want to analyze: options are listed in the vinette",
      variables = "variables you want to analyze. This is why I made the acs2020 variable earlier",
      summary_var = "variable you want to compare against the variables you pulled. This is usually total population",
      year = #whatever year you want to analyze,
      survey = "dataset you want to analyze",
      geometry = #put TRUE if you want GIS coordinates, otherwise put FALSE
    )

Below is a completed example.

    ZIP_race <- get_acs(
      geography = "zcta",
      variables = acs2020,
      cache_table = TRUE,
      summary_var = "B03002_001",
      year = 2020,
      survey = "acs5",
      geometry = TRUE
    )

Visit the [tidycensus website](https://walker-data.com/tidycensus/) for
more information on how you can build your API call to fit your needs.

## Step 4: Leaflet

Now comes the fun part!

First, install `leaflet` if you don’t have it already.

    install.packages("leaflet")
    library(leaflet)

You can create a base map with no data using the following code.

    map = leaflet() %>% 
      addTiles()

You will need to create a palette for the colors you want displayed on
your map. To do this, you can find the quartiles and base your colors on
that.

The code below shows the full process of finding the quartiles and
creating the palettes using “bins.”

    quantile(ZIP_race$estimate)

    zbinsb <- c(0,0.1,30.0,598.5,82872.0)
    zpalb <- colorBin("YlOrRd", domain = ZIP_race$estimate, bins = zbinsb)

Now can add the Census data using the `addPolygons` function and
specifying data and palette we just made.

    map = leaflet() %>% 
      addTiles() %>% 
      addPolygons(data = ZIP_race,
                  fillColor = ~zpalb(estimate)
        
      )

You learn more about Leaflet customization on their website
<https://leafletjs.com/>

Below is an example of a fully customized map. The possiblities are
endless!

    map = leaflet() %>% 
      
      setView(lng = 74.0060, lat = 40.7128, zoom = 20.25) %>% 

      addProviderTiles("CartoDB.Positron") %>%
      
      addPolygons(data = ZIP_race,
                  fillColor = ~zpalb(estimate),
                  opacity = 1,
                  fillOpacity = 0.6,               
                  label = ZIP_race$NAME,
                  labelOptions = labelOptions(
                   textsize = "14px",
                   style = list(
                     "font-weight" = "bold",
                     padding = "5px"
                   )
                 ),
                  group = "Black or African American",
                  popup = paste0("Location: ", ZIP_race$NAME, "<br>",
                                  "Count: ", ZIP_race$estimate),
                  highlightOptions = highlightOptions(
                    weight = 2,
                    color = "#A4F2F4",
                    fillOpacity = 0.6,
                    bringToFront = TRUE)
                  ) %>% 
      addLegend(data = ZIP_race,
        pal = zpalb, 
        group = "Black or African American",
        values = ~estimate,
        opacity = 0.6, 
        title = "Count: Black or African American",
        position = "bottomleft")
