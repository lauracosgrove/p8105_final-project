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

MIMIC is an openly accessible critical care database created by the MIT Lab for Computational Physiology. It comprises deidentified health-related data associated with over forty thousand patients who stayed in critical care units of the Beth Israel Deaconess Medical Center between 2001 and 2012. It includes the following information: demographics, vital sign measurements made at the bedside (~1 data point per hour), laboratory test results, procedures, medications, caregiver notes, imaging reports, and mortality (both in and out of hospital). After completing the CITI “Data or Specimens Only Research” training course, PhysioNet granted us access to the MIMIC datasets.

The next step was to follow the

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

``` r
## Distribution of insurance type
admissions %>% 
  group_by(insurance) %>% 
  count() %>% 
  mutate(percent_of_patients = round(n/nrow(admissions)*100, digits = 2)) %>% 
  arrange(desc(percent_of_patients)) %>% 
  select(-n)
```

    ## # A tibble: 5 x 2
    ## # Groups:   insurance [5]
    ##   insurance  percent_of_patients
    ##   <chr>                    <dbl>
    ## 1 Medicare                 47.8 
    ## 2 Private                  38.3 
    ## 3 Medicaid                  9.81
    ## 4 Government                3.02
    ## 5 Self Pay                  1.04

41.1% of the critical care patients were married, 22.47% of patients were single, 12.23 were widowed and 5.45 were divorced. The remaining 18.75% were either separated, with a life partner, or marital status was unknown.

For the scope of this project, we are especially interested in focusing on patient mortalities and length of hospital stay. The depth of our initial analysis is concentrated here.

Out of the 58976 patient admissions, 90.07% patients were ultimately discharged. The remaining 9.93% were recorded as patient deaths.

**Let's explain the process of making simpler diagnostic groups** Then include top 10 diagnoses associated with death.

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

![](report_files/figure-markdown_github/unnamed-chunk-5-1.png)

Additional Analysis
-------------------

Discussion
----------

We were very ambitious and there were some limitations to using such an ambitious dataset. For example, ...

Conclusion
----------
