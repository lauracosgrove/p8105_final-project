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
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## ── Conflicts ────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
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
# The year should be delt with to become normal. The discharge time means the time when the patient leave the hospital.


# see types
class(admissions$admittime)
```

    ## [1] "factor"

``` r
class(admissions$dischtime)
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
  separate(dischtime, into = c("dischtime_year", "dischtime_month", "dischtime_day"), sep = "-") %>% 
  separate(dischtime_day, into = c("dischtime_day", "dischtime_time"), sep = " ")
admissions <- 
  admissions %>% 
  separate(deathtime, into = c("deathtime_year", "deathtime_month", "deathtime_day"), sep = "-") %>% 
  separate(deathtime_day, into = c("deathtime_day", "deathtime_time"), sep = " ")
```

    ## Warning: Expected 3 pieces. Missing pieces filled with `NA` in 53122
    ## rows [1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
    ## 21, ...].

``` r
admissions <- 
  admissions %>% 
  separate(edregtime, into = c("edregtime_year", "edregtime_month", "edregtime_day"), sep = "-") %>% 
  separate(edregtime_day, into = c("edregtime_day", "edregtime_time"), sep = " ")
```

    ## Warning: Expected 3 pieces. Missing pieces filled with `NA` in 28099 rows
    ## [2, 3, 4, 6, 7, 8, 9, 10, 11, 13, 15, 16, 18, 20, 21, 22, 28, 29, 35,
    ## 36, ...].

``` r
admissions <- 
  admissions %>% 
  separate(edouttime, into = c("edouttime_year", "edouttime_month", "edouttime_day"), sep = "-") %>% 
  separate(edouttime_day, into = c("edouttime_day", "edouttime_time"), sep = " ")
```

    ## Warning: Expected 3 pieces. Missing pieces filled with `NA` in 28099 rows
    ## [2, 3, 4, 6, 7, 8, 9, 10, 11, 13, 15, 16, 18, 20, 21, 22, 28, 29, 35,
    ## 36, ...].

``` r
# Correct year to normal
admissions <- 
  admissions %>% 
  mutate(admittime_year = as.numeric(admittime_year) - 200, dischtime_year = as.numeric(dischtime_year) - 200, deathtime_year = as.numeric(deathtime_year) - 200, edregtime_year = as.numeric(edregtime_year) - 200, edouttime_year = as.numeric(edouttime_year) - 200)
# Take a look
head(admissions)
```

    ##   row_id subject_id hadm_id admittime_year admittime_month admittime_day
    ## 1     21         22  165315           1996              04   09 12:26:00
    ## 2     22         23  152223           1953              09   03 07:15:00
    ## 3     23         23  124321           1957              10   18 19:34:00
    ## 4     24         24  161859           1939              06   06 16:14:00
    ## 5     25         25  129635           1960              11   02 02:06:00
    ## 6     26         26  197661           1926              05   06 15:16:00
    ##   dischtime_year dischtime_month dischtime_day dischtime_time
    ## 1           1996              04            10       15:54:00
    ## 2           1953              09            08       19:10:00
    ## 3           1957              10            25       14:00:00
    ## 4           1939              06            09       12:48:00
    ## 5           1960              11            05       14:55:00
    ## 6           1926              05            13       15:00:00
    ##   deathtime_year deathtime_month deathtime_day deathtime_time
    ## 1             NA            <NA>          <NA>           <NA>
    ## 2             NA            <NA>          <NA>           <NA>
    ## 3             NA            <NA>          <NA>           <NA>
    ## 4             NA            <NA>          <NA>           <NA>
    ## 5             NA            <NA>          <NA>           <NA>
    ## 6             NA            <NA>          <NA>           <NA>
    ##   admission_type        admission_location        discharge_location
    ## 1      EMERGENCY      EMERGENCY ROOM ADMIT DISC-TRAN CANCER/CHLDRN H
    ## 2       ELECTIVE PHYS REFERRAL/NORMAL DELI          HOME HEALTH CARE
    ## 3      EMERGENCY TRANSFER FROM HOSP/EXTRAM          HOME HEALTH CARE
    ## 4      EMERGENCY TRANSFER FROM HOSP/EXTRAM                      HOME
    ## 5      EMERGENCY      EMERGENCY ROOM ADMIT                      HOME
    ## 6      EMERGENCY TRANSFER FROM HOSP/EXTRAM                      HOME
    ##   insurance language          religion marital_status
    ## 1   Private               UNOBTAINABLE        MARRIED
    ## 2  Medicare                   CATHOLIC        MARRIED
    ## 3  Medicare     ENGL          CATHOLIC        MARRIED
    ## 4   Private          PROTESTANT QUAKER         SINGLE
    ## 5   Private               UNOBTAINABLE        MARRIED
    ## 6  Medicare                   CATHOLIC         SINGLE
    ##               ethnicity edregtime_year edregtime_month edregtime_day
    ## 1                 WHITE           1996              04            09
    ## 2                 WHITE             NA            <NA>          <NA>
    ## 3                 WHITE             NA            <NA>          <NA>
    ## 4                 WHITE             NA            <NA>          <NA>
    ## 5                 WHITE           1960              11            02
    ## 6 UNKNOWN/NOT SPECIFIED             NA            <NA>          <NA>
    ##   edregtime_time edouttime_year edouttime_month edouttime_day
    ## 1       10:06:00           1996              04            09
    ## 2           <NA>             NA            <NA>          <NA>
    ## 3           <NA>             NA            <NA>          <NA>
    ## 4           <NA>             NA            <NA>          <NA>
    ## 5       01:01:00           1960              11            02
    ## 6           <NA>             NA            <NA>          <NA>
    ##   edouttime_time                                                 diagnosis
    ## 1       13:24:00                                   BENZODIAZEPINE OVERDOSE
    ## 2           <NA> CORONARY ARTERY DISEASE\\CORONARY ARTERY BYPASS GRAFT/SDA
    ## 3           <NA>                                                BRAIN MASS
    ## 4           <NA>                            INTERIOR MYOCARDIAL INFARCTION
    ## 5       04:27:00                                   ACUTE CORONARY SYNDROME
    ## 6           <NA>                                                    V-TACH
    ##   hospital_expire_flag has_chartevents_data
    ## 1                    0                    1
    ## 2                    0                    1
    ## 3                    0                    1
    ## 4                    0                    1
    ## 5                    0                    1
    ## 6                    0                    1

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
