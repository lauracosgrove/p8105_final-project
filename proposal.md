Proposal
================
Laura Cosgrove
11/6/2018

Proposal
--------

### The group members

The members of this group are Samantha Brown (UNI: slb2240), Laura Cosgrove (UNI: lec2197), and Francis Z. Fang (UNI: zf2211).

### The tentative project title

Influential Factors in Critical Care Patients

### The motivation for this project

In critical care, we assume that the factors influencing care are limited to physiologic and disease characteristics. For this project, we’d like to explore if demographic characteristics like ethnicity, language, and insurance coverage have associations with outcomes like number of prescribed medicines and mortality. If we find a significant association, we can further explore if this difference in outcomes is driven by clinical factors like diagnosis or number of chart events rather than demographic factors. Finally, for the most commonly prescribed medicines in the ICU, we plan to compare any adverse chart events experienced by the patients with adverse events reported in to the FDA for the same active ingredients over the same period of time to answer the question of whether adverse medication events are reflected in this critical care dataset.

### The intended final products

We will create a report that allows for exploration of the association between demographic and other characteristics with mortality and length-of-care outcomes. We will aim to include as many interactive plots as possible to allow for user exploration of the data.

### The anticipated data sources

MIMIC-physio net is a critical care database used in many open research projects. It comprises deidentified health-related data associated with over forty thousand patients who stayed in critical care units of the Beth Israel Deaconess Medical Center between 2001 and 2012. It includes the following information: demographics, vital sign measurements made at the bedside (~1 data point per hour), laboratory test results, procedures, medications, caregiver notes, imaging reports, and mortality (both in and out of hospital). We are making use of the open-access [support](https://github.com/MIT-LCP/mimic-code) for the dataset to create a local database in order to query the data. We’ve applied for access to the datasets through PhysioNet.

OpenFDA provides APIs to a number of high-value, high priority and scalable structured datasets, including adverse events, drug product labeling, and recall enforcement reports. We plan to use [this R package](https://github.com/rOpenHealth/openfda) to support common-sense queries of the data.

### The planned analyses / visualizations / coding challenges

1.  Multiple Linear Regression: Looking at predictors of death or length of stay for particular diseases.

2.  We'll consider plots to check for heteroscedasticity, normality, and influential patient observations

3.  A coding challenge will be merging the FDA and the MIMIC datasets.

### The planned timeline

Our planned timeline is as follows:

-   Sunday, November 11: wrangling meeting (clean and tidy data; divie up analyses)
-   Thursday, November 15: meet to discuss individual initial analyses
-   Thursday, November 15 - Thursday November 29: continue/finish analyses
-   Thursday, November 29: combine our analyses to make a synthesized report
-   Sunday, December 2: make webpage and screencast
-   Thursday, December 6: report, webpage, screencast, peer assessments due
-   Tuesday, December 11: in-class discussion of projects
