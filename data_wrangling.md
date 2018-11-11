data\_wrangling
================
Francis
11/10/2018

After consideration, the `admissions.csv` database seems really interesting. It is useful to analyze the connection between mutiple factors and death.

``` r
library(tidyverse)
```

    ## ── Attaching packages ───────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.7
    ## ✔ tidyr   0.8.2     ✔ stringr 1.3.1
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## ── Conflicts ──────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(devtools)
```

MIMIC3
======

### Import data

``` r
admissions <- 
  read.csv("./database/MIMIC3/admissions.csv") %>% 
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
# The year should be delt with to become normal. Since the discharge time equals to death time, we transfered for easier discussion.
admissions <- 
  admissions %>% 
  mutate(deathtime = dischtime)

# see types
class(admissions$admittime)
```

    ## [1] "factor"

``` r
class(admissions$deathtime)
```

    ## [1] "factor"

``` r
class(admissions$admission_type) 
```

    ## [1] "factor"

``` r
class(admissions$insurance)
```

    ## [1] "factor"

``` r
class(admissions$religion)
```

    ## [1] "factor"

``` r
class(admissions$ethnicity)
```

    ## [1] "factor"

``` r
class(admissions$edregtime)
```

    ## [1] "factor"

``` r
class(admissions$edouttime)
```

    ## [1] "factor"

All of them are factors.

### Tidy data

``` r
#Create year, month, day variables
admissions <- 
  admissions %>% 
  separate(admittime, into = c("admittime_year", "admittime_month", "admittime_day"), sep = "-")
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
  mutate(admittime_year = as.numeric(admittime_year) - 200, deathtime_year = as.numeric(deathtime_year) - 200, edregtime_year = as.numeric(edregtime_year) - 200, edouttime_year = as.numeric(edouttime_year) - 200)
# Take a look
head(admissions)
```

    ##   row_id subject_id hadm_id admittime_year admittime_month admittime_day
    ## 1     83         82  110641           1950              06            24
    ## 2     84         83  158569           1942              04            01
    ## 3     85         84  120969           1996              02            02
    ## 4     86         84  166401           1996              04            14
    ## 5     87         85  116630           1962              03            02
    ## 6     88         85  112077           1967              07            25
    ##             dischtime deathtime_year deathtime_month deathtime_day
    ## 1 2150-06-29 15:00:00           1950              06            29
    ## 2 2142-04-08 14:46:00           1942              04            08
    ## 3 2196-02-04 17:48:00           1996              02            04
    ## 4 2196-04-17 13:42:00           1996              04            17
    ## 5 2162-03-10 13:15:00           1962              03            10
    ## 6 2167-07-30 15:24:00           1967              07            30
    ##   deathtime_time admission_type        admission_location
    ## 1       15:00:00        NEWBORN PHYS REFERRAL/NORMAL DELI
    ## 2       14:46:00         URGENT TRANSFER FROM HOSP/EXTRAM
    ## 3       17:48:00       ELECTIVE PHYS REFERRAL/NORMAL DELI
    ## 4       13:42:00      EMERGENCY      EMERGENCY ROOM ADMIT
    ## 5       13:15:00      EMERGENCY CLINIC REFERRAL/PREMATURE
    ## 6       15:24:00      EMERGENCY CLINIC REFERRAL/PREMATURE
    ##         discharge_location insurance language     religion marital_status
    ## 1                     HOME   Private     <NA> UNOBTAINABLE           <NA>
    ## 2         HOME HEALTH CARE  Medicare     <NA> UNOBTAINABLE        MARRIED
    ## 3                     HOME   Private     <NA>        OTHER        MARRIED
    ## 4             DEAD/EXPIRED   Private     <NA>        OTHER        MARRIED
    ## 5 REHAB/DISTINCT PART HOSP  Medicare     ENGL     CATHOLIC        MARRIED
    ## 6                      SNF  Medicare     ENGL     CATHOLIC        MARRIED
    ##               ethnicity edregtime_year edregtime_month edregtime_day
    ## 1                 OTHER             NA            <NA>          <NA>
    ## 2 UNKNOWN/NOT SPECIFIED             NA            <NA>          <NA>
    ## 3                 WHITE             NA            <NA>          <NA>
    ## 4                 WHITE           1996              04            13
    ## 5                 WHITE             NA            <NA>          <NA>
    ## 6                 WHITE           1967              07            25
    ##   edregtime_time edouttime_year edouttime_month edouttime_day
    ## 1           <NA>             NA            <NA>          <NA>
    ## 2           <NA>             NA            <NA>          <NA>
    ## 3           <NA>             NA            <NA>          <NA>
    ## 4       22:23:00           1996              04            14
    ## 5           <NA>             NA            <NA>          <NA>
    ## 6       16:37:00           1967              07            25
    ##   edouttime_time                     diagnosis hospital_expire_flag
    ## 1           <NA>                       NEWBORN                    0
    ## 2           <NA>              CAROTID STENOSIS                    0
    ## 3           <NA>     MEDIAL PARIETAL TUMOR/SDA                    0
    ## 4       04:31:00           GLIOBLASTOMA,NAUSEA                    1
    ## 5           <NA> AORTIC STENOSIS\\CARDIAC CATH                    0
    ## 6       20:46:00                     PNEUMONIA                    0
    ##   has_chartevents_data
    ## 1                    1
    ## 2                    1
    ## 3                    0
    ## 4                    1
    ## 5                    1
    ## 6                    1

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
