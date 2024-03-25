Statistical Modelling - Model Building
================
Carina Fan<br/>
Last updated: March 25, 2024

- [Set up](#set-up)
  - [Convert numeric variables to factors, as specified by user upload;
    convert any binary numeric variables to factor; convert effect-coded
    variable to be
    dummy-coded](#convert-numeric-variables-to-factors-as-specified-by-user-upload-convert-any-binary-numeric-variables-to-factor-convert-effect-coded-variable-to-be-dummy-coded)
  - [Subset data](#subset-data)
  - [Check categorical variables](#check-categorical-variables)
  - [Centre continuous variables](#centre-continuous-variables)
  - [Determine analysis to run](#determine-analysis-to-run)
- [Print descriptive information about
  data](#print-descriptive-information-about-data)
- [Build model](#build-model)
- [Export R script and workspace](#export-r-script-and-workspace)
- [Session info](#session-info)

This script builds a regression model, outputs the results, and runs
model fit metrics.

The type of regression that should be run depends on the type of the
outcome variable. This script will look at the user’s selected outcome
and automatically determine the correct regression, and it includes
logic to to build & return the appropriate models. The following
analyses are able to be implemented at this time:

- linear regression (for continuous outcome variables)
- logistic regression (for binary outcome variables)
- multinomial logistic regression (for multiclass outcome variables)

Analyses to think about adding:

- poisson regression (for counts/frequencies)
- zero-inflated poisson regression (for counts with excess zeroes)
- beta regression (for proportions/probabilities)

<!-- ======================================================================= -->

# Set up

User-inputted variables:

- `outcome`
- `predictor1`
- `predictor2`
- `covariate`

Requirements for user-inputted variables:

- `outcome` must be selected
- `predictor1` must be selected
- `predictor2` and `covariate` are optional; if they are not selected,
  they should be set as `FALSE` by default
- all variables must have more than 1 unique value; this script checks
  for that and will print an error message, but on the front end we need
  to make sure that the rest of the script doesn’t run after that point
  (or it will break)
- if the user selects a binary outcome variable, then they should also
  be given the option to indicate which level of that variable
  represents the “positive” class. if they choose not to indicate the
  positive class, or if they select a different class of outcome
  variable, `pos_class` should be set to `FALSE`.
- if the user selects a multiclass categorical outcome variable, then
  they should also be given the option to select a reference level. if
  they choose not to select a reference level, or if they select a
  different class of outcome variable, `ref_level` should be set to
  `FALSE`.
- `convertToFactor` is a list of any columns/predictors that should have
  their column type switched from numeric to categorical. The user would
  specify these changes at the upload phase and then those columns will
  have to be piped through to each R script flow.

Notes on how analysis will be run depending on predictors:

- all continuous predictors (including covariate) will be centered
- if 2 predictors are selected & the analysis is linear or logistic,
  their interaction will be included
- if 2 predictors are selected & the analysis is multinomial logistic,
  their interaction will not be included (it complicates the model
  output hugely to have interaction effects for all combinations of
  levels of the input/output variables)
- no interactions with the covariate will be included

## Convert numeric variables to factors, as specified by user upload; convert any binary numeric variables to factor; convert effect-coded variable to be dummy-coded

## Subset data

## Check categorical variables

We may need a script break at this point to allow for error printing &
for the user to go back to select new variables if necessary.

## Centre continuous variables

## Determine analysis to run

<!-- ======================================================================= -->

# Print descriptive information about data

Number of rows for analysis:

    ## [1] "After removing missing data, there are 2000 rows in the dataset."

Range/levels of variables:

    ## [1] "The levels of your outcome variable, Churn, are: N, Y."

    ## [1] "The levels of your outcome variable, Churn, are: N, Y."

    ## [1] "The levels of your second predictor, CompanySize, are: L, M, S."

<!-- ======================================================================= -->

# Build model

Set up regression formula based on selected combination of predictors
and covariate:

Source appropriate analysis script:

    ## [1] The "positive" result for Churn is Y. This means that any change in the odds of Churn predicted by the model is a change in the odds of Churn being Y.
    ## [1] "Feature1 was statistically significant in predicting Churn."
    ## [1] "The reference level of Feature1 is N. The table below shows you the change in odds of Churn predicted by each other level of Feature1 compared to the reference level."
    ##   Change in odds
    ## Y        -91.41%
    ## [1] "CompanySize did not statistically significantly predict Churn."
    ## [1] "The interaction between Feature1 and CompanySize was not statistically significant in predicting Churn."
    ## [1] "The pseudo R-squared value for your model is 0.193. This value ranges from 0 to 1, with values closer to 1 indicating a better model fit."
    ## [1] Sensitivity is the "true positive" rate, or the percentage of observations the model correctly predicted for the "positive" result. In this case, the "positive" result for your outcome variable, Churn, is Y. The sensitivity of the model is 53.2%.
    ## [1] Specificity is the "true negative" rate, or the percentage of observations the model correctly predicted for the "negative" result. In this case, the "negative" result for your outcome variable, Churn, is N. The specificity of the model is 91.8%.
    ## [1] "Misclassification error is the percentage of observations the model predicted the outcome incorrectly for. The total misclassification error rate for your model is 31.7%."

<!-- ======================================================================= -->

# Export R script and workspace

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
    ##  [1] lubridate_1.9.3        forcats_1.0.0          stringr_1.5.1         
    ##  [4] dplyr_1.1.4            purrr_1.0.2            readr_2.1.4           
    ##  [7] tidyr_1.3.1            tibble_3.2.1           tidyverse_2.0.0       
    ## [10] magrittr_2.0.3         yardstick_1.2.0        reshape2_1.4.4        
    ## [13] pscl_1.5.5.1           nnet_7.3-19            moments_0.14.1        
    ## [16] jsonlite_1.8.8         InformationValue_1.3.1 ggfortify_0.4.16      
    ## [19] ggplot2_3.5.0          car_3.1-2              carData_3.0-5         
    ## [22] broom_1.0.5           
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] tidyselect_1.2.1     timeDate_4032.109    fastmap_1.1.1       
    ##  [4] pROC_1.18.5          caret_6.0-94         digest_0.6.33       
    ##  [7] rpart_4.1.23         timechange_0.2.0     lifecycle_1.0.4     
    ## [10] survival_3.5-8       compiler_4.3.3       rlang_1.1.2         
    ## [13] tools_4.3.3          utf8_1.2.4           yaml_2.3.8          
    ## [16] data.table_1.14.10   knitr_1.45           plyr_1.8.9          
    ## [19] abind_1.4-5          withr_3.0.0          stats4_4.3.3        
    ## [22] grid_4.3.3           fansi_1.0.6          colorspace_2.1-0    
    ## [25] future_1.33.1        globals_0.16.2       scales_1.3.0        
    ## [28] iterators_1.0.14     MASS_7.3-60.0.1      cli_3.6.2           
    ## [31] rmarkdown_2.25       generics_0.1.3       rstudioapi_0.15.0   
    ## [34] future.apply_1.11.1  tzdb_0.4.0           splines_4.3.3       
    ## [37] parallel_4.3.3       vctrs_0.6.5          hardhat_1.3.0       
    ## [40] Matrix_1.6-5         hms_1.1.3            listenv_0.9.0       
    ## [43] foreach_1.5.2        gower_1.0.1          recipes_1.0.9       
    ## [46] glue_1.6.2           parallelly_1.36.0    codetools_0.2-19    
    ## [49] stringi_1.8.3        gtable_0.3.4         munsell_0.5.0       
    ## [52] pillar_1.9.0         htmltools_0.5.7      ipred_0.9-14        
    ## [55] lava_1.7.3           R6_2.5.1             evaluate_0.23       
    ## [58] lattice_0.22-5       backports_1.4.1      class_7.3-22        
    ## [61] Rcpp_1.0.11          gridExtra_2.3        nlme_3.1-164        
    ## [64] prodlim_2023.08.28   xfun_0.41            pkgconfig_2.0.3     
    ## [67] ModelMetrics_1.2.2.2
