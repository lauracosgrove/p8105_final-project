data\_wrangling
================
Francis
11/10/2018

After consideration, the `admissions.csv` database seems really interesting. It is useful to analyze the connection between mutiple factors and death.

``` r
library(tidyverse)
```

    ## ── Attaching packages ──────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.7
    ## ✔ tidyr   0.8.2     ✔ stringr 1.3.1
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## ── Conflicts ─────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
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

    ##   row_id subject_id hadm_id admittime_year admittime_month admittime_day
    ## 1     83         82  110641           1950              06            24
    ## 2     84         83  158569           1942              04            01
    ## 3     85         84  120969           1996              02            02
    ## 4     86         84  166401           1996              04            14
    ## 5     87         85  116630           1962              03            02
    ## 6     88         85  112077           1967              07            25
    ##   dischtime_year dischtime_month dischtime_day dischtime_time
    ## 1           1950              06            29       15:00:00
    ## 2           1942              04            08       14:46:00
    ## 3           1996              02            04       17:48:00
    ## 4           1996              04            17       13:42:00
    ## 5           1962              03            10       13:15:00
    ## 6           1967              07            30       15:24:00
    ##   deathtime_year deathtime_month deathtime_day deathtime_time
    ## 1             NA            <NA>          <NA>           <NA>
    ## 2             NA            <NA>          <NA>           <NA>
    ## 3             NA            <NA>          <NA>           <NA>
    ## 4           1996              04            17       13:42:00
    ## 5             NA            <NA>          <NA>           <NA>
    ## 6             NA            <NA>          <NA>           <NA>
    ##   admission_type        admission_location       discharge_location
    ## 1        NEWBORN PHYS REFERRAL/NORMAL DELI                     HOME
    ## 2         URGENT TRANSFER FROM HOSP/EXTRAM         HOME HEALTH CARE
    ## 3       ELECTIVE PHYS REFERRAL/NORMAL DELI                     HOME
    ## 4      EMERGENCY      EMERGENCY ROOM ADMIT             DEAD/EXPIRED
    ## 5      EMERGENCY CLINIC REFERRAL/PREMATURE REHAB/DISTINCT PART HOSP
    ## 6      EMERGENCY CLINIC REFERRAL/PREMATURE                      SNF
    ##   insurance language     religion marital_status             ethnicity
    ## 1   Private     <NA> UNOBTAINABLE           <NA>                 OTHER
    ## 2  Medicare     <NA> UNOBTAINABLE        MARRIED UNKNOWN/NOT SPECIFIED
    ## 3   Private     <NA>        OTHER        MARRIED                 WHITE
    ## 4   Private     <NA>        OTHER        MARRIED                 WHITE
    ## 5  Medicare     ENGL     CATHOLIC        MARRIED                 WHITE
    ## 6  Medicare     ENGL     CATHOLIC        MARRIED                 WHITE
    ##   edregtime_year edregtime_month edregtime_day edregtime_time
    ## 1             NA            <NA>          <NA>           <NA>
    ## 2             NA            <NA>          <NA>           <NA>
    ## 3             NA            <NA>          <NA>           <NA>
    ## 4           1996              04            13       22:23:00
    ## 5             NA            <NA>          <NA>           <NA>
    ## 6           1967              07            25       16:37:00
    ##   edouttime_year edouttime_month edouttime_day edouttime_time
    ## 1             NA            <NA>          <NA>           <NA>
    ## 2             NA            <NA>          <NA>           <NA>
    ## 3             NA            <NA>          <NA>           <NA>
    ## 4           1996              04            14       04:31:00
    ## 5             NA            <NA>          <NA>           <NA>
    ## 6           1967              07            25       20:46:00
    ##                       diagnosis hospital_expire_flag has_chartevents_data
    ## 1                       NEWBORN                    0                    1
    ## 2              CAROTID STENOSIS                    0                    1
    ## 3     MEDIAL PARIETAL TUMOR/SDA                    0                    0
    ## 4           GLIOBLASTOMA,NAUSEA                    1                    1
    ## 5 AORTIC STENOSIS\\CARDIAC CATH                    0                    1
    ## 6                     PNEUMONIA                    0                    1

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
