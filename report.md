Influential Factors in Critical Care Patients
================

Group members: Samantha Brown (UNI: slb2240), Laura Cosgrove (UNI: lec2197), and Francis Z. Fang (UNI: zf2211).

### Introduction

Critical care involves the specialized treatment of patients whose conditions pose life-threatening risks and require around-the-clock care. Critical care treatment typically takes place in an intensive care unit (ICU) of a hospital. Due to the nature of critical care, many patients eventually recover, but some die. For the purpose of this project, we seek to explore factors that influence critical care patients.

### Motivation and Related Work

Previous research has focused on the physiological- and disease-driven factors that influence critical care. In this report, we consider whether demographic characteristics of patients in critical care influence outcomes such as mortality and length of hospital stay.

Related work includes MIT Computational Lab's 2015 research publication titled *"Mortality prediction in the ICU: can we do better? Results from the Super ICU Learner Algorithm (SICULA) project, a population-based study"*, which considers whether a machine learning technique can help predict mortality for patients in critical care. This publication considers MIMIC, an openly accessible critical care database created by the MIT Lab for Computational Physiology. With the goal of engaging in open research by reproducing an analysis, this project works to understand and analyze the MIMIC database. MIMIC will be described further in the Data Collection section of the report.

Data Collection
---------------

The MIMIC database comprises deidentified health-related data associated with over forty thousand patients who stayed in critical care units of the Beth Israel Deaconess Medical Center between 2001 and 2012. It includes the following information: demographics, vital sign measurements made at the bedside (~1 data point per hour), laboratory test results, procedures, medications, caregiver notes, imaging reports, and mortality (both in and out of hospital). After completing the CITI “Data or Specimens Only Research” training course, PhysioNet granted us access to the MIMIC database.

The next step was to follow the MIMIC website's open tutorial to install MIMIC in a local Postgres database. We referenced the public Github MIMIC-code repository for MIT Lab for Computational Physiology (<https://github.com/MIT-LCP/mimic-code/tree/master/buildmimic>) as a guide. The MIT researchers who built the MIMIC database also built this repository with the goal of sharing how they performed the technical analysis described in their published literature. Given the time constraints of this project, we were limited to how much we could understand how to make use of the MIMIC data on our own. Therefore, MIT's Lab for Computational Physiology Github repository served as the most productive and efficient way for us to understand and analyze the MIMIC database.

From this database query, we were able to gain a clean table of demographic data for all patients who were in the ICU. Using the query, we performed exploratory analysis on the Admissions data from MIMIC. This gave us patient admit time and discharge time, along with several other demographic variables. However we wanted to dig deeper into the relationship between length of total stay in the hospital, length of stay in the ICU, and proportion of mortalities to try to predict the probability of death. This led us to focus a portion of our analysis on patient severity scores. These scores represent aggregate indices of a patient's condition when they arrive at the ICU. Ultimately we found that the SAPS-II severity score performs best. Using the SAPS-II scores, we ran a regression to obtain the probability of patient mortality. This data collection process allowed us to perform a comprehensive analysis on the MIMIC data.

Initial Questions
-----------------

Our analysis is centered around the question of which factors affect patient mortality. From previous research, prior to the project we understood and recognized the signficant role that physiological factors play in determining the probability of death in a patient. However, we wanted to explore the question: do demographic characteristics help to predict patient mortality? And how strong is their influence of this prediction? For example, does the health insurance coverage of a patient play a part in mortality, beyond basic diagnostic factors? Additionally, we wished to consider whether length of patient hospital stay is associated with the patient's demographic characteristics. More specifically, which demographic factors influence patient length of stay? Finally, we hoped to determine whether length of patient hospital stay is associated with patient mortality. For this question, we examined both patients' total hospital stay and patients' stay in the ICU. Our work to explore these questions is detailed in the Exploratory Analysis and Additional Analysis sections of the report.

Exploratory Analysis
--------------------

The raw admissions dataset consists of 58976 observations of the following 19 variables:

-   Row ID
-   Subject ID
-   HADM ID (hospital admission ID)
-   Admit time
-   Discharge time
-   Death time
-   Admission type
-   Admission location
-   Discharge location
-   Insurance
-   Language
-   Religion
-   Marital status
-   Ethnicity
-   ED REG time
-   ED OUT time
-   Diagnosis
-   Hospital expire flag
-   Has chart events

*Initial exploration*

-   71.34% of patients admitted were classified as emergencies, while 13.33% were newborns and 71.34% were elective. The remaining 2.26% were classified as urgent.

-   47.84% of patients had Medicare, while 38.29% had private insurance and 9.81% had Medicaid. The remaining patients either were insured by the government or paid out of pocket.

-   41.1% of the critical care patients were married, 22.47% of patients were single, 12.23% were widowed and 5.45% were divorced. The remaining 18.75% were either separated, with a life partner, or marital status was unknown.

-   42.95% of the patients were missing data entries for language. Of the patients that did have language entries, 49.32% of them spoke English.

For the scope of this project, we are especially interested in focusing on patient mortalities and length of hospital stay. The depth of our initial analysis is concentrated here.

Out of the 58976 patient admissions, 90.07% patients were ultimately discharged. The remaining 9.93% were recorded as patient deaths.

``` r
icu_data = read_csv("./database/data/icu_detail.csv") %>% 
  filter(!(los_hospital < 0), !(los_icu < 0))

ggplot(icu_data, aes(x = los_icu, y = los_hospital)) +
  geom_point() +
  labs(
    x = "Length of Stay in ICU, in days", 
    y = "Total Length of Stay in Hospital, in days" 
  )
```

![](report_files/figure-markdown_github/unnamed-chunk-8-1.png)

**Let's explain the process of making simpler diagnostic groups** Then include top 10 diagnoses associated with death.

We just looked into relationship between different factors and `living` so we try different SLR to try to look into the relationship.

And we make the time form normal since first the year was added 200.

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
    ##    <int>      <int>   <int>          <dbl> <chr>           <chr>        
    ## 1     21         22  165315           1996 04              09 12:26:00  
    ## 2     22         23  152223           1953 09              03 07:15:00  
    ## 3     23         23  124321           1957 10              18 19:34:00  
    ## 4     24         24  161859           1939 06              06 16:14:00  
    ## 5     25         25  129635           1960 11              02 02:06:00  
    ## 6     26         26  197661           1926 05              06 15:16:00  
    ## # … with 27 more variables: dischtime_year <dbl>, dischtime_month <chr>,
    ## #   dischtime_day <chr>, dischtime_time <chr>, deathtime_year <dbl>,
    ## #   deathtime_month <chr>, deathtime_day <chr>, deathtime_time <chr>,
    ## #   admission_type <chr>, admission_location <chr>,
    ## #   discharge_location <chr>, insurance <chr>, language <chr>,
    ## #   religion <chr>, marital_status <chr>, ethnicity <chr>,
    ## #   edregtime_year <dbl>, edregtime_month <chr>, edregtime_day <chr>,
    ## #   edregtime_time <chr>, edouttime_year <dbl>, edouttime_month <chr>,
    ## #   edouttime_day <chr>, edouttime_time <chr>, diagnosis <fct>,
    ## #   hospital_expire_flag <int>, has_chartevents_data <int>

\*\* First we need to filter newborn out because the newborn is different from other diagnosis.

Additional Analysis
-------------------

Discussion
----------

We were very ambitious and there were some limitations to using such an ambitious dataset. For example, ...

Conclusion
----------
