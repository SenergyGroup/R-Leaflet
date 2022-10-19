# R-Leaflet

Creating leaflet objects in R using Census data.

## Prerequisite

Before you begin. You will need an API to access the Census data. You
can obtain one here: <https://api.census.gov/data/key_signup.html>.

Once you have acquired an API key you are ready to go!

## Step 1: The setup

Start by installing the `tidycensus` package.

    install.packages("tidycensus")
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
        Hispanic = "B03002_012",
        White = "B03002_003",
        Black = "B03002_004"
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
