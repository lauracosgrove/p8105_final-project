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
deaths = admissions %>% 
  filter(deathtime != "NA") %>%
  count(diagnosis) %>% 
  top_n(10)
```

    ## Selecting by n

``` r
deaths
```

    ## # A tibble: 10 x 2
    ##    diagnosis                                      n
    ##    <fct>                                      <int>
    ##  1 ABDOMINAL PAIN                                 7
    ##  2 ALTERED MENTAL STATUS                         10
    ##  3 CARDIAC ARREST                                11
    ##  4 CONGESTIVE HEART FAILURE                      12
    ##  5 HEAD BLEED                                     9
    ##  6 HYPOTENSION                                   12
    ##  7 INTRACRANIAL HEMORRHAGE                       23
    ##  8 PNEUMONIA                                     23
    ##  9 SEPSIS                                        33
    ## 10 STROKE;TELEMETRY;TRANSIENT ISCHEMIC ATTACK     9

Exploratory:
------------

I want to discuss with you guys the challenge in finding length of care -- if someone was admitted on December 1, 1980 and discharged January 5, 1981, I'm not sure if we can just subtract the times? Using Francis's method, it results in negative number for month. Also, we cannot measure minutes/hours, because admit time does not have minutes

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
