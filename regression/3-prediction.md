Statistical Modelling - Beginner Tier - Prediction
================
Carina Fan (<carina@dacture.com>)<br/>
Last updated: January 09, 2024

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

``` r
library(broom)
library(ggfortify)
```

    ## Loading required package: ggplot2

``` r
library(magrittr)
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.3     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ lubridate 1.9.2     ✔ tibble    3.2.1
    ## ✔ purrr     1.0.2     ✔ tidyr     1.3.0

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ tidyr::extract()   masks magrittr::extract()
    ## ✖ dplyr::filter()    masks stats::filter()
    ## ✖ dplyr::lag()       masks stats::lag()
    ## ✖ purrr::set_names() masks magrittr::set_names()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

Load the data from the model building script:

``` r
load("1-model-building.RData")
```

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

``` r
predictor1_value = 90
predictor2_value = 'N'
```

<!-- ======================================================================= -->

# Predict outcome from hypothetical predictor values

Right now, if the covariate is categorical, then I just select the first
level of the category to use as the prediction value, and if it’s
continuous then I set it as the mean (0). We will probably want to
change this to either let the user select the covariate value, or maybe
select the modal factor.

``` r
# centre continuous predictors and create dataframe of explanatory data

## centre predictor 1
if (is.numeric(df.analysis[[predictor1]])) {
  predictor1_value = predictor1_value - mean(df.analysis.uncentered[[predictor1]])
}

## initialize explanatory data frame
explanatory_data = data.frame(predictor1_value) %>%
  rename({{predictor1}} := "predictor1_value")

## centre predictor2 and add to explanatory data
if (predictor2 != FALSE) {
  if (is.numeric(df.analysis[[predictor2]])) {
    predictor2_value = predictor2_value - mean(df.analysis.uncentered[[predictor2]])
  }
  explanatory_data %<>% data.frame(predictor2_value) %>%
    rename({{predictor2}} := "predictor2_value")
}

## set covariate to either mean or first level, then add to explanatory data
if (covariate != FALSE) {
  if (is.numeric(df.analysis[[covariate]])) { # if covariate is continuous
    # set covariate value as mean of covariate (0 since covariate is automatically mean-centered)
    covariate_value = 0
  } else if (is.factor(df.analysis[[covariate]])) { # if covariate is categorical
    # set covariate value as first level of covariate
    covariate_value = levels(df.analysis[[covariate]])[1]
  }
  explanatory_data %<>% data.frame(covariate_value) %>%
    rename({{covariate}} := "covariate_value")
} 

# run prediction
if (analysis == "logistic") {
  
  prediction_data = explanatory_data |> 
    mutate(outcome = predict(model, explanatory_data, type = "response"),
           most_likely_outcome = round(outcome),
           odds_ratio = outcome / (1 - outcome),
           one_minus_outcome = 1-outcome)
  
  if (prediction_data$most_likely_outcome == 0) {
    predicted_outcome = levels(df.analysis[[outcome]])[1]
  } else {
    predicted_outcome = levels(df.analysis[[outcome]])[2]
  }
  
  if (predicted_outcome == pos_class) {
    prob = prediction_data$outcome*100
  } else {
    prob = prediction_data$one_minus_outcome*100
  }
  
  paste0("The predicted outcome for ", outcome, " is ", predicted_outcome, " with a probability of ", prob |> round(2), "%.") |>
    print()

  } else if (analysis == "linear") {
    
  prediction_data = explanatory_data |> 
    mutate(outcome = predict(model, explanatory_data))
  
  paste0("The predicted value of ", outcome, " is ", prediction_data$outcome |> round(2), ".") |> 
    print()
  
}
```

    ## [1] "The predicted outcome for Churn is Y with a probability of 99.88%."

<!-- ======================================================================= -->

# Export R script

``` r
knitr::purl("3-prediction.Rmd")
```

<!-- ======================================================================= -->

# Session info

``` r
sessionInfo()
```

    ## R version 4.3.1 (2023-06-16)
    ## Platform: aarch64-apple-darwin20 (64-bit)
    ## Running under: macOS Sonoma 14.2.1
    ## 
    ## Matrix products: default
    ## BLAS:   /Library/Frameworks/R.framework/Versions/4.3-arm64/Resources/lib/libRblas.0.dylib 
    ## LAPACK: /Library/Frameworks/R.framework/Versions/4.3-arm64/Resources/lib/libRlapack.dylib;  LAPACK version 3.11.0
    ## 
    ## locale:
    ## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
    ## 
    ## time zone: America/Edmonton
    ## tzcode source: internal
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] lubridate_1.9.2  forcats_1.0.0    stringr_1.5.0    dplyr_1.1.3     
    ##  [5] purrr_1.0.2      readr_2.1.4      tidyr_1.3.0      tibble_3.2.1    
    ##  [9] tidyverse_2.0.0  magrittr_2.0.3   ggfortify_0.4.16 ggplot2_3.4.4   
    ## [13] broom_1.0.5     
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] gtable_0.3.4      compiler_4.3.1    tidyselect_1.2.0  gridExtra_2.3    
    ##  [5] scales_1.2.1      yaml_2.3.7        fastmap_1.1.1     R6_2.5.1         
    ##  [9] generics_0.1.3    knitr_1.45        backports_1.4.1   munsell_0.5.0    
    ## [13] tzdb_0.4.0        pillar_1.9.0      rlang_1.1.1       utf8_1.2.4       
    ## [17] stringi_1.7.12    xfun_0.40         timechange_0.2.0  cli_3.6.1        
    ## [21] withr_2.5.2       digest_0.6.33     grid_4.3.1        rstudioapi_0.15.0
    ## [25] hms_1.1.3         lifecycle_1.0.3   vctrs_0.6.4       evaluate_0.21    
    ## [29] glue_1.6.2        fansi_1.0.5       colorspace_2.1-0  rmarkdown_2.24   
    ## [33] tools_4.3.1       pkgconfig_2.0.3   htmltools_0.5.6
