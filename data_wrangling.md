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

### linear regression

``` r
# read original data
admissions_origin <- 
  read.csv("./database/admissions.csv") %>% 
  janitor::clean_names()


# add a death factor and duration factor
admissions_death <- 
  mutate(admissions_origin, living = is.na(admissions_origin$deathtime), hospitaltime = as.duration(dischtime %--% admittime))
```

``` r
# glance data
skimr::skim(admissions_death)
```

    ## Warning in .x(x): Variable contains value(s) of "" that have been converted
    ## to "empty".

    ## Warning in .x(x): Variable contains value(s) of "" that have been converted
    ## to "empty".

    ## Warning in .x(x): Variable contains value(s) of "" that have been converted
    ## to "empty".

    ## Warning in .x(x): Variable contains value(s) of "" that have been converted
    ## to "empty".

    ## Warning in .x(x): Variable contains value(s) of "" that have been converted
    ## to "empty".

    ## Warning in .x(x): Variable contains value(s) of "" that have been converted
    ## to "empty".

    ## Warning in .x(x): Variable contains value(s) of "" that have been converted
    ## to "empty".

    ## Warning: No summary functions for vectors of class: Duration.
    ## Coercing to character

    ## Skim summary statistics
    ##  n obs: 58976 
    ##  n variables: 21 
    ## 
    ## ── Variable type:character ─────────────────────────────────────────────────────────────────────────────────────
    ##      variable missing complete     n min max empty n_unique
    ##  hospitaltime       0    58976 58976  16  24     0    26232
    ## 
    ## ── Variable type:factor ────────────────────────────────────────────────────────────────────────────────────────
    ##            variable missing complete     n n_unique
    ##  admission_location       0    58976 58976        9
    ##      admission_type       0    58976 58976        4
    ##           admittime       0    58976 58976    58651
    ##           deathtime       0    58976 58976     5835
    ##           diagnosis       0    58976 58976    15692
    ##  discharge_location       0    58976 58976       17
    ##           dischtime       0    58976 58976    58657
    ##           edouttime       0    58976 58976    30865
    ##           edregtime       0    58976 58976    30875
    ##           ethnicity       0    58976 58976       41
    ##           insurance       0    58976 58976        5
    ##            language       0    58976 58976       76
    ##      marital_status       0    58976 58976        8
    ##            religion       0    58976 58976       21
    ##                                     top_counts ordered
    ##  EME: 22754, PHY: 15079, CLI: 12032, TRA: 8456   FALSE
    ##    EME: 42071, NEW: 7863, ELE: 7706, URG: 1336   FALSE
    ##                 210: 4, 219: 4, 210: 3, 211: 3   FALSE
    ##             emp: 53122, 210: 2, 210: 2, 210: 2   FALSE
    ##      NEW: 7823, PNE: 1566, SEP: 1184, CON: 928   FALSE
    ##   HOM: 18962, HOM: 13963, SNF: 7705, REH: 6429   FALSE
    ##                 210: 3, 212: 3, 210: 2, 210: 2   FALSE
    ##             emp: 28099, 210: 2, 210: 2, 210: 2   FALSE
    ##             emp: 28099, 210: 2, 213: 2, 213: 2   FALSE
    ##    WHI: 40996, BLA: 5440, UNK: 4523, HIS: 1696   FALSE
    ##   Med: 28215, Pri: 22582, Med: 5785, Gov: 1783   FALSE
    ##    ENG: 29086, emp: 25332, SPA: 1083, RUS: 790   FALSE
    ##  MAR: 24239, SIN: 13254, emp: 10128, WID: 7211   FALSE
    ##   CAT: 20606, NOT: 11753, UNO: 8269, PRO: 7134   FALSE
    ## 
    ## ── Variable type:integer ───────────────────────────────────────────────────────────────────────────────────────
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
    ## ── Variable type:logical ───────────────────────────────────────────────────────────────────────────────────────
    ##  variable missing complete     n mean             count
    ##    living       0    58976 58976    0 FAL: 58976, NA: 0

``` r
#admissions_death <- 
#  admissions_death %>% 
#  mutate(religion = as.character(religion), hotpitaltime = as.numeric(hospitaltime)) %>% View
# regression
living_lm <- 
  lm(as.numeric(hospitaltime) ~ as.character(religion), data = admissions_death)
summary(living_lm)
```

    ## 
    ## Call:
    ## lm(formula = as.numeric(hospitaltime) ~ as.character(religion), 
    ##     data = admissions_death)
    ## 
    ## Residuals:
    ##       Min        1Q    Median        3Q       Max 
    ## -24590505   -136099    312029    557495   1275239 
    ## 
    ## Coefficients:
    ##                                              Estimate Std. Error t value
    ## (Intercept)                                   -860003      50248 -17.115
    ## as.character(religion)7TH DAY ADVENTIST       -320276     129619  -2.471
    ## as.character(religion)BAPTIST                  -86557     209342  -0.413
    ## as.character(religion)BUDDHIST                 122942      82800   1.485
    ## as.character(religion)CATHOLIC                 -44782      50803  -0.881
    ## as.character(religion)CHRISTIAN SCIENTIST      -16679      72252  -0.231
    ## as.character(religion)EPISCOPALIAN              -8152      63395  -0.129
    ## as.character(religion)GREEK ORTHODOX           -59173      71022  -0.833
    ## as.character(religion)HEBREW                    73804     273493   0.270
    ## as.character(religion)HINDU                     80567     112953   0.713
    ## as.character(religion)JEHOVAH'S WITNESS       -108052     104135  -1.038
    ## as.character(religion)JEWISH                    45897      52368   0.876
    ## as.character(religion)LUTHERAN                 273143    1076524   0.254
    ## as.character(religion)METHODIST               -483234     409538  -1.180
    ## as.character(religion)MUSLIM                   -72222      87546  -0.825
    ## as.character(religion)NOT SPECIFIED             51351      51218   1.003
    ## as.character(religion)OTHER                    -66083      54349  -1.216
    ## as.character(religion)PROTESTANT QUAKER        -48318      51836  -0.932
    ## as.character(religion)ROMANIAN EAST. ORTH     -423396     128285  -3.300
    ## as.character(religion)UNITARIAN-UNIVERSALIST   -24639     108860  -0.226
    ## as.character(religion)UNOBTAINABLE             -26781      51621  -0.519
    ##                                              Pr(>|t|)    
    ## (Intercept)                                   < 2e-16 ***
    ## as.character(religion)7TH DAY ADVENTIST      0.013480 *  
    ## as.character(religion)BAPTIST                0.679261    
    ## as.character(religion)BUDDHIST               0.137602    
    ## as.character(religion)CATHOLIC               0.378062    
    ## as.character(religion)CHRISTIAN SCIENTIST    0.817432    
    ## as.character(religion)EPISCOPALIAN           0.897682    
    ## as.character(religion)GREEK ORTHODOX         0.404758    
    ## as.character(religion)HEBREW                 0.787272    
    ## as.character(religion)HINDU                  0.475674    
    ## as.character(religion)JEHOVAH'S WITNESS      0.299452    
    ## as.character(religion)JEWISH                 0.380803    
    ## as.character(religion)LUTHERAN               0.799708    
    ## as.character(religion)METHODIST              0.238025    
    ## as.character(religion)MUSLIM                 0.409395    
    ## as.character(religion)NOT SPECIFIED          0.316057    
    ## as.character(religion)OTHER                  0.224026    
    ## as.character(religion)PROTESTANT QUAKER      0.351274    
    ## as.character(religion)ROMANIAN EAST. ORTH    0.000966 ***
    ## as.character(religion)UNITARIAN-UNIVERSALIST 0.820938    
    ## as.character(religion)UNOBTAINABLE           0.603901    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1075000 on 58955 degrees of freedom
    ## Multiple R-squared:  0.002023,   Adjusted R-squared:  0.001684 
    ## F-statistic: 5.975 on 20 and 58955 DF,  p-value: 3.686e-16

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
