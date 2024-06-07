---


title: "County & Tract-level demographic data via TidyCensus"


format:


  html:


    theme: cosmo


---







### About this page





Below, I have provided the code to produce dataframes with county and tract level demographic information, including race/ethnicity, poverty rate, and median income. These dataframes are stored in the `census_tract_stats` and `census_county_stats` variables. These dataframes are written to csv files. 





A future iteration of this code will functionalize the process of retrieving nationwide county and tract data in an R script.








### Set up libraries







::: {.cell}

```{.r .cell-code}
library(tidyverse)


library(tidycensus)


library(janitor)
```
:::



::: {.cell}

```{.r .cell-code}
# set api key for tidycensus


# census_api_key("API_KEY", install=TRUE)
```
:::







### Instantiate global vars







::: {.cell}

```{.r .cell-code}
# instantiate global variable 


data(fips_codes)





## store to var


fips_codes <- fips_codes  
```
:::







### Retrieve tract-level demographic data







::: {.cell}

```{.r .cell-code}
## Get list of states (Exclude non-states, except DC)


states <- fips_codes %>%


  select(state) %>%


  distinct() %>%


  head(51) %>%


  as_vector() 





# Get census tract data for all states


census_tract_stats <- get_acs(geography = "tract", variables = c( "B01001_001","B02001_002","B02001_003","B02001_004","B03001_003","B06012_002","B19013_001"), state=states, year = 2022) %>%


  select(GEOID, variable, estimate) %>%


  pivot_wider(names_from = variable, values_from = estimate) %>%


  rename(

    total_pop = B01001_001,

    white_pop = B02001_002,

    black_pop = B02001_003,

    native_pop = B02001_004,

    hispanic_pop = B03001_003,

    poverty_pop = B06012_002,

    median_income = B19013_001


  ) %>%


  mutate(pct_white = round(white_pop/total_pop, 2) * 100,

         pct_nonwhite = 100 - round(white_pop/total_pop, 2) * 100,

         pct_black = round(black_pop/total_pop, 2) * 100,

         pct_native = round(native_pop/total_pop, 2) * 100,

         pct_hispanic = round(hispanic_pop/total_pop, 2) * 100,

         pct_poverty = round(poverty_pop/total_pop, 2) * 100

         ) %>%


  clean_names()





# Extract state and county FIPS codes from GEOID


census_tract_stats <- census_tract_stats %>%


  mutate(state_fips = substr(geoid, 1, 2),

         county_fips = substr(geoid, 3, 5))





# Join with fips_codes to get state and county names


fips_codes <- fips_codes %>%


  select(state_fips = state_code, county_fips = county_code, state, county)





census_tract_stats <- census_tract_stats %>%


  left_join(fips_codes, by = c("state_fips", "county_fips")) %>%


  select(geoid, state, county, starts_with("pct"), median_income)
```
:::



::: {.cell}

```{.r .cell-code}
# write dataframe to csv


write.csv(census_tract_stats, file = "data/census-tract-stats.csv", row.names = FALSE)
```
:::










### Retrieve county-level demographic data







::: {.cell}

```{.r .cell-code}
# Get county-level data for all states


census_county_stats <- get_acs(geography = "county", variables = c( "B01001_001","B02001_002","B02001_003","B02001_004","B03001_003","B06012_002","B19013_001"), year = 2022) %>%


  select(GEOID, NAME, variable, estimate) %>%


  pivot_wider(names_from = variable, values_from = estimate) %>%


  rename(

    total_pop = B01001_001,

    white_pop = B02001_002,

    black_pop = B02001_003,

    native_pop = B02001_004,

    hispanic_pop = B03001_003,

    poverty_pop = B06012_002,

    median_income = B19013_001


  ) %>%


  mutate(pct_white = round(white_pop/total_pop, 2) * 100,

         pct_nonwhite = 100 - round(white_pop/total_pop, 2) * 100,

         pct_black = round(black_pop/total_pop, 2) * 100,

         pct_native = round(native_pop/total_pop, 2) * 100,

         pct_hispanic = round(hispanic_pop/total_pop, 2) * 100,

         pct_poverty = round(poverty_pop/total_pop, 2) * 100

         ) %>%


  clean_names() %>% 


  separate(name, into = c("county", "state"), sep = ", ")
```
:::



::: {.cell}

```{.r .cell-code}
# write dataframe to csv


write.csv(census_county_stats, file = "data/census-county-stats.csv", row.names = FALSE)
```
:::



