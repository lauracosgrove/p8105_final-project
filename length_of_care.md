length\_of\_care
================
Samantha Brown
11/17/2018

``` r
admissions <- 
  read.csv("./database/admissions.csv") %>% 
  janitor::clean_names()
names(admissions)
```

    ##  [1] "row_id"               "subject_id"           "hadm_id"             
    ##  [4] "admittime"            "dischtime"            "deathtime"           
    ##  [7] "admission_type"       "admission_location"   "discharge_location"  
    ## [10] "insurance"            "language"             "religion"            
    ## [13] "marital_status"       "ethnicity"            "edregtime"           
    ## [16] "edouttime"            "diagnosis"            "hospital_expire_flag"
    ## [19] "has_chartevents_data"

``` r
## Top 10 causes of recorded mortalities
top_causes_of_deaths = admissions %>% 
  filter(deathtime != "NA") %>%
  count(diagnosis) %>% 
  top_n(10)
```

    ## Selecting by n

``` r
sepsis = admissions %>% 
  filter(diagnosis == "SEPSIS") %>% 
  mutate(mortality = ifelse(deathtime == "NA", 0, 1))

## Look at top 10 causes of mortalities
mortalities = admissions %>% 
  filter(diagnosis == c("ABDOMINAL PAIN", "ALTERED MENTAL STATUS", "CARDIAC ARREST", "CONGESTIVE HEART FAILURE", "HEAD BLEED", "HYPOTENSION", "INTACRANIAL HEMORRHAGE", "PNEUMONIA", "SEPSIS", "STROKE;TELEMETRY;TRANSIENT ISCHEMIC ATTACK"))
```

Exploratory:
------------

I want to discuss with you guys the challenge in finding length of care -- if someone was admitted on December 1, 1980 and discharged January 5, 1981, I'm not sure if we can just subtract the times? Using Francis's method, it results in negative number for month. Also, we cannot measure minutes/hours, because admit time does not have minutes

Should we measure length of stays by hours? then we can combine day and hour to one unit measurement

``` r
## Try to manipulate times in POSIXct format
## Admit time does not have minutes. Only day, month, year

class(admissions$admittime)
```

    ## [1] "factor"

``` r
class(admissions$dischtime)
```

    ## [1] "factor"

``` r
time_in = as.POSIXct.Date(admissions$admittime)
time_out = as.POSIXct.Date(admissions$dischtime)

length_of_care = time_out - time_in %>% 
  View()
```

``` r
## Manipulate time with Francis's method 
admissions <- 
  admissions %>% 
  separate(admittime, into = c("admittime_year", "admittime_month", "admittime_day"), sep = "-")
admissions <- 
  admissions %>% 
  separate(dischtime, into = c("dischtime_year", "dischtime_month", "dischtime_day"), sep = "-") %>% 
  separate(dischtime_day, into = c("dischtime_day", "dischtime_time"), sep = " ")
admissions <- 
  admissions %>% 
  separate(deathtime, into = c("deathtime_year", "deathtime_month", "deathtime_day"), sep = "-") %>% 
  separate(deathtime_day, into = c("deathtime_day", "deathtime_time"), sep = " ")
admissions <- 
  admissions %>% 
  separate(edregtime, into = c("edregtime_year", "edregtime_month", "edregtime_day"), sep = "-") %>% 
  separate(edregtime_day, into = c("edregtime_day", "edregtime_time"), sep = " ")
admissions <- 
  admissions %>% 
  separate(edouttime, into = c("edouttime_year", "edouttime_month", "edouttime_day"), sep = "-") %>% 
  separate(edouttime_day, into = c("edouttime_day", "edouttime_time"), sep = " ")

# Correct year to normal
admissions <- 
  admissions %>% 
  mutate(admittime_year = as.numeric(admittime_year) - 200, dischtime_year = as.numeric(dischtime_year) - 200, deathtime_year = as.numeric(deathtime_year) - 200, edregtime_year = as.numeric(edregtime_year) - 200, edouttime_year = as.numeric(edouttime_year) - 200)
```

``` r
## Laura's Method
admissions_data <- read_csv("./database/data/admissions.csv") %>% 
  janitor::clean_names()
```

    ## Parsed with column specification:
    ## cols(
    ##   row_id = col_integer(),
    ##   subject_id = col_integer(),
    ##   hadm_id = col_integer(),
    ##   admittime = col_integer(),
    ##   dischtime = col_date(format = ""),
    ##   deathtime = col_datetime(format = ""),
    ##   admission_type = col_datetime(format = ""),
    ##   admission_location = col_character(),
    ##   discharge_location = col_character(),
    ##   insurance = col_character(),
    ##   language = col_character(),
    ##   religion = col_character(),
    ##   marital_status = col_character(),
    ##   ethnicity = col_character(),
    ##   edregtime = col_character(),
    ##   edouttime = col_datetime(format = ""),
    ##   diagnosis = col_datetime(format = ""),
    ##   hospital_expire_flag = col_character(),
    ##   has_chartevents_data = col_integer()
    ## )

    ## Warning in rbind(names(probs), probs_f): number of columns of result is not
    ## a multiple of vector length (arg 1)

    ## Warning: 5772 parsing failures.
    ## row # A tibble: 5 x 5 col     row col   expected   actual     file                             expected   <int> <chr> <chr>      <chr>      <chr>                            actual 1     1 <NA>  19 columns 20 columns './database/data/admissions.csv' file 2     2 <NA>  19 columns 20 columns './database/data/admissions.csv' row 3     3 <NA>  19 columns 20 columns './database/data/admissions.csv' col 4     4 <NA>  19 columns 20 columns './database/data/admissions.csv' expected 5     5 <NA>  19 columns 20 columns './database/data/admissions.csv'
    ## ... ................. ... .................................................................... ........ .................................................................... ...... .................................................................... .... .................................................................... ... .................................................................... ... .................................................................... ........ ....................................................................
    ## See problems(...) for more details.

``` r
difference = admissions_data %>% 
  mutate(difference = as.POSIXct(dischtime - admittime, origin = "1556-04-29 19:03:58")) %>% 
  select(difference)
```
