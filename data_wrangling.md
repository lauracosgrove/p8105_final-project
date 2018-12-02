data\_wrangling
================
Francis
11/10/2018

After consideration, the `admissions.csv` database seems really interesting. It is useful to analyze the connection between mutiple factors and death.

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.8
    ## ✔ tidyr   0.8.2     ✔ stringr 1.3.1
    ## ✔ readr   1.2.1     ✔ forcats 0.3.0

    ## ── Conflicts ────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(devtools)
library(readr)
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
knitr::opts_chunk$set(echo = TRUE)
```

MIMIC3
======

### Import data

``` r
admissions <- 
  read_csv("./database/admissions.csv") %>% 
  janitor::clean_names()
```

    ## Parsed with column specification:
    ## cols(
    ##   ROW_ID = col_double(),
    ##   SUBJECT_ID = col_double(),
    ##   HADM_ID = col_double(),
    ##   ADMITTIME = col_datetime(format = ""),
    ##   DISCHTIME = col_datetime(format = ""),
    ##   DEATHTIME = col_datetime(format = ""),
    ##   ADMISSION_TYPE = col_character(),
    ##   ADMISSION_LOCATION = col_character(),
    ##   DISCHARGE_LOCATION = col_character(),
    ##   INSURANCE = col_character(),
    ##   LANGUAGE = col_character(),
    ##   RELIGION = col_character(),
    ##   MARITAL_STATUS = col_character(),
    ##   ETHNICITY = col_character(),
    ##   EDREGTIME = col_datetime(format = ""),
    ##   EDOUTTIME = col_datetime(format = ""),
    ##   DIAGNOSIS = col_character(),
    ##   HOSPITAL_EXPIRE_FLAG = col_double(),
    ##   HAS_CHARTEVENTS_DATA = col_double()
    ## )

``` r
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
# The year should be delt with to become normal. The discharge time means the time when the patient leave the hospital.


# see types
class(admissions$admittime)
```

    ## [1] "POSIXct" "POSIXt"

``` r
class(admissions$dischtime)
```

    ## [1] "POSIXct" "POSIXt"

``` r
class(admissions$deathtime)
```

    ## [1] "POSIXct" "POSIXt"

``` r
class(admissions$admission_type) 
```

    ## [1] "character"

``` r
class(admissions$insurance)
```

    ## [1] "character"

``` r
class(admissions$religion)
```

    ## [1] "character"

``` r
class(admissions$ethnicity)
```

    ## [1] "character"

``` r
class(admissions$edregtime)
```

    ## [1] "POSIXct" "POSIXt"

``` r
class(admissions$edouttime)
```

    ## [1] "POSIXct" "POSIXt"

All of them are factors.

### Tidy data

``` r
#Create year, month, day variables
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
# Take a look
head(admissions)
```

    ## # A tibble: 6 x 33
    ##   row_id subject_id hadm_id admittime_year admittime_month admittime_day
    ##    <dbl>      <dbl>   <dbl>          <dbl> <chr>           <chr>        
    ## 1     21         22  165315           1996 04              09 12:26:00  
    ## 2     22         23  152223           1953 09              03 07:15:00  
    ## 3     23         23  124321           1957 10              18 19:34:00  
    ## 4     24         24  161859           1939 06              06 16:14:00  
    ## 5     25         25  129635           1960 11              02 02:06:00  
    ## 6     26         26  197661           1926 05              06 15:16:00  
    ## # ... with 27 more variables: dischtime_year <dbl>, dischtime_month <chr>,
    ## #   dischtime_day <chr>, dischtime_time <chr>, deathtime_year <dbl>,
    ## #   deathtime_month <chr>, deathtime_day <chr>, deathtime_time <chr>,
    ## #   admission_type <chr>, admission_location <chr>,
    ## #   discharge_location <chr>, insurance <chr>, language <chr>,
    ## #   religion <chr>, marital_status <chr>, ethnicity <chr>,
    ## #   edregtime_year <dbl>, edregtime_month <chr>, edregtime_day <chr>,
    ## #   edregtime_time <chr>, edouttime_year <dbl>, edouttime_month <chr>,
    ## #   edouttime_day <chr>, edouttime_time <chr>, diagnosis <chr>,
    ## #   hospital_expire_flag <dbl>, has_chartevents_data <dbl>

### linear regression

``` r
# read original data
admissions_origin <- 
  read_csv("./database/admissions.csv") %>% 
  janitor::clean_names()
```

    ## Parsed with column specification:
    ## cols(
    ##   ROW_ID = col_double(),
    ##   SUBJECT_ID = col_double(),
    ##   HADM_ID = col_double(),
    ##   ADMITTIME = col_datetime(format = ""),
    ##   DISCHTIME = col_datetime(format = ""),
    ##   DEATHTIME = col_datetime(format = ""),
    ##   ADMISSION_TYPE = col_character(),
    ##   ADMISSION_LOCATION = col_character(),
    ##   DISCHARGE_LOCATION = col_character(),
    ##   INSURANCE = col_character(),
    ##   LANGUAGE = col_character(),
    ##   RELIGION = col_character(),
    ##   MARITAL_STATUS = col_character(),
    ##   ETHNICITY = col_character(),
    ##   EDREGTIME = col_datetime(format = ""),
    ##   EDOUTTIME = col_datetime(format = ""),
    ##   DIAGNOSIS = col_character(),
    ##   HOSPITAL_EXPIRE_FLAG = col_double(),
    ##   HAS_CHARTEVENTS_DATA = col_double()
    ## )

``` r
# add a death factor and duration factor
admissions_death <- 
  mutate(admissions_origin, living = is.na(admissions_origin$deathtime), hospitaltime =  admissions_origin$dischtime - admissions_origin$admittime, edtime = admissions_origin$edouttime - admissions_origin$edregtime)
```

``` r
# glance data
skimr::skim(admissions_death)
```

    ## Skim summary statistics
    ##  n obs: 58976 
    ##  n variables: 22 
    ## 
    ## ── Variable type:character ─────────────────────────────────────────────────────────────────────────────────────
    ##            variable missing complete     n min max empty n_unique
    ##  admission_location       0    58976 58976  17  25     0        9
    ##      admission_type       0    58976 58976   6   9     0        4
    ##           diagnosis      25    58951 58976   2 190     0    15646
    ##  discharge_location       0    58976 58976   3  25     0       17
    ##           ethnicity       0    58976 58976   5  56     0       41
    ##           insurance       0    58976 58976   7  10     0        5
    ##            language   25332    33644 58976   4   4     0       75
    ##      marital_status   10128    48848 58976   6  17     0        7
    ##            religion     458    58518 58976   5  22     0       20
    ## 
    ## ── Variable type:difftime ──────────────────────────────────────────────────────────────────────────────────────
    ##      variable missing complete     n           min         max
    ##        edtime   28099    30877 58976 -2504940 secs 253680 secs
    ##  hospitaltime       0    58976 58976    -1361 mins 424311 mins
    ##             median n_unique
    ##  17700 secs            1483
    ##        9312.5 mins    26302
    ## 
    ## ── Variable type:logical ───────────────────────────────────────────────────────────────────────────────────────
    ##  variable missing complete     n mean                        count
    ##    living       0    58976 58976  0.9 TRU: 53122, FAL: 5854, NA: 0
    ## 
    ## ── Variable type:numeric ───────────────────────────────────────────────────────────────────────────────────────
    ##              variable missing complete     n       mean       sd    p0
    ##               hadm_id       0    58976 58976 149970.81  28883.1  1e+05
    ##  has_chartevents_data       0    58976 58976      0.97      0.16     0
    ##  hospital_expire_flag       0    58976 58976      0.099     0.3      0
    ##                row_id       0    58976 58976  29488.5   17025.05     1
    ##            subject_id       0    58976 58976  33755.58  28092.73     2
    ##        p25      p50       p75  p100     hist
    ##  124952.75 149989.5 174966.5  2e+05 ▇▇▇▇▇▇▇▇
    ##       1         1        1        1 ▁▁▁▁▁▁▁▇
    ##       0         0        0        1 ▇▁▁▁▁▁▁▁
    ##   14744.75  29488.5  44232.25 58976 ▇▇▇▇▇▇▇▇
    ##   11993.75  24133.5  53851.5  99999 ▇▇▅▂▂▂▂▂
    ## 
    ## ── Variable type:POSIXct ───────────────────────────────────────────────────────────────────────────────────────
    ##   variable missing complete     n        min        max     median
    ##  admittime       0    58976 58976 2100-06-07 2210-08-17 2151-01-15
    ##  deathtime   53122     5854 58976 2100-06-19 2208-02-05 2150-09-15
    ##  dischtime       0    58976 58976 2100-06-09 2210-08-24 2151-01-29
    ##  edouttime   28099    30877 58976 2100-06-08 2210-08-17 2150-12-12
    ##  edregtime   28099    30877 58976 2100-06-07 2210-08-17 2150-12-12
    ##  n_unique
    ##     58651
    ##      5834
    ##     58657
    ##     30864
    ##     30874

``` r
### try logistic regression step by step.

#SLR

living_lm1 <- 
  lm(living ~ admission_type, data = admissions_death)
summary(living_lm1)    

living_lm2 <- 
  lm(living ~ admission_location, data = admissions_death)
summary(living_lm2)

living_lm3 <- 
  lm(living ~ insurance, data = admissions_death)
summary(living_lm3)

living_lm4 <- 
  lm(living ~ language, data = na.omit(select(admissions_death, living, language)))
summary(living_lm4)

living_lm5 <- 
  lm(living ~ religion, data = admissions_death)
summary(living_lm5)

living_lm6 <- 
  lm(living ~ marital_status, data = admissions_death)
summary(living_lm6)

living_lm7 <- 
  lm(living ~ ethnicity, data = admissions_death)
summary(living_lm7)

living_lm8 <- 
  lm(living ~ diagnosis, data = admissions_death)
summary(living_lm8)

living_lm9 <- 
  lm(living ~ hospital_expire_flag, data = admissions_death)
summary(living_lm9)

living_lm10 <- 
  lm(living ~ has_chartevents_data, data = admissions_death)
summary(living_lm10)

living_lm11 <- 
  lm(living ~ hospitaltime, data = admissions_death)
summary(living_lm11)

living_lm12 <- 
  lm(living ~ edtime, data = filter(edtime > 0))
summary(living_lm12)
```

``` r
#MLR

living_mlr <- 
  lm(living ~ hospitaltime + admission_type + admission_location + insurance + language + religion + marital_status + ethnicity + edtime + hospital_expire_flag + has_chartevents_data + insurance, data = admissions_death)
summary(living_mlr)


coefficients(living_mlr)
confint(living_mlr)
fitted(living_mlr)
residuals(living_mlr)
anova(living_mlr)
vcov(living_mlr)
influence(living_mlr)

ggplot(living_mlr)
```

``` r
# K-fold cross-validation
library(DAAG)
cv.lm(df = admissions_death, living_mlr, m = 3) # 3 fold cross-validation
```

``` r
# step seems not working???
```

``` r
# All Subsets Regression
library(leaps)
attach(admissions_death)
leaps <- 
  regsubsets(living ~ hospitaltime + admission_type + admission_location + insurance + language + religion + marital_status + ethnicity + edtime + hospital_expire_flag + has_chartevents_data + insurance, data = admissions_death, nbest = 10)
# view results 
summary(leaps)
# plot a table of models showing variables in each model.
# models are ordered by the selection statistic.
plot(leaps,scale = "r2")
# plot statistic by subset size 
library(car)
subsets(leaps, statistic = "rsq")
```

``` r
# Calculate Relative Importance for Each Predictor
library(relaimpo)
calc.relimp(fit,type=c("lmg","last","first","pratt"),
   rela=TRUE)
```

``` r
# Bootstrap Measures of Relative Importance (1000 samples) 
boot <- boot.relimp(admissions_death, b = 1000, type = c("lmg", 
  "last", "first", "pratt"), rank = TRUE, 
  diff = TRUE, rela = TRUE)
booteval.relimp(boot) # print result
plot(booteval.relimp(boot,sort=TRUE)) # plot result
```

OpenFDA
=======

### Import OpenFDA

``` r
# Already installed openfda data
# Load OpenFDA
library(openfda)


library(jsonlite)
```

    ## 
    ## Attaching package: 'jsonlite'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     flatten

``` r
fda <- 
  fromJSON("https://api.fda.gov/drug/event.json") %>% 
  janitor::clean_names()

names(fda$results)
```

    ##  [1] "receiptdateformat"          "receiver"                  
    ##  [3] "companynumb"                "receivedateformat"         
    ##  [5] "primarysource"              "seriousnessother"          
    ##  [7] "transmissiondateformat"     "fulfillexpeditecriteria"   
    ##  [9] "safetyreportid"             "sender"                    
    ## [11] "receivedate"                "patient"                   
    ## [13] "seriousnesshospitalization" "transmissiondate"          
    ## [15] "serious"                    "receiptdate"

There are 16 observations in the `event` dataset under `drug`, they are: receiptdateformat, receiver, companynumb, receivedateformat, primarysource, seriousnessother, transmissiondateformat, fulfillexpeditecriteria, safetyreportid, sender, receivedate, patient, seriousnesshospitalization, transmissiondate, serious, receiptdate.
