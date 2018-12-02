Influential Factors in Critical Care Patients
================
Samantha Brown
12/2/2018

Group members: Samantha Brown (UNI: slb2240), Laura Cosgrove (UNI: lec2197), and Francis Z. Fang (UNI: zf2211).

### Introduction

Critical care involves the specialized treatment of patients whose conditions pose life-threatening risks and require around-the-clock care. Critical care treatment typically takes place in an intensive care unit (ICU) of a hospital. Due to the nature of critical care, many patients eventually recover, but some die. For the purpose of this project, we seek to explore factors that influence critical care patients.

### Motivation and Related Work

Previous research has focused on the physiologic- and disease-driven factors that influence critical care. In this report, we consider whether demographic characteristics of patients in critical care influence outcomes such as mortality and length of hospital stay.

Related work?

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
-   HADM ID
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

Out of 58976 patient admissions, 53122 patients were ultimately discharged. The remaining 9.93% were recorded as deaths.

Additional Analysis
-------------------

Discussion
----------

Conclusion
----------
