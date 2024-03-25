Statistical Modelling - Prediction
================
Carina Fan<br/>
Last updated: March 25, 2024

- [Set up](#set-up)
- [Predict outcome from hypothetical predictor
  values](#predict-outcome-from-hypothetical-predictor-values)
- [Export R script](#export-r-script)
- [Session info](#session-info)

After a regression model has been built, this script allows users to
predict the value of their outcome variable using hypothetical values on
the predictors.

<!-- ======================================================================= -->

# Set up

Load the data from the model building script:

User-inputted variables:

- `predictor1_value`
- `predictor2_value`

Requirements for user-inputted variables:

- the predictor values are hypothetical values the user would want to
  set, in order to predict a value on the outcome variable (e.g., “if I
  set age to 65 and gender to Male, what are the odds of having a
  stroke?”)
- `predictor2_value` should be FALSE by default and only given a value
  if the `predictor2 != FALSE` (in the model building script)

<!-- ======================================================================= -->

# Predict outcome from hypothetical predictor values

Right now, if the covariate is categorical, then I just select the first
level of the category to use as the prediction value, and if it’s
continuous then I set it as the mean (0). We will probably want to
change this to either let the user select the covariate value, or maybe
select the modal factor.

    ## [1] "The predicted outcome for Churn is N with a probability of 80.28%."

<!-- ======================================================================= -->

# Export R script

<!-- ======================================================================= -->

# Session info

    ## R version 4.3.3 (2024-02-29 ucrt)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 10 x64 (build 19045)
    ## 
    ## Matrix products: default
    ## 
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_Canada.utf8  LC_CTYPE=English_Canada.utf8   
    ## [3] LC_MONETARY=English_Canada.utf8 LC_NUMERIC=C                   
    ## [5] LC_TIME=English_Canada.utf8    
    ## 
    ## time zone: America/Toronto
    ## tzcode source: internal
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] lubridate_1.9.3  forcats_1.0.0    stringr_1.5.1    dplyr_1.1.4     
    ##  [5] purrr_1.0.2      readr_2.1.4      tidyr_1.3.1      tibble_3.2.1    
    ##  [9] tidyverse_2.0.0  magrittr_2.0.3   ggfortify_0.4.16 ggplot2_3.5.0   
    ## [13] broom_1.0.5     
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] gtable_0.3.4      compiler_4.3.3    tidyselect_1.2.1  gridExtra_2.3    
    ##  [5] scales_1.3.0      yaml_2.3.8        fastmap_1.1.1     R6_2.5.1         
    ##  [9] generics_0.1.3    knitr_1.45        backports_1.4.1   munsell_0.5.0    
    ## [13] tzdb_0.4.0        pillar_1.9.0      rlang_1.1.2       utf8_1.2.4       
    ## [17] stringi_1.8.3     xfun_0.41         timechange_0.2.0  cli_3.6.2        
    ## [21] withr_3.0.0       digest_0.6.33     grid_4.3.3        rstudioapi_0.15.0
    ## [25] hms_1.1.3         lifecycle_1.0.4   vctrs_0.6.5       evaluate_0.23    
    ## [29] glue_1.6.2        fansi_1.0.6       colorspace_2.1-0  rmarkdown_2.25   
    ## [33] tools_4.3.3       pkgconfig_2.0.3   htmltools_0.5.7
