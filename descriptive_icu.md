icu details
================
Laura Cosgrove
12/1/2018

``` r
# Load configuration settings
dbdriver <- 'PostgreSQL'
host  <- '127.0.0.1'
port  <- '5432'
user  <- 'postgres'
password <- 'postgres'
dbname <- 'mimic'
schema <- 'mimiciii'
# Connect to the database using the configuration settings
con <- dbConnect(dbDriver(dbdriver), dbname = dbname, host = host, port = port, 
                 user = user, password = password)
# Set the default schema
dbExecute(con, paste("SET search_path TO ", schema, sep=" "))
```

    ## [1] 0

Set this database as the connection for all future sql chunks:

``` r
knitr::opts_chunk$set(connection = "con")
```

Set evaluation to false so that you don't requery your database every time you knit.

``` r
icu_detail_view <- read_file("./database/mimic-code/concepts/demographics/icustay-detail.sql")


#Generate materialized views
dbGetQuery(con, icu_detail_view)
```

``` r
icu_detail_query <- "SELECT *
              FROM icustay_detail i;"
icu_detail_data <- as.tibble(dbGetQuery(con, icu_detail_query))
#write_csv(sapsii_data, path = "./database/sapsii.csv")
icu_detail_data
```

    ## # A tibble: 61,051 x 19
    ##    subject_id hadm_id icustay_id gender dod                
    ##  *      <int>   <int>      <int> <chr>  <dttm>             
    ##  1          2  163353     243653 M      NA                 
    ##  2          3  145834     211552 M      2102-06-14 00:00:00
    ##  3          4  185777     294638 F      NA                 
    ##  4          5  178980     214757 M      NA                 
    ##  5          6  107064     228232 F      NA                 
    ##  6          7  118037     278444 F      NA                 
    ##  7          7  118037     236754 F      NA                 
    ##  8          8  159514     262299 M      NA                 
    ##  9          9  150750     220597 M      2149-11-14 00:00:00
    ## 10         10  184167     288409 F      NA                 
    ## # ... with 61,041 more rows, and 14 more variables: admittime <dttm>,
    ## #   dischtime <dttm>, los_hospital <dbl>, admission_age <dbl>,
    ## #   ethnicity <chr>, admission_type <chr>, hospital_expire_flag <int>,
    ## #   hospstay_seq <dbl>, first_hosp_stay <lgl>, intime <dttm>,
    ## #   outtime <dttm>, los_icu <dbl>, icustay_seq <dbl>, first_icu_stay <lgl>

The ICU detail query extracts both hospital level length of stay as well as ICU-level length of stay.
