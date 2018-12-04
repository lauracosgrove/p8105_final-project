Minor improvement in generating LOS
================
Laura Cosgrove
11/20/2018

This implementation works if you use knitr to generate the analysis! But in building the analysis it is maybe better to run your query by saving the sql query as a character object then using `dbGetQuery`.

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

Credit for SQL code authoring is not mine:

``` sql
SELECT i.subject_id, i.hadm_id, i.los
              FROM icustays i;
```

Another option is to save the SQL directly using readLines, then use a wrapper to apply `dbGetQuery` to each line. This will be implemented for larger SQL files.

``` r
head(los_data)
```

    ##   subject_id hadm_id    los
    ## 1        268  110404 3.2490
    ## 2        269  106296 3.2788
    ## 3        270  188028 2.8939
    ## 4        271  173727 2.0600
    ## 5        272  164716 1.6202
    ## 6        273  158689 1.4862

Now calculate medians (redo in tibble framework later)

``` r
avg_los <- median(los_data$los, na.rm=TRUE)
rounded_avg_los <-round(avg_los, digits = 2)
```

Todo is to redo plot in ggplot framework
