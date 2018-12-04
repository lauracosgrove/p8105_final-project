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
library(broom)
```

MIMIC3
======

### Import data

``` r
admissions <- 
  read_csv("./database/data/admissions.csv") %>% 
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
  read_csv("./database/data/admissions.csv") %>% 
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
# exclude NEWBORN, because they are not treated as patients.
### There are too many diagnosis, so the diagnosis will be analyzed by some analysis of scores in another part.
admissions_death <- 
  mutate(admissions_origin, living = is.na(admissions_origin$deathtime), hospitaltime =  admissions_origin$dischtime - admissions_origin$admittime, edtime = admissions_origin$edouttime - admissions_origin$edregtime) %>% 
  filter(diagnosis != "NEWBORN")
```

``` r
# glance data
skimr::skim(admissions_death)
```

    ## Skim summary statistics
    ##  n obs: 51128 
    ##  n variables: 22 
    ## 
    ## ── Variable type:character ─────────────────────────────────────────────────────────────────────────────────────
    ##            variable missing complete     n min max empty n_unique
    ##  admission_location       0    51128 51128  17  25     0        9
    ##      admission_type       0    51128 51128   6   9     0        4
    ##           diagnosis       0    51128 51128   2 190     0    15645
    ##  discharge_location       0    51128 51128   3  25     0       17
    ##           ethnicity       0    51128 51128   5  56     0       41
    ##           insurance       0    51128 51128   7  10     0        5
    ##            language   17786    33342 51128   4   4     0       75
    ##      marital_status    2605    48523 51128   6  17     0        7
    ##            religion     451    50677 51128   5  22     0       20
    ## 
    ## ── Variable type:difftime ──────────────────────────────────────────────────────────────────────────────────────
    ##      variable missing complete     n           min         max
    ##        edtime   20252    30876 51128 -2504940 secs 253680 secs
    ##  hospitaltime       0    51128 51128    -1361 mins 424311 mins
    ##             median n_unique
    ##  17700 secs            1483
    ##        9891.5 mins    24462
    ## 
    ## ── Variable type:logical ───────────────────────────────────────────────────────────────────────────────────────
    ##  variable missing complete     n mean                        count
    ##    living       0    51128 51128 0.89 TRU: 45337, FAL: 5791, NA: 0
    ## 
    ## ── Variable type:numeric ───────────────────────────────────────────────────────────────────────────────────────
    ##              variable missing complete     n      mean       sd    p0
    ##               hadm_id       0    51128 51128 149980.03 28920.51 1e+05
    ##  has_chartevents_data       0    51128 51128      0.97     0.17     0
    ##  hospital_expire_flag       0    51128 51128      0.11     0.32     0
    ##                row_id       0    51128 51128  31075    17208.1      2
    ##            subject_id       0    51128 51128  36524.63 28971.81     3
    ##        p25      p50       p75  p100     hist
    ##  124877.75 150082.5 175035.25 2e+05 ▇▇▇▇▇▇▇▇
    ##       1         1        1        1 ▁▁▁▁▁▁▁▇
    ##       0         0        0        1 ▇▁▁▁▁▁▁▁
    ##   16097.75  32292.5  46189.25 58976 ▆▆▆▆▇▇▇▇
    ##   13116.75  26395    59788.25 99999 ▇▇▅▂▃▂▂▂
    ## 
    ## ── Variable type:POSIXct ───────────────────────────────────────────────────────────────────────────────────────
    ##   variable missing complete     n        min        max     median
    ##  admittime       0    51128 51128 2100-06-07 2210-08-17 2151-02-24
    ##  deathtime   45337     5791 51128 2100-06-19 2208-02-05 2150-08-29
    ##  dischtime       0    51128 51128 2100-06-09 2210-08-24 2151-03-06
    ##  edouttime   20252    30876 51128 2100-06-08 2210-08-17 2150-12-12
    ##  edregtime   20252    30876 51128 2100-06-07 2210-08-17 2150-12-12
    ##  n_unique
    ##     50813
    ##      5772
    ##     50877
    ##     30863
    ##     30873

``` r
### try logistic regression step by step.

#SLR

living_lm1 <- 
  glm(living ~ admission_type, data = admissions_death)
summary(living_lm1)
```

    ## 
    ## Call:
    ## glm(formula = living ~ admission_type, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##      Min        1Q    Median        3Q       Max  
    ## -0.97438   0.02562   0.12917   0.12917   0.12917  
    ## 
    ## Coefficients:
    ##                          Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)              0.974379   0.003589 271.455   <2e-16 ***
    ## admission_typeEMERGENCY -0.103549   0.003904 -26.525   <2e-16 ***
    ## admission_typeNEWBORN    0.025621   0.047057   0.544    0.586    
    ## admission_typeURGENT    -0.095159   0.009338 -10.190   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.09906754)
    ## 
    ##     Null deviance: 5135.1  on 51127  degrees of freedom
    ## Residual deviance: 5064.7  on 51124  degrees of freedom
    ## AIC: 26895
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm1)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         5135.   51127 -13443. 26895. 26940.    5065.       51124

``` r
broom::tidy(living_lm1)
```

    ## # A tibble: 4 x 5
    ##   term                    estimate std.error statistic   p.value
    ##   <chr>                      <dbl>     <dbl>     <dbl>     <dbl>
    ## 1 (Intercept)               0.974    0.00359   271.    0.       
    ## 2 admission_typeEMERGENCY  -0.104    0.00390   -26.5   5.52e-154
    ## 3 admission_typeNEWBORN     0.0256   0.0471      0.544 5.86e-  1
    ## 4 admission_typeURGENT     -0.0952   0.00934   -10.2   2.31e- 24

``` r
living_lm2 <- 
  glm(living ~ admission_location, data = admissions_death)
summary(living_lm1)
```

    ## 
    ## Call:
    ## glm(formula = living ~ admission_type, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##      Min        1Q    Median        3Q       Max  
    ## -0.97438   0.02562   0.12917   0.12917   0.12917  
    ## 
    ## Coefficients:
    ##                          Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)              0.974379   0.003589 271.455   <2e-16 ***
    ## admission_typeEMERGENCY -0.103549   0.003904 -26.525   <2e-16 ***
    ## admission_typeNEWBORN    0.025621   0.047057   0.544    0.586    
    ## admission_typeURGENT    -0.095159   0.009338 -10.190   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.09906754)
    ## 
    ##     Null deviance: 5135.1  on 51127  degrees of freedom
    ## Residual deviance: 5064.7  on 51124  degrees of freedom
    ## AIC: 26895
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm2)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         5135.   51127 -13454. 26927. 27016.    5067.       51119

``` r
broom::tidy(living_lm2)
```

    ## # A tibble: 9 x 5
    ##   term                             estimate std.error statistic    p.value
    ##   <chr>                               <dbl>     <dbl>     <dbl>      <dbl>
    ## 1 (Intercept)                        0.8        0.141    5.68      1.34e-8
    ## 2 admission_locationCLINIC REFERR…   0.0907     0.141    0.644     5.20e-1
    ## 3 admission_locationEMERGENCY ROO…   0.0641     0.141    0.455     6.49e-1
    ## 4 admission_locationHMO REFERRAL/…   0.200      0.345    0.580     5.62e-1
    ## 5 admission_locationPHYS REFERRAL…   0.159      0.141    1.13      2.60e-1
    ## 6 admission_locationTRANSFER FROM…   0.0655     0.141    0.465     6.42e-1
    ## 7 admission_locationTRANSFER FROM…  -0.0676     0.146   -0.464     6.43e-1
    ## 8 admission_locationTRANSFER FROM…   0.0125     0.142    0.0880    9.30e-1
    ## 9 admission_locationTRSF WITHIN T…   0.200      0.199    1.00      3.15e-1

``` r
living_lm3 <- 
  glm(living ~ insurance, data = admissions_death)
summary(living_lm3)
```

    ## 
    ## Call:
    ## glm(formula = living ~ insurance, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##      Min        1Q    Median        3Q       Max  
    ## -0.93585   0.08088   0.08249   0.13848   0.16376  
    ## 
    ## Coefficients:
    ##                    Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)        0.935852   0.008425 111.084  < 2e-16 ***
    ## insuranceMedicaid -0.018342   0.009637  -1.903   0.0570 .  
    ## insuranceMedicare -0.074337   0.008632  -8.612  < 2e-16 ***
    ## insurancePrivate  -0.016732   0.008778  -1.906   0.0566 .  
    ## insuranceSelf Pay -0.099615   0.015635  -6.371 1.89e-10 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.0995792)
    ## 
    ##     Null deviance: 5135.1  on 51127  degrees of freedom
    ## Residual deviance: 5090.8  on 51123  degrees of freedom
    ## AIC: 27160
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm3)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         5135.   51127 -13574. 27160. 27213.    5091.       51123

``` r
broom::tidy(living_lm3)
```

    ## # A tibble: 5 x 5
    ##   term              estimate std.error statistic  p.value
    ##   <chr>                <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)         0.936    0.00842    111.   0.      
    ## 2 insuranceMedicaid  -0.0183   0.00964     -1.90 5.70e- 2
    ## 3 insuranceMedicare  -0.0743   0.00863     -8.61 7.38e-18
    ## 4 insurancePrivate   -0.0167   0.00878     -1.91 5.66e- 2
    ## 5 insuranceSelf Pay  -0.0996   0.0156      -6.37 1.89e-10

``` r
living_lm4 <- 
  glm(living ~ language, data = na.omit(select(admissions_death, living, language)))
summary(living_lm4)
```

    ## 
    ## Call:
    ## glm(formula = living ~ language, data = na.omit(select(admissions_death, 
    ##     living, language)))
    ## 
    ## Deviance Residuals: 
    ##      Min        1Q    Median        3Q       Max  
    ## -0.95745   0.08662   0.08662   0.08662   0.66667  
    ## 
    ## Coefficients:
    ##                Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   1.000e+00  2.031e-01   4.924 8.51e-07 ***
    ## language* FU  3.627e-12  3.517e-01   0.000  1.00000    
    ## language** T  3.713e-12  3.517e-01   0.000  1.00000    
    ## language**SH  3.621e-12  2.872e-01   0.000  1.00000    
    ## language**TO -6.667e-01  2.622e-01  -2.543  0.01100 *  
    ## language*AMH -5.000e-01  2.872e-01  -1.741  0.08169 .  
    ## language*ARA  3.528e-12  3.517e-01   0.000  1.00000    
    ## language*ARM -4.444e-01  2.245e-01  -1.980  0.04775 *  
    ## language*BEN -1.429e-01  2.303e-01  -0.620  0.53500    
    ## language*BOS  3.754e-12  3.517e-01   0.000  1.00000    
    ## language*BUL  3.599e-12  2.403e-01   0.000  1.00000    
    ## language*BUR -2.500e-01  2.487e-01  -1.005  0.31482    
    ## language*CAN  3.621e-12  2.872e-01   0.000  1.00000    
    ## language*CDI -2.500e-01  2.487e-01  -1.005  0.31482    
    ## language*CHI  3.666e-12  2.303e-01   0.000  1.00000    
    ## language*CRE  3.816e-12  3.517e-01   0.000  1.00000    
    ## language*DEA  3.691e-12  3.517e-01   0.000  1.00000    
    ## language*DUT  3.734e-12  2.872e-01   0.000  1.00000    
    ## language*FAR  3.801e-12  3.517e-01   0.000  1.00000    
    ## language*FIL  3.779e-12  3.517e-01   0.000  1.00000    
    ## language*FUL -5.000e-01  2.872e-01  -1.741  0.08169 .  
    ## language*GUJ -1.429e-01  2.303e-01  -0.620  0.53500    
    ## language*HUN -1.429e-01  2.303e-01  -0.620  0.53500    
    ## language*IBO  3.496e-12  2.487e-01   0.000  1.00000    
    ## language*KHM  3.579e-12  2.872e-01   0.000  1.00000    
    ## language*LEB  3.715e-12  2.872e-01   0.000  1.00000    
    ## language*LIT  3.663e-12  3.517e-01   0.000  1.00000    
    ## language*MAN  3.532e-12  2.622e-01   0.000  1.00000    
    ## language*MOR  3.589e-12  2.872e-01   0.000  1.00000    
    ## language*NEP  3.550e-12  3.517e-01   0.000  1.00000    
    ## language*PER  3.806e-12  3.517e-01   0.000  1.00000    
    ## language*PHI  3.586e-12  3.517e-01   0.000  1.00000    
    ## language*PUN  3.808e-12  3.517e-01   0.000  1.00000    
    ## language*ROM -1.000e+00  3.517e-01  -2.843  0.00447 ** 
    ## language*RUS -1.000e+00  3.517e-01  -2.843  0.00447 ** 
    ## language*SPA  3.708e-12  3.517e-01   0.000  1.00000    
    ## language*TAM  3.602e-12  3.517e-01   0.000  1.00000    
    ## language*TEL -3.333e-01  2.622e-01  -1.271  0.20358    
    ## language*TOI  3.635e-12  2.872e-01   0.000  1.00000    
    ## language*TOY -5.000e-01  2.872e-01  -1.741  0.08169 .  
    ## language*URD -5.000e-01  2.487e-01  -2.010  0.04440 *  
    ## language*YID -2.857e-01  2.303e-01  -1.241  0.21469    
    ## language*YOR  3.407e-12  3.517e-01   0.000  1.00000    
    ## languageALBA -1.176e-01  2.147e-01  -0.548  0.58370    
    ## languageAMER -9.677e-02  2.095e-01  -0.462  0.64417    
    ## languageARAB -4.255e-02  2.074e-01  -0.205  0.83740    
    ## languageBENG  3.769e-12  2.872e-01   0.000  1.00000    
    ## languageCAMB -1.351e-01  2.085e-01  -0.648  0.51689    
    ## languageCANT -1.522e-01  2.036e-01  -0.748  0.45467    
    ## languageCAPE -8.696e-02  2.040e-01  -0.426  0.66986    
    ## languageENGL -8.662e-02  2.031e-01  -0.427  0.66972    
    ## languageETHI  3.781e-12  2.208e-01   0.000  1.00000    
    ## languageFREN -2.500e-01  2.154e-01  -1.161  0.24579    
    ## languageGERM -1.000e+00  3.517e-01  -2.843  0.00447 ** 
    ## languageGREE -1.333e-01  2.058e-01  -0.648  0.51700    
    ## languageHAIT -7.333e-02  2.044e-01  -0.359  0.71980    
    ## languageHIND -1.667e-01  2.114e-01  -0.789  0.43040    
    ## languageITAL -6.452e-02  2.047e-01  -0.315  0.75264    
    ## languageJAPA -3.333e-01  2.622e-01  -1.271  0.20358    
    ## languageKORE -1.304e-01  2.117e-01  -0.616  0.53785    
    ## languageLAOT -2.857e-01  2.303e-01  -1.241  0.21469    
    ## languageMAND -1.349e-01  2.047e-01  -0.659  0.50979    
    ## languagePERS -6.818e-02  2.076e-01  -0.328  0.74264    
    ## languagePOLI -8.824e-02  2.090e-01  -0.422  0.67284    
    ## languagePORT -8.833e-02  2.037e-01  -0.434  0.66459    
    ## languagePTUN -1.834e-01  2.034e-01  -0.902  0.36714    
    ## languageRUSS -1.510e-01  2.033e-01  -0.743  0.45767    
    ## languageSERB  3.851e-12  3.517e-01   0.000  1.00000    
    ## languageSOMA -7.692e-02  2.181e-01  -0.353  0.72436    
    ## languageSPAN -8.045e-02  2.033e-01  -0.396  0.69227    
    ## languageTAGA -3.333e-01  2.622e-01  -1.271  0.20358    
    ## languageTHAI  3.737e-12  2.193e-01   0.000  1.00000    
    ## languageTURK  3.715e-12  2.872e-01   0.000  1.00000    
    ## languageURDU  3.630e-12  2.403e-01   0.000  1.00000    
    ## languageVIET -1.011e-01  2.053e-01  -0.492  0.62240    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.08247903)
    ## 
    ##     Null deviance: 2763.6  on 33341  degrees of freedom
    ## Residual deviance: 2743.8  on 33267  degrees of freedom
    ## AIC: 11502
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm4)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>  <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         2764.   33341 -5675. 11502. 12142.    2744.       33267

``` r
broom::tidy(living_lm4)
```

    ## # A tibble: 75 x 5
    ##    term           estimate std.error statistic     p.value
    ##    <chr>             <dbl>     <dbl>     <dbl>       <dbl>
    ##  1 (Intercept)   10.00e- 1     0.203  4.92e+ 0 0.000000851
    ##  2 language* FU   3.63e-12     0.352  1.03e-11 1.000      
    ##  3 language** T   3.71e-12     0.352  1.06e-11 1.000      
    ##  4 language**SH   3.62e-12     0.287  1.26e-11 1.000      
    ##  5 language**TO  -6.67e- 1     0.262 -2.54e+ 0 0.0110     
    ##  6 language*AMH  -5.00e- 1     0.287 -1.74e+ 0 0.0817     
    ##  7 language*ARA   3.53e-12     0.352  1.00e-11 1.000      
    ##  8 language*ARM  -4.44e- 1     0.225 -1.98e+ 0 0.0478     
    ##  9 language*BEN  -1.43e- 1     0.230 -6.20e- 1 0.535      
    ## 10 language*BOS   3.75e-12     0.352  1.07e-11 1.000      
    ## # ... with 65 more rows

``` r
living_lm5 <- 
  glm(living ~ religion, data = admissions_death)
summary(living_lm5)
```

    ## 
    ## Call:
    ## glm(formula = living ~ religion, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##      Min        1Q    Median        3Q       Max  
    ## -0.92896   0.08981   0.10294   0.10294   0.28571  
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                     0.9285714  0.0375567  24.725  < 2e-16 ***
    ## religionBAPTIST                -0.2142857  0.0702621  -3.050  0.00229 ** 
    ## religionBUDDHIST               -0.0177733  0.0432903  -0.411  0.68140    
    ## religionCATHOLIC               -0.0315102  0.0376275  -0.837  0.40236    
    ## religionCHRISTIAN SCIENTIST    -0.0216645  0.0413159  -0.524  0.60003    
    ## religionEPISCOPALIAN           -0.0233380  0.0393720  -0.593  0.55335    
    ## religionGREEK ORTHODOX         -0.0184059  0.0405453  -0.454  0.64986    
    ## religionHEBREW                 -0.1785714  0.0870716  -2.051  0.04029 *  
    ## religionHINDU                  -0.0553320  0.0529258  -1.045  0.29581    
    ## religionJEHOVAH'S WITNESS      -0.0545557  0.0467755  -1.166  0.24349    
    ## religionJEWISH                 -0.0660766  0.0378263  -1.747  0.08067 .  
    ## religionLUTHERAN                0.0714286  0.3164581   0.226  0.82143    
    ## religionMETHODIST              -0.0714286  0.1245614  -0.573  0.56635    
    ## religionMUSLIM                  0.0003903  0.0441593   0.009  0.99295    
    ## religionNOT SPECIFIED          -0.0183831  0.0376834  -0.488  0.62567    
    ## religionOTHER                  -0.0165017  0.0380980  -0.433  0.66492    
    ## religionPROTESTANT QUAKER      -0.0273875  0.0377557  -0.725  0.46822    
    ## religionROMANIAN EAST. ORTH    -0.0903361  0.0535023  -1.688  0.09133 .  
    ## religionUNITARIAN-UNIVERSALIST -0.0983827  0.0483939  -2.033  0.04206 *  
    ## religionUNOBTAINABLE           -0.1234699  0.0377921  -3.267  0.00109 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.09873523)
    ## 
    ##     Null deviance: 5055.0  on 50676  degrees of freedom
    ## Residual deviance: 5001.6  on 50657  degrees of freedom
    ##   (451 observations deleted due to missingness)
    ## AIC: 26504
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm5)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         5055.   50676 -13231. 26504. 26689.    5002.       50657

``` r
broom::tidy(living_lm5)
```

    ## # A tibble: 20 x 5
    ##    term                            estimate std.error statistic   p.value
    ##    <chr>                              <dbl>     <dbl>     <dbl>     <dbl>
    ##  1 (Intercept)                     0.929       0.0376  24.7     3.65e-134
    ##  2 religionBAPTIST                -0.214       0.0703  -3.05    2.29e-  3
    ##  3 religionBUDDHIST               -0.0178      0.0433  -0.411   6.81e-  1
    ##  4 religionCATHOLIC               -0.0315      0.0376  -0.837   4.02e-  1
    ##  5 religionCHRISTIAN SCIENTIST    -0.0217      0.0413  -0.524   6.00e-  1
    ##  6 religionEPISCOPALIAN           -0.0233      0.0394  -0.593   5.53e-  1
    ##  7 religionGREEK ORTHODOX         -0.0184      0.0405  -0.454   6.50e-  1
    ##  8 religionHEBREW                 -0.179       0.0871  -2.05    4.03e-  2
    ##  9 religionHINDU                  -0.0553      0.0529  -1.05    2.96e-  1
    ## 10 religionJEHOVAH'S WITNESS      -0.0546      0.0468  -1.17    2.43e-  1
    ## 11 religionJEWISH                 -0.0661      0.0378  -1.75    8.07e-  2
    ## 12 religionLUTHERAN                0.0714      0.316    0.226   8.21e-  1
    ## 13 religionMETHODIST              -0.0714      0.125   -0.573   5.66e-  1
    ## 14 religionMUSLIM                  0.000390    0.0442   0.00884 9.93e-  1
    ## 15 religionNOT SPECIFIED          -0.0184      0.0377  -0.488   6.26e-  1
    ## 16 religionOTHER                  -0.0165      0.0381  -0.433   6.65e-  1
    ## 17 religionPROTESTANT QUAKER      -0.0274      0.0378  -0.725   4.68e-  1
    ## 18 religionROMANIAN EAST. ORTH    -0.0903      0.0535  -1.69    9.13e-  2
    ## 19 religionUNITARIAN-UNIVERSALIST -0.0984      0.0484  -2.03    4.21e-  2
    ## 20 religionUNOBTAINABLE           -0.123       0.0378  -3.27    1.09e-  3

``` r
living_lm6 <- 
  glm(living ~ marital_status, data = admissions_death)
summary(living_lm6)
```

    ## 
    ## Call:
    ## glm(formula = living ~ marital_status, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##      Min        1Q    Median        3Q       Max  
    ## -0.91578   0.08422   0.10774   0.10774   0.18950  
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                      0.900654   0.005443 165.460  < 2e-16 ***
    ## marital_statusLIFE PARTNER       0.099346   0.079827   1.245   0.2133    
    ## marital_statusMARRIED           -0.008398   0.005793  -1.450   0.1471    
    ## marital_statusSEPARATED          0.004775   0.014009   0.341   0.7332    
    ## marital_statusSINGLE             0.015125   0.006081   2.487   0.0129 *  
    ## marital_statusUNKNOWN (DEFAULT) -0.090158   0.017522  -5.146 2.68e-07 ***
    ## marital_statusWIDOWED           -0.046803   0.006545  -7.151 8.73e-13 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.09514164)
    ## 
    ##     Null deviance: 4636.5  on 48522  degrees of freedom
    ## Residual deviance: 4615.9  on 48516  degrees of freedom
    ##   (2605 observations deleted due to missingness)
    ## AIC: 23566
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm6)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         4636.   48522 -11775. 23566. 23637.    4616.       48516

``` r
broom::tidy(living_lm6)
```

    ## # A tibble: 7 x 5
    ##   term                            estimate std.error statistic  p.value
    ##   <chr>                              <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)                      0.901     0.00544   165.    0.      
    ## 2 marital_statusLIFE PARTNER       0.0993    0.0798      1.24  2.13e- 1
    ## 3 marital_statusMARRIED           -0.00840   0.00579    -1.45  1.47e- 1
    ## 4 marital_statusSEPARATED          0.00478   0.0140      0.341 7.33e- 1
    ## 5 marital_statusSINGLE             0.0151    0.00608     2.49  1.29e- 2
    ## 6 marital_statusUNKNOWN (DEFAULT) -0.0902    0.0175     -5.15  2.68e- 7
    ## 7 marital_statusWIDOWED           -0.0468    0.00654    -7.15  8.73e-13

``` r
living_lm7 <- 
  glm(living ~ ethnicity, data = admissions_death)
summary(living_lm7)
```

    ## 
    ## Call:
    ## glm(formula = living ~ ethnicity, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##      Min        1Q    Median        3Q       Max  
    ## -0.98214   0.09111   0.11068   0.11068   0.33333  
    ## 
    ## Coefficients:
    ##                                                                     Estimate
    ## (Intercept)                                                        9.167e-01
    ## ethnicityAMERICAN INDIAN/ALASKA NATIVE FEDERALLY RECOGNIZED TRIBE -2.500e-01
    ## ethnicityASIAN                                                    -4.798e-02
    ## ethnicityASIAN - ASIAN INDIAN                                      4.630e-02
    ## ethnicityASIAN - CAMBODIAN                                        -1.520e-01
    ## ethnicityASIAN - CHINESE                                          -2.964e-02
    ## ethnicityASIAN - FILIPINO                                          3.586e-13
    ## ethnicityASIAN - JAPANESE                                         -2.500e-01
    ## ethnicityASIAN - KOREAN                                            9.215e-13
    ## ethnicityASIAN - OTHER                                            -8.333e-02
    ## ethnicityASIAN - THAI                                              8.333e-02
    ## ethnicityASIAN - VIETNAMESE                                       -4.433e-02
    ## ethnicityBLACK/AFRICAN                                             8.333e-03
    ## ethnicityBLACK/AFRICAN AMERICAN                                   -2.150e-03
    ## ethnicityBLACK/CAPE VERDEAN                                        1.476e-02
    ## ethnicityBLACK/HAITIAN                                             2.211e-02
    ## ethnicityCARIBBEAN ISLAND                                          8.333e-02
    ## ethnicityHISPANIC OR LATINO                                        6.468e-03
    ## ethnicityHISPANIC/LATINO - CENTRAL AMERICAN (OTHER)                8.333e-02
    ## ethnicityHISPANIC/LATINO - COLOMBIAN                               8.333e-02
    ## ethnicityHISPANIC/LATINO - CUBAN                                   8.333e-02
    ## ethnicityHISPANIC/LATINO - DOMINICAN                               4.437e-02
    ## ethnicityHISPANIC/LATINO - GUATEMALAN                              5.769e-02
    ## ethnicityHISPANIC/LATINO - HONDURAN                                8.333e-02
    ## ethnicityHISPANIC/LATINO - MEXICAN                                 8.333e-02
    ## ethnicityHISPANIC/LATINO - PUERTO RICAN                            3.024e-02
    ## ethnicityHISPANIC/LATINO - SALVADORAN                              8.333e-02
    ## ethnicityMIDDLE EASTERN                                            3.333e-02
    ## ethnicityMULTI RACE ETHNICITY                                     -3.205e-03
    ## ethnicityNATIVE HAWAIIAN OR OTHER PACIFIC ISLANDER                -1.833e-01
    ## ethnicityOTHER                                                    -1.885e-02
    ## ethnicityPATIENT DECLINED TO ANSWER                               -7.778e-03
    ## ethnicityPORTUGUESE                                                3.161e-02
    ## ethnicitySOUTH AMERICAN                                            8.333e-02
    ## ethnicityUNABLE TO OBTAIN                                         -1.190e-01
    ## ethnicityUNKNOWN/NOT SPECIFIED                                    -9.610e-02
    ## ethnicityWHITE                                                    -2.735e-02
    ## ethnicityWHITE - BRAZILIAN                                         6.548e-02
    ## ethnicityWHITE - EASTERN EUROPEAN                                  3.727e-13
    ## ethnicityWHITE - OTHER EUROPEAN                                    4.386e-03
    ## ethnicityWHITE - RUSSIAN                                          -5.081e-02
    ##                                                                   Std. Error
    ## (Intercept)                                                        6.449e-02
    ## ethnicityAMERICAN INDIAN/ALASKA NATIVE FEDERALLY RECOGNIZED TRIBE  1.935e-01
    ## ethnicityASIAN                                                     6.546e-02
    ## ethnicityASIAN - ASIAN INDIAN                                      7.342e-02
    ## ethnicityASIAN - CAMBODIAN                                         1.001e-01
    ## ethnicityASIAN - CHINESE                                           6.765e-02
    ## ethnicityASIAN - FILIPINO                                          9.120e-02
    ## ethnicityASIAN - JAPANESE                                          1.442e-01
    ## ethnicityASIAN - KOREAN                                            1.117e-01
    ## ethnicityASIAN - OTHER                                             1.117e-01
    ## ethnicityASIAN - THAI                                              1.706e-01
    ## ethnicityASIAN - VIETNAMESE                                        7.926e-02
    ## ethnicityBLACK/AFRICAN                                             8.157e-02
    ## ethnicityBLACK/AFRICAN AMERICAN                                    6.466e-02
    ## ethnicityBLACK/CAPE VERDEAN                                        6.877e-02
    ## ethnicityBLACK/HAITIAN                                             7.195e-02
    ## ethnicityCARIBBEAN ISLAND                                          1.235e-01
    ## ethnicityHISPANIC OR LATINO                                        6.507e-02
    ## ethnicityHISPANIC/LATINO - CENTRAL AMERICAN (OTHER)                1.150e-01
    ## ethnicityHISPANIC/LATINO - COLOMBIAN                               1.235e-01
    ## ethnicityHISPANIC/LATINO - CUBAN                                   9.120e-02
    ## ethnicityHISPANIC/LATINO - DOMINICAN                               7.386e-02
    ## ethnicityHISPANIC/LATINO - GUATEMALAN                              8.196e-02
    ## ethnicityHISPANIC/LATINO - HONDURAN                                1.706e-01
    ## ethnicityHISPANIC/LATINO - MEXICAN                                 1.150e-01
    ## ethnicityHISPANIC/LATINO - PUERTO RICAN                            6.782e-02
    ## ethnicityHISPANIC/LATINO - SALVADORAN                              9.850e-02
    ## ethnicityMIDDLE EASTERN                                            8.157e-02
    ## ethnicityMULTI RACE ETHNICITY                                      7.154e-02
    ## ethnicityNATIVE HAWAIIAN OR OTHER PACIFIC ISLANDER                 1.040e-01
    ## ethnicityOTHER                                                     6.516e-02
    ## ethnicityPATIENT DECLINED TO ANSWER                                6.618e-02
    ## ethnicityPORTUGUESE                                                7.668e-02
    ## ethnicitySOUTH AMERICAN                                            1.357e-01
    ## ethnicityUNABLE TO OBTAIN                                          6.547e-02
    ## ethnicityUNKNOWN/NOT SPECIFIED                                     6.468e-02
    ## ethnicityWHITE                                                     6.451e-02
    ## ethnicityWHITE - BRAZILIAN                                         7.708e-02
    ## ethnicityWHITE - EASTERN EUROPEAN                                  9.120e-02
    ## ethnicityWHITE - OTHER EUROPEAN                                    7.397e-02
    ## ethnicityWHITE - RUSSIAN                                           6.904e-02
    ##                                                                   t value
    ## (Intercept)                                                        14.215
    ## ethnicityAMERICAN INDIAN/ALASKA NATIVE FEDERALLY RECOGNIZED TRIBE  -1.292
    ## ethnicityASIAN                                                     -0.733
    ## ethnicityASIAN - ASIAN INDIAN                                       0.631
    ## ethnicityASIAN - CAMBODIAN                                         -1.517
    ## ethnicityASIAN - CHINESE                                           -0.438
    ## ethnicityASIAN - FILIPINO                                           0.000
    ## ethnicityASIAN - JAPANESE                                          -1.734
    ## ethnicityASIAN - KOREAN                                             0.000
    ## ethnicityASIAN - OTHER                                             -0.746
    ## ethnicityASIAN - THAI                                               0.488
    ## ethnicityASIAN - VIETNAMESE                                        -0.559
    ## ethnicityBLACK/AFRICAN                                              0.102
    ## ethnicityBLACK/AFRICAN AMERICAN                                    -0.033
    ## ethnicityBLACK/CAPE VERDEAN                                         0.215
    ## ethnicityBLACK/HAITIAN                                              0.307
    ## ethnicityCARIBBEAN ISLAND                                           0.675
    ## ethnicityHISPANIC OR LATINO                                         0.099
    ## ethnicityHISPANIC/LATINO - CENTRAL AMERICAN (OTHER)                 0.724
    ## ethnicityHISPANIC/LATINO - COLOMBIAN                                0.675
    ## ethnicityHISPANIC/LATINO - CUBAN                                    0.914
    ## ethnicityHISPANIC/LATINO - DOMINICAN                                0.601
    ## ethnicityHISPANIC/LATINO - GUATEMALAN                               0.704
    ## ethnicityHISPANIC/LATINO - HONDURAN                                 0.488
    ## ethnicityHISPANIC/LATINO - MEXICAN                                  0.724
    ## ethnicityHISPANIC/LATINO - PUERTO RICAN                             0.446
    ## ethnicityHISPANIC/LATINO - SALVADORAN                               0.846
    ## ethnicityMIDDLE EASTERN                                             0.409
    ## ethnicityMULTI RACE ETHNICITY                                      -0.045
    ## ethnicityNATIVE HAWAIIAN OR OTHER PACIFIC ISLANDER                 -1.763
    ## ethnicityOTHER                                                     -0.289
    ## ethnicityPATIENT DECLINED TO ANSWER                                -0.118
    ## ethnicityPORTUGUESE                                                 0.412
    ## ethnicitySOUTH AMERICAN                                             0.614
    ## ethnicityUNABLE TO OBTAIN                                          -1.817
    ## ethnicityUNKNOWN/NOT SPECIFIED                                     -1.486
    ## ethnicityWHITE                                                     -0.424
    ## ethnicityWHITE - BRAZILIAN                                          0.850
    ## ethnicityWHITE - EASTERN EUROPEAN                                   0.000
    ## ethnicityWHITE - OTHER EUROPEAN                                     0.059
    ## ethnicityWHITE - RUSSIAN                                           -0.736
    ##                                                                   Pr(>|t|)
    ## (Intercept)                                                         <2e-16
    ## ethnicityAMERICAN INDIAN/ALASKA NATIVE FEDERALLY RECOGNIZED TRIBE   0.1963
    ## ethnicityASIAN                                                      0.4636
    ## ethnicityASIAN - ASIAN INDIAN                                       0.5283
    ## ethnicityASIAN - CAMBODIAN                                          0.1292
    ## ethnicityASIAN - CHINESE                                            0.6613
    ## ethnicityASIAN - FILIPINO                                           1.0000
    ## ethnicityASIAN - JAPANESE                                           0.0830
    ## ethnicityASIAN - KOREAN                                             1.0000
    ## ethnicityASIAN - OTHER                                              0.4556
    ## ethnicityASIAN - THAI                                               0.6252
    ## ethnicityASIAN - VIETNAMESE                                         0.5760
    ## ethnicityBLACK/AFRICAN                                              0.9186
    ## ethnicityBLACK/AFRICAN AMERICAN                                     0.9735
    ## ethnicityBLACK/CAPE VERDEAN                                         0.8300
    ## ethnicityBLACK/HAITIAN                                              0.7586
    ## ethnicityCARIBBEAN ISLAND                                           0.4998
    ## ethnicityHISPANIC OR LATINO                                         0.9208
    ## ethnicityHISPANIC/LATINO - CENTRAL AMERICAN (OTHER)                 0.4688
    ## ethnicityHISPANIC/LATINO - COLOMBIAN                                0.4998
    ## ethnicityHISPANIC/LATINO - CUBAN                                    0.3608
    ## ethnicityHISPANIC/LATINO - DOMINICAN                                0.5480
    ## ethnicityHISPANIC/LATINO - GUATEMALAN                               0.4815
    ## ethnicityHISPANIC/LATINO - HONDURAN                                 0.6252
    ## ethnicityHISPANIC/LATINO - MEXICAN                                  0.4688
    ## ethnicityHISPANIC/LATINO - PUERTO RICAN                             0.6557
    ## ethnicityHISPANIC/LATINO - SALVADORAN                               0.3976
    ## ethnicityMIDDLE EASTERN                                             0.6828
    ## ethnicityMULTI RACE ETHNICITY                                       0.9643
    ## ethnicityNATIVE HAWAIIAN OR OTHER PACIFIC ISLANDER                  0.0779
    ## ethnicityOTHER                                                      0.7724
    ## ethnicityPATIENT DECLINED TO ANSWER                                 0.9064
    ## ethnicityPORTUGUESE                                                 0.6802
    ## ethnicitySOUTH AMERICAN                                             0.5392
    ## ethnicityUNABLE TO OBTAIN                                           0.0692
    ## ethnicityUNKNOWN/NOT SPECIFIED                                      0.1373
    ## ethnicityWHITE                                                      0.6716
    ## ethnicityWHITE - BRAZILIAN                                          0.3956
    ## ethnicityWHITE - EASTERN EUROPEAN                                   1.0000
    ## ethnicityWHITE - OTHER EUROPEAN                                     0.9527
    ## ethnicityWHITE - RUSSIAN                                            0.4618
    ##                                                                      
    ## (Intercept)                                                       ***
    ## ethnicityAMERICAN INDIAN/ALASKA NATIVE FEDERALLY RECOGNIZED TRIBE    
    ## ethnicityASIAN                                                       
    ## ethnicityASIAN - ASIAN INDIAN                                        
    ## ethnicityASIAN - CAMBODIAN                                           
    ## ethnicityASIAN - CHINESE                                             
    ## ethnicityASIAN - FILIPINO                                            
    ## ethnicityASIAN - JAPANESE                                         .  
    ## ethnicityASIAN - KOREAN                                              
    ## ethnicityASIAN - OTHER                                               
    ## ethnicityASIAN - THAI                                                
    ## ethnicityASIAN - VIETNAMESE                                          
    ## ethnicityBLACK/AFRICAN                                               
    ## ethnicityBLACK/AFRICAN AMERICAN                                      
    ## ethnicityBLACK/CAPE VERDEAN                                          
    ## ethnicityBLACK/HAITIAN                                               
    ## ethnicityCARIBBEAN ISLAND                                            
    ## ethnicityHISPANIC OR LATINO                                          
    ## ethnicityHISPANIC/LATINO - CENTRAL AMERICAN (OTHER)                  
    ## ethnicityHISPANIC/LATINO - COLOMBIAN                                 
    ## ethnicityHISPANIC/LATINO - CUBAN                                     
    ## ethnicityHISPANIC/LATINO - DOMINICAN                                 
    ## ethnicityHISPANIC/LATINO - GUATEMALAN                                
    ## ethnicityHISPANIC/LATINO - HONDURAN                                  
    ## ethnicityHISPANIC/LATINO - MEXICAN                                   
    ## ethnicityHISPANIC/LATINO - PUERTO RICAN                              
    ## ethnicityHISPANIC/LATINO - SALVADORAN                                
    ## ethnicityMIDDLE EASTERN                                              
    ## ethnicityMULTI RACE ETHNICITY                                        
    ## ethnicityNATIVE HAWAIIAN OR OTHER PACIFIC ISLANDER                .  
    ## ethnicityOTHER                                                       
    ## ethnicityPATIENT DECLINED TO ANSWER                                  
    ## ethnicityPORTUGUESE                                                  
    ## ethnicitySOUTH AMERICAN                                              
    ## ethnicityUNABLE TO OBTAIN                                         .  
    ## ethnicityUNKNOWN/NOT SPECIFIED                                       
    ## ethnicityWHITE                                                       
    ## ethnicityWHITE - BRAZILIAN                                           
    ## ethnicityWHITE - EASTERN EUROPEAN                                    
    ## ethnicityWHITE - OTHER EUROPEAN                                      
    ## ethnicityWHITE - RUSSIAN                                             
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.09980305)
    ## 
    ##     Null deviance: 5135.1  on 51127  degrees of freedom
    ## Residual deviance: 5098.6  on 51087  degrees of freedom
    ## AIC: 27311
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm7)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         5135.   51127 -13613. 27311. 27682.    5099.       51087

``` r
broom::tidy(living_lm7)
```

    ## # A tibble: 41 x 5
    ##    term                             estimate std.error statistic   p.value
    ##    <chr>                               <dbl>     <dbl>     <dbl>     <dbl>
    ##  1 (Intercept)                      9.17e- 1    0.0645  1.42e+ 1  9.05e-46
    ##  2 ethnicityAMERICAN INDIAN/ALASK… -2.50e- 1    0.193  -1.29e+ 0  1.96e- 1
    ##  3 ethnicityASIAN                  -4.80e- 2    0.0655 -7.33e- 1  4.64e- 1
    ##  4 ethnicityASIAN - ASIAN INDIAN    4.63e- 2    0.0734  6.31e- 1  5.28e- 1
    ##  5 ethnicityASIAN - CAMBODIAN      -1.52e- 1    0.100  -1.52e+ 0  1.29e- 1
    ##  6 ethnicityASIAN - CHINESE        -2.96e- 2    0.0676 -4.38e- 1  6.61e- 1
    ##  7 ethnicityASIAN - FILIPINO        3.59e-13    0.0912  3.93e-12 10.00e- 1
    ##  8 ethnicityASIAN - JAPANESE       -2.50e- 1    0.144  -1.73e+ 0  8.30e- 2
    ##  9 ethnicityASIAN - KOREAN          9.22e-13    0.112   8.25e-12 10.00e- 1
    ## 10 ethnicityASIAN - OTHER          -8.33e- 2    0.112  -7.46e- 1  4.56e- 1
    ## # ... with 31 more rows

``` r
#living_lm8 <-   Too many diagnosis 
#  glm(living ~ diagnosis, data = admissions_death)
#summary(living_lm8)

living_lm9 <- 
  glm(living ~ hospital_expire_flag, data = admissions_death)
summary(living_lm9)
```

    ## 
    ## Call:
    ## glm(formula = living ~ hospital_expire_flag, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##       Min         1Q     Median         3Q        Max  
    ## 3.994e-13  7.255e-13  7.255e-13  7.255e-13  7.255e-13  
    ## 
    ## Coefficients:
    ##                        Estimate Std. Error    t value Pr(>|t|)    
    ## (Intercept)           1.000e+00  3.270e-15  3.058e+14   <2e-16 ***
    ## hospital_expire_flag -1.000e+00  9.717e-15 -1.029e+14   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 4.848551e-25)
    ## 
    ##     Null deviance: 5.1351e+03  on 51127  degrees of freedom
    ## Residual deviance: 2.4789e-20  on 51126  degrees of freedom
    ## AIC: -2717351
    ## 
    ## Number of Fisher Scoring iterations: 1

``` r
broom::glance(living_lm9)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null   logLik       AIC       BIC deviance df.residual
    ##           <dbl>   <int>    <dbl>     <dbl>     <dbl>    <dbl>       <int>
    ## 1         5135.   51127 1358678. -2717351. -2717324. 2.48e-20       51126

``` r
broom::tidy(living_lm9)
```

    ## # A tibble: 2 x 5
    ##   term                 estimate std.error statistic p.value
    ##   <chr>                   <dbl>     <dbl>     <dbl>   <dbl>
    ## 1 (Intercept)             1.000  3.27e-15   3.06e14       0
    ## 2 hospital_expire_flag   -1.000  9.72e-15  -1.03e14       0

``` r
living_lm10 <- 
  glm(living ~ has_chartevents_data, data = admissions_death)
summary(living_lm10)
```

    ## 
    ## Call:
    ## glm(formula = living ~ has_chartevents_data, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -0.9830   0.1161   0.1161   0.1161   0.1161  
    ## 
    ## Coefficients:
    ##                       Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)           0.983028   0.008246  119.21   <2e-16 ***
    ## has_chartevents_data -0.099149   0.008368  -11.85   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.1001647)
    ## 
    ##     Null deviance: 5135.1  on 51127  degrees of freedom
    ## Residual deviance: 5121.0  on 51126  degrees of freedom
    ## AIC: 27457
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm10)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         5135.   51127 -13725. 27457. 27483.    5121.       51126

``` r
broom::tidy(living_lm10)
```

    ## # A tibble: 2 x 5
    ##   term                 estimate std.error statistic  p.value
    ##   <chr>                   <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)            0.983    0.00825     119.  0.      
    ## 2 has_chartevents_data  -0.0991   0.00837     -11.8 2.40e-32

``` r
living_lm11 <- 
  glm(living ~ hospitaltime, data = admissions_death)
summary(living_lm11)
```

    ## 
    ## Call:
    ## glm(formula = living ~ hospitaltime, data = admissions_death)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -0.8890   0.1118   0.1124   0.1135   0.1734  
    ## 
    ## Coefficients:
    ##                Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   8.888e-01  1.896e-03 468.684   <2e-16 ***
    ## hospitaltime -1.468e-07  8.930e-08  -1.644      0.1    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.1004345)
    ## 
    ##     Null deviance: 5135.1  on 51127  degrees of freedom
    ## Residual deviance: 5134.8  on 51126  degrees of freedom
    ## AIC: 27594
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm11)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         5135.   51127 -13794. 27594. 27621.    5135.       51126

``` r
broom::tidy(living_lm11)
```

    ## # A tibble: 2 x 5
    ##   term             estimate    std.error statistic p.value
    ##   <chr>               <dbl>        <dbl>     <dbl>   <dbl>
    ## 1 (Intercept)   0.889       0.00190         469.     0    
    ## 2 hospitaltime -0.000000147 0.0000000893     -1.64   0.100

``` r
living_lm12 <- 
  glm(living ~ edtime, data = na.omit(select(admissions_death, living, edtime)))
summary(living_lm12)
```

    ## 
    ## Call:
    ## glm(formula = living ~ edtime, data = na.omit(select(admissions_death, 
    ##     living, edtime)))
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -0.9222   0.1275   0.1327   0.1355   0.2149  
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 8.588e-01  2.751e-03 312.167  < 2e-16 ***
    ## edtime      4.286e-07  9.493e-08   4.515 6.34e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for gaussian family taken to be 0.1147309)
    ## 
    ##     Null deviance: 3544.5  on 30875  degrees of freedom
    ## Residual deviance: 3542.2  on 30874  degrees of freedom
    ## AIC: 20775
    ## 
    ## Number of Fisher Scoring iterations: 2

``` r
broom::glance(living_lm12)
```

    ## # A tibble: 1 x 7
    ##   null.deviance df.null  logLik    AIC    BIC deviance df.residual
    ##           <dbl>   <int>   <dbl>  <dbl>  <dbl>    <dbl>       <int>
    ## 1         3545.   30875 -10384. 20775. 20800.    3542.       30874

``` r
broom::tidy(living_lm12)
```

    ## # A tibble: 2 x 5
    ##   term           estimate    std.error statistic    p.value
    ##   <chr>             <dbl>        <dbl>     <dbl>      <dbl>
    ## 1 (Intercept) 0.859       0.00275         312.   0         
    ## 2 edtime      0.000000429 0.0000000949      4.52 0.00000634

``` r
#MLR
living_mlr <- 
  glm(living ~ admission_type + admission_location + insurance + language + religion + marital_status + ethnicity + hospital_expire_flag + has_chartevents_data + hospitaltime + edtime, data = admissions_death)
broom::glance(living_mlr)







# Do more regression analysis

coefficients(living_mlr)
confint(living_mlr)
fitted(living_mlr)
residuals(living_mlr)
anova(living_mlr)
vcov(living_mlr)


###backward elimination
#Backward elimination: take out non-significant variables ’one at a time’ starting with the highest p-value
step1 <- update(living_mlr, . ~. -admission_type)
broom::glance(step1)
# remove

step2 <- update(living_mlr, . ~. -admission_type-admission_location)
broom::glance(step2)
#keep

step3 <- update(living_mlr, . ~. -admission_type-insurance)
broom::glance(step3)
#keep

step4 <- update(living_mlr, . ~. -admission_type-language)
broom::glance(step4)
#remove

step5 <- update(living_mlr, . ~. -admission_type-language-religion)
broom::glance(step5)
#keep

step6 <- update(living_mlr, . ~. -admission_type-language-marital_status)
broom::glance(step6)
#keep

step7 <- update(living_mlr, . ~. -admission_type-language-ethnicity)
broom::glance(step7)
#keep

step8 <- update(living_mlr, . ~. -admission_type-language-hospital_expire_flag)
broom::glance(step8)
#keep

step9 <- update(living_mlr, . ~. -admission_type-language-has_chartevents_data)
broom::glance(step9)
#keep

step10 <- update(living_mlr, . ~.  -admission_type-language-hospitaltime)
broom::glance(step10)
#keep

step11 <- update(living_mlr, . ~. -admission_type-language-edtime)
broom::glance(step11)
#keep
```

So we got this regression model:

$\\hat{Living}$ = $\\hat{\\beta\_0}$ + $\\hat{\\beta\_1}$ Admission.Location + $\\hat{\\beta\_2}$ Insurance + $\\hat{\\beta\_3}$ Insurance + $\\hat{\\beta\_4}$ Religion + $\\hat{\\beta\_5}$ Marital.Status + $\\hat{\\beta\_6}$ Ethnicity + $\\hat{\\beta\_7}$ Hospital.Expire.Flag + $\\hat{\\beta\_8}$ Has.ChartEvents.Data + $\\hat{\\beta\_9}$ Hospital.Time + $\\hat{\\beta\_10}$ Edtime

Maybe we need this step:

``` r
# Remove NA lines
#admissions_death_nona <- 
#  admissions_death %>% 
#  select(admission_type, admission_location, insurance, religion, marital_status, ethnicity, hospitaltime, edtime) %>% 
#  na.omit() 

#MLR with no NAs
#living_mlr_nona <- 
#  glm(living ~ admission_type + admission_location + insurance + language + religion + marital_status + ethnicity + hospital_expire_flag + has_chartevents_data + hospitaltime + edtime, data = admissions_death_nona)
#summary(living_mlr_nona)
#broom::glance(living_mlr_nona)
#broom::tidy(living_mlr_nona)
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

### Visulization

Let's take a look at the combination of MIMIC data and ICU together(for we want to know the ICU time since it is more critical for mortality)

``` r
icu <- 
  read_csv("./database/data/icu_detail.csv") %>% 
  janitor::clean_names()
```

    ## Parsed with column specification:
    ## cols(
    ##   subject_id = col_double(),
    ##   hadm_id = col_double(),
    ##   icustay_id = col_double(),
    ##   gender = col_character(),
    ##   dod = col_datetime(format = ""),
    ##   admittime = col_datetime(format = ""),
    ##   dischtime = col_datetime(format = ""),
    ##   los_hospital = col_double(),
    ##   admission_age = col_double(),
    ##   ethnicity = col_character(),
    ##   admission_type = col_character(),
    ##   hospital_expire_flag = col_double(),
    ##   hospstay_seq = col_double(),
    ##   first_hosp_stay = col_logical(),
    ##   intime = col_datetime(format = ""),
    ##   outtime = col_datetime(format = ""),
    ##   los_icu = col_double(),
    ##   icustay_seq = col_double(),
    ##   first_icu_stay = col_logical()
    ## )

``` r
# See what is the types of columns in icu dataset
output = vector("list", length = 19)

for (i in 1:19) {
  output[[i]] = class(icu[[i]])
}
```

Let's combine these two dataset so we can visualize it correctly

``` r
total_patient <-
  merge(admissions_death, icu, by="hadm_id") %>% 
  as.tibble()

#mutate icu time
total_patient <- 
  mutate(total_patient, icutime = outtime - intime)

skimr::skim(total_patient) %>% 
  skimr::skim()
```

    ## Skim summary statistics
    ##  n obs: 339 
    ##  n variables: 6 
    ## 
    ## ── Variable type:character ─────────────────────────────────────────────────────────────────────────────────────
    ##   variable missing complete   n min max empty n_unique
    ##  formatted       0      339 339   1  13     0      138
    ##      level       3      336 339   4   5     0        3
    ##       stat       0      339 339   1   8     0       17
    ##       type       0      339 339   7   9     0        5
    ##   variable       0      339 339   3  22     0       41
    ## 
    ## ── Variable type:numeric ───────────────────────────────────────────────────────────────────────────────────────
    ##  variable missing complete   n    mean      sd       p0 p25      p50   p75
    ##     value      13      326 339 5.3e+08 1.7e+09 -2504940   1 16716.54 53219
    ##     p100     hist
    ##  7.6e+09 ▇▁▁▁▁▁▁▁

``` r
head(total_patient) %>% 
  knitr::kable()
```

|  hadm\_id|  row\_id|  subject\_id.x| admittime.x         | dischtime.x         | deathtime | admission\_type.x | admission\_location       | discharge\_location | insurance | language | religion          | marital\_status | ethnicity.x            | edregtime           | edouttime           | diagnosis               |  hospital\_expire\_flag.x|  has\_chartevents\_data| living | hospitaltime | edtime     |  subject\_id.y|  icustay\_id| gender | dod                 | admittime.y         | dischtime.y         |  los\_hospital|  admission\_age| ethnicity.y            | admission\_type.y |  hospital\_expire\_flag.y|  hospstay\_seq| first\_hosp\_stay | intime              | outtime             |  los\_icu|  icustay\_seq| first\_icu\_stay | icutime     |
|---------:|--------:|--------------:|:--------------------|:--------------------|:----------|:------------------|:--------------------------|:--------------------|:----------|:---------|:------------------|:----------------|:-----------------------|:--------------------|:--------------------|:------------------------|-------------------------:|-----------------------:|:-------|:-------------|:-----------|--------------:|------------:|:-------|:--------------------|:--------------------|:--------------------|--------------:|---------------:|:-----------------------|:------------------|-------------------------:|--------------:|:------------------|:--------------------|:--------------------|---------:|-------------:|:-----------------|:------------|
|    100001|    45749|          58526| 2117-09-11 11:46:00 | 2117-09-17 16:45:00 | NA        | EMERGENCY         | CLINIC REFERRAL/PREMATURE | HOME                | Private   | ENGL     | PROTESTANT QUAKER | DIVORCED        | WHITE                  | 2117-09-11 08:59:00 | 2117-09-11 12:35:00 | DIABETIC KETOACIDOSIS   |                         0|                       1| TRUE   | 8939 mins    | 12960 secs |          58526|       275225| F      | NA                  | 2117-09-11 04:00:00 | 2117-09-17 04:00:00 |         6.2076|         35.4765| WHITE                  | EMERGENCY         |                         0|              1| TRUE              | 2117-09-11 04:00:00 | 2117-09-15 04:00:00 |    4.2567|             1| TRUE             | 345600 secs |
|    100003|    44463|          54610| 2150-04-17 15:34:00 | 2150-04-21 17:30:00 | NA        | EMERGENCY         | EMERGENCY ROOM ADMIT      | HOME                | Private   | ENGL     | NOT SPECIFIED     | SINGLE          | WHITE                  | 2150-04-17 13:10:00 | 2150-04-17 17:47:00 | UPPER GI BLEED          |                         0|                       1| TRUE   | 5876 mins    | 16620 secs |          54610|       209281| M      | 2150-12-28 05:00:00 | 2150-04-17 04:00:00 | 2150-04-21 04:00:00 |         4.0806|         59.9127| WHITE                  | EMERGENCY         |                         0|              1| TRUE              | 2150-04-17 04:00:00 | 2150-04-19 04:00:00 |    1.9425|             1| TRUE             | 172800 secs |
|    100006|    12108|           9895| 2108-04-06 15:49:00 | 2108-04-18 17:18:00 | NA        | EMERGENCY         | EMERGENCY ROOM ADMIT      | HOME                | Private   | NA       | NOT SPECIFIED     | SINGLE          | BLACK/AFRICAN AMERICAN | 2108-04-06 11:39:00 | 2108-04-06 17:56:00 | COPD FLARE              |                         0|                       1| TRUE   | 17369 mins   | 22620 secs |           9895|       291788| F      | 2109-10-24 04:00:00 | 2108-04-06 04:00:00 | 2108-04-18 04:00:00 |        12.0618|         48.9173| BLACK/AFRICAN AMERICAN | EMERGENCY         |                         0|              1| TRUE              | 2108-04-06 04:00:00 | 2108-04-11 04:00:00 |    4.9776|             1| TRUE             | 432000 secs |
|    100007|    28086|          23018| 2145-03-31 05:33:00 | 2145-04-07 12:40:00 | NA        | EMERGENCY         | EMERGENCY ROOM ADMIT      | HOME                | Private   | NA       | JEWISH            | MARRIED         | WHITE                  | 2145-03-30 20:43:00 | 2145-03-31 06:08:00 | BOWEL OBSTRUCTION       |                         0|                       1| TRUE   | 10507 mins   | 33900 secs |          23018|       217937| F      | NA                  | 2145-03-31 04:00:00 | 2145-04-07 04:00:00 |         7.2965|         73.8229| WHITE                  | EMERGENCY         |                         0|              1| TRUE              | 2145-03-31 04:00:00 | 2145-04-04 04:00:00 |    4.0998|             1| TRUE             | 345600 secs |
|    100009|      671|            533| 2162-05-16 15:56:00 | 2162-05-21 13:37:00 | NA        | EMERGENCY         | TRANSFER FROM HOSP/EXTRAM | HOME HEALTH CARE    | Private   | NA       | CATHOLIC          | MARRIED         | WHITE                  | NA                  | NA                  | CORONARY ARTERY DISEASE |                         0|                       1| TRUE   | 7061 mins    | NA         |            533|       253656| M      | NA                  | 2162-05-16 04:00:00 | 2162-05-21 04:00:00 |         4.9035|         60.7971| WHITE                  | EMERGENCY         |                         0|              1| TRUE              | 2162-05-17 04:00:00 | 2162-05-19 04:00:00 |    2.4908|             1| TRUE             | 172800 secs |
|    100010|    44865|          55853| 2109-12-10 07:15:00 | 2109-12-14 16:45:00 | NA        | ELECTIVE          | PHYS REFERRAL/NORMAL DELI | HOME                | Private   | ENGL     | EPISCOPALIAN      | MARRIED         | WHITE                  | NA                  | NA                  | RENAL MASS LEFT/SDA     |                         0|                       1| TRUE   | 6330 mins    | NA         |          55853|       271147| F      | NA                  | 2109-12-10 05:00:00 | 2109-12-14 05:00:00 |         4.3958|         54.5208| WHITE                  | ELECTIVE          |                         0|              1| TRUE              | 2109-12-10 05:00:00 | 2109-12-12 05:00:00 |    1.5940|             1| TRUE             | 172800 secs |

Let's see some plots from these two datasets Let's focus on the `hospitaltime` and `icutime`

``` r
mlr_time <- 
  glm(living ~ hospitaltime + icutime, data = total_patient)
plot(mlr_time)

plot(total_patient)
```
