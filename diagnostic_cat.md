Diagnostic Categories
================
Laura Cosgrove
12/4/2018

This R code makes use of the Clinical Classification Software (CCS), which categorizes ICD-9 coded diagnoses into clinically meaningful groups. The categorization was developed by the Agency for Healthcare Research and Quality (AHRQ). More detail can be found on the AHRQ website: <https://www.hcup-us.ahrq.gov/tools_software.jsp>.

This software contains two tables: `ccs_single_level_dx.csv.gz` and `ccs_multi_level_dx.csv.gz`. The first table contains high-level diagnostic category infomation, while the multi-level diagnostic category information contains four levels of diagnostic detail.

The purpose of reading in AHRQ's clinical categories is to help make diagnosis descriptions for our data more intuitive.

Read in data
------------

``` r
multi_ccs <- read_csv("./database/mimic-code/concepts/diagnosis/ccs_multi_level.csv.gz",
                      col_names = c("ccs_cat_multi", "description_multi_1", "description_multi_2", "description_multi_3", "description_multi_4","icd9_code"))
```

    ## Parsed with column specification:
    ## cols(
    ##   ccs_cat_multi = col_double(),
    ##   description_multi_1 = col_character(),
    ##   description_multi_2 = col_character(),
    ##   description_multi_3 = col_character(),
    ##   description_multi_4 = col_character(),
    ##   icd9_code = col_character()
    ## )

    ## Warning in rbind(names(probs), probs_f): number of columns of result is not
    ## a multiple of vector length (arg 1)

    ## Warning: 4039 parsing failures.
    ## row # A tibble: 5 x 5 col     row col       expected        actual file                              expected   <int> <chr>     <chr>           <chr>  <chr>                             actual 1  2195 ccs_cat_… no trailing ch… .2     './database/mimic-code/concepts/… file 2  2196 ccs_cat_… no trailing ch… .2     './database/mimic-code/concepts/… row 3  2197 ccs_cat_… no trailing ch… .2     './database/mimic-code/concepts/… col 4  2198 ccs_cat_… no trailing ch… .2     './database/mimic-code/concepts/… expected 5  2199 ccs_cat_… no trailing ch… .2     './database/mimic-code/concepts/…
    ## ... ................. ... .......................................................................... ........ .......................................................................... ...... .......................................................................... .... .......................................................................... ... .......................................................................... ... .......................................................................... ........ ..........................................................................
    ## See problems(...) for more details.

``` r
single_ccs <- read_csv("./database/mimic-code/concepts/diagnosis/ccs_single_level.csv.gz",
                       col_names = c("ccs_cat_single", "description_single", "icd9_code"))
```

    ## Parsed with column specification:
    ## cols(
    ##   ccs_cat_single = col_integer(),
    ##   description_single = col_character(),
    ##   icd9_code = col_character()
    ## )

``` r
d_icd_diagnoses <- read_csv("./database/data/D_ICD_DIAGNOSES.csv.gz") %>% 
  janitor::clean_names()
```

    ## Parsed with column specification:
    ## cols(
    ##   ROW_ID = col_integer(),
    ##   ICD9_CODE = col_character(),
    ##   SHORT_TITLE = col_character(),
    ##   LONG_TITLE = col_character()
    ## )

### Inner join

``` r
d_icd_diagnoses_single <- d_icd_diagnoses %>% 
  inner_join(., single_ccs, by = "icd9_code") 

d_icd_diagnoses_multi <- d_icd_diagnoses %>% 
  inner_join(., multi_ccs, by = "icd9_code") 
```

Now, the diagnosis coding the hospital uses is given overall diagnostic categories.

Join with the patient data
--------------------------

``` r
diagnoses_icd <- read_csv("./database/data/DIAGNOSES_ICD.csv.gz") %>% 
  janitor::clean_names()
```

    ## Parsed with column specification:
    ## cols(
    ##   ROW_ID = col_integer(),
    ##   SUBJECT_ID = col_integer(),
    ##   HADM_ID = col_integer(),
    ##   SEQ_NUM = col_integer(),
    ##   ICD9_CODE = col_character()
    ## )

``` r
diagnoses_icd_single_cat <- diagnoses_icd %>% 
  inner_join(., d_icd_diagnoses_single, by = "icd9_code") 

#Our high-level categories
diagnoses_icd_single_cat %>% 
  distinct(description_single)
```

    ## # A tibble: 281 x 1
    ##    description_single                                                      
    ##    <chr>                                                                   
    ##  1 Hypertension with complications and secondary hypertension              
    ##  2 Pneumonia (except that caused by tuberculosis or sexually transmitted d…
    ##  3 Nephritis; nephrosis; renal sclerosis                                   
    ##  4 Chronic kidney disease                                                  
    ##  5 Peri-; endo-; and myocarditis; cardiomyopathy (except that caused by tu…
    ##  6 Fluid and electrolyte disorders                                         
    ##  7 Systemic lupus erythematosus and connective tissue disorders            
    ##  8 Spondylosis; intervertebral disc disorders; other back problems         
    ##  9 Complications of surgical procedures or medical care                    
    ## 10 Coagulation and hemorrhagic disorders                                   
    ## # ... with 271 more rows

``` r
#The EMR high-level categories
diagnoses_icd_single_cat %>% 
  distinct(short_title)
```

    ## # A tibble: 6,769 x 1
    ##    short_title             
    ##    <chr>                   
    ##  1 Mal hyp kid w cr kid V  
    ##  2 Pneumonia, organism NOS 
    ##  3 Chr nephritis in oth dis
    ##  4 Chron kidney dis stage V
    ##  5 Prim cardiomyopathy NEC 
    ##  6 Acidosis                
    ##  7 Syst lupus erythematosus
    ##  8 Hyperpotassemia         
    ##  9 Sciatica                
    ## 10 Iatrogenc hypotnsion NEC
    ## # ... with 6,759 more rows

``` r
#Unique ICD 9 codes: about the same dimension as the short titles for diagnoses
diagnoses_icd_single_cat %>% 
  distinct(icd9_code)
```

    ## # A tibble: 6,840 x 1
    ##    icd9_code
    ##    <chr>    
    ##  1 40301    
    ##  2 486      
    ##  3 58281    
    ##  4 5855     
    ##  5 4254     
    ##  6 2762     
    ##  7 7100     
    ##  8 2767     
    ##  9 7243     
    ## 10 45829    
    ## # ... with 6,830 more rows

``` r
diagnoses_icd_single_cat %>% 
  distinct(short_title)
```

    ## # A tibble: 6,769 x 1
    ##    short_title             
    ##    <chr>                   
    ##  1 Mal hyp kid w cr kid V  
    ##  2 Pneumonia, organism NOS 
    ##  3 Chr nephritis in oth dis
    ##  4 Chron kidney dis stage V
    ##  5 Prim cardiomyopathy NEC 
    ##  6 Acidosis                
    ##  7 Syst lupus erythematosus
    ##  8 Hyperpotassemia         
    ##  9 Sciatica                
    ## 10 Iatrogenc hypotnsion NEC
    ## # ... with 6,759 more rows

``` r
#Another comparision, initial diagnosis from admissions dataset: > 15,000 unique
admissions <- read_csv("./database/data/ADMISSIONS.csv.gz") %>% 
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
admissions %>% 
  distinct(diagnosis)
```

    ## # A tibble: 15,647 x 1
    ##    diagnosis                                                  
    ##    <chr>                                                      
    ##  1 BENZODIAZEPINE OVERDOSE                                    
    ##  2 "CORONARY ARTERY DISEASE\\CORONARY ARTERY BYPASS GRAFT/SDA"
    ##  3 BRAIN MASS                                                 
    ##  4 INTERIOR MYOCARDIAL INFARCTION                             
    ##  5 ACUTE CORONARY SYNDROME                                    
    ##  6 V-TACH                                                     
    ##  7 NEWBORN                                                    
    ##  8 "UNSTABLE ANGINA\\CATH"                                    
    ##  9 STATUS EPILEPTICUS                                         
    ## 10 TRACHEAL STENOSIS/SDA                                      
    ## # ... with 15,637 more rows

``` r
# Check if the sheet succesfully joins to admissions. Should be multiple diagnoses per subject
admissions %>% 
  inner_join(., diagnoses_icd_single_cat, by = "hadm_id") %>% 
  select(hadm_id, diagnosis, icd9_code, short_title, description_single) %>% 
  group_by(hadm_id)
```

    ## # A tibble: 634,377 x 5
    ## # Groups:   hadm_id [58,925]
    ##    hadm_id diagnosis         icd9_code short_title   description_single   
    ##      <int> <chr>             <chr>     <chr>         <chr>                
    ##  1  165315 BENZODIAZEPINE O… 9678      Pois-sedativ… Poisoning by other m…
    ##  2  165315 BENZODIAZEPINE O… 9693      Poison-antip… Poisoning by psychot…
    ##  3  165315 BENZODIAZEPINE O… E9502     Poison-sedat… Suicide and intentio…
    ##  4  165315 BENZODIAZEPINE O… E9503     Poison-psych… Suicide and intentio…
    ##  5  165315 BENZODIAZEPINE O… 29620     Depress psyc… Mood disorders       
    ##  6  165315 BENZODIAZEPINE O… 4019      Hypertension… Essential hypertensi…
    ##  7  152223 "CORONARY ARTERY… 41401     Crnry athrsc… Coronary atheroscler…
    ##  8  152223 "CORONARY ARTERY… 4111      Intermed cor… Coronary atheroscler…
    ##  9  152223 "CORONARY ARTERY… 4241      Aortic valve… Heart valve disorders
    ## 10  152223 "CORONARY ARTERY… V4582     Status-post … Coronary atheroscler…
    ## # ... with 634,367 more rows

We've succesfully reduced the complexity of the high-level diagnostic categories. We may be missing some important data, but we could try to remedy that, in part, by joining the multi\_ccs dataset to our patient data. But for now, I will export the short datasheet of patients + diagnosis category so that my team members can join it to admissions data for their analyses.

Export diagnosis with subject data and admissions diagnosis
-----------------------------------------------------------

``` r
subject_diag_cat <- admissions %>% 
  inner_join(., diagnoses_icd_single_cat, by = "hadm_id") %>% 
  select(hadm_id, diagnosis, icd9_code, short_title, description_single) %>% 
  rename(admit_diagnosis = diagnosis, category = description_single) %>% 
  group_by(hadm_id)

write_csv(subject_diag_cat, "./database/subject_diag_cat.csv")
```
