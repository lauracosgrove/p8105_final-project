length\_of\_care
================
Samantha Brown
11/17/2018

``` r
admissions <- 
  read_csv("./database/data/admissions.csv") %>% 
  janitor::clean_names() %>% 
  mutate(diagnosis = factor(diagnosis))
```

``` r
n_admits = admissions %>% 
  nrow()

n_discharge = admissions %>% 
  filter(is.na(deathtime)) %>% 
  nrow()

round((1 - (n_discharge/n_admits)) * 100, digits = 2)
```

    ## [1] 9.93

**Out of 58976 patient admissions, 53122 patients were ultimately discharged. The remaining 9.93% were recorded deaths.**

``` r
## Top 10 causes of recorded mortalities
admissions %>% 
  filter(deathtime > 0) %>%
  count(diagnosis) %>% 
  top_n(10) %>% 
  arrange(desc(n))
```

    ## Warning: Factor `diagnosis` contains implicit NA, consider using
    ## `forcats::fct_explicit_na`

    ## Selecting by n

    ## # A tibble: 10 x 2
    ##    diagnosis                    n
    ##    <fct>                    <int>
    ##  1 SEPSIS                     267
    ##  2 PNEUMONIA                  264
    ##  3 INTRACRANIAL HEMORRHAGE    231
    ##  4 CONGESTIVE HEART FAILURE   126
    ##  5 ALTERED MENTAL STATUS       88
    ##  6 CARDIAC ARREST              81
    ##  7 ABDOMINAL PAIN              80
    ##  8 S/P FALL                    78
    ##  9 HYPOTENSION                 74
    ## 10 SUBARACHNOID HEMORRHAGE     71

``` r
## Look at sepsis diagnoses
sepsis = admissions %>% 
  filter(diagnosis == "SEPSIS") %>% 
  ## discharge = 1, mortality = 0
  mutate(mortality = ifelse(is.na(deathtime), 1, 0)) %>% 
  ## create indicator variable: married = 1, not married = 0
  mutate(marital_status = ifelse(marital_status == "MARRIED", 1, 0)) %>% 
  select(subject_id, insurance, marital_status, ethnicity, mortality)


## Look at top 10 causes of mortalities
top_10_causes = admissions %>% 
  filter(diagnosis == c("ABDOMINAL PAIN", "ALTERED MENTAL STATUS", "CARDIAC ARREST", "CONGESTIVE HEART FAILURE", "HEAD BLEED", "HYPOTENSION", "INTACRANIAL HEMORRHAGE", "PNEUMONIA", "SEPSIS", "STROKE;TELEMETRY;TRANSIENT ISCHEMIC ATTACK"))
```

    ## Warning in `==.default`(diagnosis, c("ABDOMINAL PAIN", "ALTERED MENTAL
    ## STATUS", : longer object length is not a multiple of shorter object length

    ## Warning in is.na(e1) | is.na(e2): longer object length is not a multiple of
    ## shorter object length

``` r
## Insurance -- mortality vs. no mortality 

## mortality
mortalities = admissions %>% 
  filter(deathtime > 0) %>% 
  group_by(insurance) %>% 
  count() %>% 
  mutate(outcome = "mortality")

total_mortalities = sum(mortalities$n)

mortalities = mortalities %>% 
  mutate(patient_proportion = round(n/total_mortalities, digits = 4))

discharge = admissions %>% 
  filter(deathtime > 0) %>% 
  group_by(insurance) %>% 
  count() %>% 
  mutate(outcome = "no_mortality")

total_discharge = sum(discharge$n)

dishcharge = discharge %>% 
  mutate(patient_proportion = round(n/total_discharge, digits = 4)) %>% 
  rbind(mortalities) %>% 
  mutate(log_count = log(n)) %>% 
  ggplot(aes(x = reorder(insurance, log_count), y = log_count)) +
  geom_point(color = "blue") 
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

    ## [1] "POSIXct" "POSIXt"

``` r
class(admissions$dischtime)
```

    ## [1] "POSIXct" "POSIXt"

``` r
time_in = as.POSIXct.Date(admissions$admittime)
time_out = as.POSIXct.Date(admissions$dischtime)

length_of_care = cbind(time_in, time_out) %>% 
  as.tibble() %>% 
  mutate(difference = as.POSIXct.Date(time_out - time_in))
```

``` r
admissions_dates = admissions %>% 
  separate(dischtime, into = c("dischtime_year", "dischtime_month", "dischtime_day"), sep = "-") %>% 
  separate(dischtime_day, into = c("dischtime_day", "dischtime_time"), sep = " ") %>% 
  separate(admittime, into = c("admittime_year", "admittime_month", "admittime_day"), sep = "-") %>% 
  mutate(dischtime_year = as.numeric(dischtime_year), 
         dischtime_month = as.numeric(dischtime_month),
         dischtime_day = as.numeric(dischtime_day),
         admittime_year = as.numeric(admittime_year), 
         admittime_month = as.numeric(admittime_month), 
         admittime_day = as.numeric(admittime_day))
```

    ## Warning: NAs introduced by coercion

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

library(lubridate)
admissions_data <- read_csv("./database/data/admissions.csv") %>% 
  janitor::clean_names()
```

    ## Parsed with column specification:
    ## cols(
    ##   ROW_ID = col_integer(),
    ##   SUBJECT_ID = col_integer(),
    ##   HADM_ID = col_integer(),
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
    ##   HOSPITAL_EXPIRE_FLAG = col_integer(),
    ##   HAS_CHARTEVENTS_DATA = col_integer()
    ## )

``` r
difference = admissions_data %>% 
  mutate(length_of_stay =  lubridate::as.duration(admittime %--% dischtime),
        marital_status = ifelse(marital_status == "MARRIED", 1, 0),
        mortality = ifelse(is.na(deathtime), 1, 0),
        insurance = recode_factor(insurance, "Private" = 1, "Medicare" = 2, "Medicaid" = 3, "Government" = 4, "Self Pay" = 5))

admissions %>% distinct(admission_type)
```

    ## # A tibble: 4 x 1
    ##   admission_type
    ##   <chr>         
    ## 1 EMERGENCY     
    ## 2 ELECTIVE      
    ## 3 NEWBORN       
    ## 4 URGENT

``` r
icu_data = read_csv("./database/data/icu_detail.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   subject_id = col_integer(),
    ##   hadm_id = col_integer(),
    ##   icustay_id = col_integer(),
    ##   gender = col_character(),
    ##   dod = col_datetime(format = ""),
    ##   admittime = col_datetime(format = ""),
    ##   dischtime = col_datetime(format = ""),
    ##   los_hospital = col_double(),
    ##   admission_age = col_double(),
    ##   ethnicity = col_character(),
    ##   admission_type = col_character(),
    ##   hospital_expire_flag = col_integer(),
    ##   hospstay_seq = col_integer(),
    ##   first_hosp_stay = col_logical(),
    ##   intime = col_datetime(format = ""),
    ##   outtime = col_datetime(format = ""),
    ##   los_icu = col_double(),
    ##   icustay_seq = col_integer(),
    ##   first_icu_stay = col_logical()
    ## )

``` r
length_of_care = lm(length_of_stay ~ insurance + marital_status + mortality, data = difference)
```

    ## Note: method with signature 'Duration#ANY' chosen for function '-',
    ##  target signature 'Duration#Duration'.
    ##  "ANY#Duration" would also be valid

``` r
summary(length_of_care)
```

    ## 
    ## Call:
    ## lm(formula = length_of_stay ~ insurance + marital_status + mortality, 
    ##     data = difference)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1016219  -507517  -258543   174203 24614124 
    ## attr(,"class")
    ## [1] "Duration"
    ## attr(,"class")attr(,"package")
    ## [1] "lubridate"
    ## 
    ## Coefficients:
    ##                Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)      902415      15888  56.799  < 2e-16 ***
    ## insurance2         3549       9565   0.371    0.711    
    ## insurance3        82724      16322   5.068 4.03e-07 ***
    ## insurance4       -16526      26536  -0.623    0.533    
    ## insurance5      -233825      45780  -5.108 3.28e-07 ***
    ## marital_status     9061       8744   1.036    0.300    
    ## mortality        -61427      13802  -4.451 8.58e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 936000 on 48841 degrees of freedom
    ##   (10128 observations deleted due to missingness)
    ## Multiple R-squared:  0.001597,   Adjusted R-squared:  0.001474 
    ## F-statistic: 13.02 on 6 and 48841 DF,  p-value: 9.059e-15
