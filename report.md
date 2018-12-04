Influential Factors in Critical Care Patients
================
Samantha Brown, Laura Cosgrove, Francis Z. Fang
12/2/2018

Group members: Samantha Brown (UNI: slb2240), Laura Cosgrove (UNI: lec2197), and Francis Z. Fang (UNI: zf2211).

### Introduction

Critical care involves the specialized treatment of patients whose conditions pose life-threatening risks and require around-the-clock care. Critical care treatment typically takes place in an intensive care unit (ICU) of a hospital. Due to the nature of critical care, many patients eventually recover, but some die. For the purpose of this project, we seek to explore factors that influence critical care patients.

### Motivation and Related Work

Previous research has focused on the physiologic- and disease-driven factors that influence critical care. In this report, we consider whether demographic characteristics of patients in critical care influence outcomes such as mortality and length of hospital stay.

Related work includes a 2015 NIH research publication titled *"Mortality prediction in the ICU: can we do better? Results from the Super ICU Learner Algorithm (SICULA) project, a population-based study"*, which considers whether a machine learning technique can help predict mortality for patients in critical care.

Data Collection
---------------

**Include instructions for where to access the data (through somewhere online like a google drive or downloadable link)**

MIMIC is an openly accessible critical care database created by the MIT Lab for Computational Physiology. It comprises deidentified health-related data associated with over forty thousand patients who stayed in critical care units of the Beth Israel Deaconess Medical Center between 2001 and 2012. It includes the following information: demographics, vital sign measurements made at the bedside (~1 data point per hour), laboratory test results, procedures, medications, caregiver notes, imaging reports, and mortality (both in and out of hospital). After completing the CITI “Data or Specimens Only Research” training course, PhysioNet granted us access to the MIMIC datasets.

Initial Questions
-----------------

Exploratory Analysis
--------------------

``` r
admissions <- 
  read_csv("./database/data/admissions.csv") %>% 
  janitor::clean_names() %>% 
  mutate(diagnosis = factor(diagnosis))
```

The raw admissions dataset consists of 58976 observations of 19 variables:

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

Out of the 58976 patient admissions, 90.07% patients were ultimately discharged. The remaining 9.93% were recorded as patient deaths.

For the scope of this project, we are especially interested in focusing on patient mortalities and length of hospital stay. The depth of our initial analysis is concentrated here.

The 10 most frequent diagnoses associated with patient mortalities are shown below:

``` r
admissions %>% 
  filter(deathtime > 0) %>%
  count(diagnosis) %>% 
  top_n(10) %>% 
  arrange(desc(n))
```

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
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
difference = admissions %>% 
  mutate(length_of_stay =  lubridate::as.duration(admittime %--% dischtime),
         mortality = ifelse(is.na(deathtime), 1, 0))

summary(difference$length_of_stay)
```

    ##                              Min.                           1st Qu. 
    ##           "81660s (~22.68 hours)"            "323460s (~3.74 days)" 
    ##                            Median                              Mean 
    ##            "558750s (~6.47 days)" "875570.349294628s (~1.45 weeks)" 
    ##                           3rd Qu.                              Max. 
    ##          "1019100s (~1.69 weeks)"        "25458660s (~42.09 weeks)"

Additional Analysis
-------------------

Discussion
----------

We were very ambitious and there were some limitations to using such an ambitious dataset. For example, ...

Conclusion
----------
