Statistical Modelling - Beginner Tier - Model Building
================
Carina Fan (<carina@dacture.com>)<br/>
Last updated: January 29, 2024

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

``` r
suppressPackageStartupMessages({
  library(broom)
  library(car)
  library(ggfortify) # autoplot
  # devtools::install_github("selva86/InformationValue")
  library(InformationValue) # misclassification error
  library(jsonlite)
  library(moments)
  library(nnet) # multinomial logistic regression
  library(pscl) # pseudo R2 for logistic regression
  library(reshape2)
  library(yardstick) # mosaic plots
  library(magrittr)
  library(tidyverse)
})
```

``` r
# path = "../../../../data/VehicleInsuranceData_2500.csv"
# df = path |> read.csv(na.strings = c("N/A", "NA", "Unknown"))

# set working directory 
setwd("/Users/verns/Documents/data/") 

# read in dataset
df <- read.csv("VehicleInsuranceData_2500.csv", header = TRUE, sep = ",",
              stringsAsFactors = TRUE, na.strings =  c("NA", "", " ", ".",  "..", "na", "N/A", "NULL"))
# stringsAsFactors specifies whether character variables should be converted to factor 
```

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

``` r
outcome = "Gender" 
predictor1 = "Vehicle.Class"
predictor2 = "Vehicle.Size"
covariate = FALSE
pos_class = FALSE
ref_level = FALSE
convertToFactor = NULL # set to NULL if it is empty
```

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

``` r
df_orig_effect <- df |> select_if(~ all(. %in% c(-1,1,NA)))
df_orig_dummy <- df |> select_if(~ all(. %in% c(0,1,NA)))
df_orig_no_dummy <- df |> select_if(purrr::negate(~ all(. %in% c(0,1,NA)))) |> select_if(purrr::negate(~ all(. %in% c(-1,1,NA))))

# drop any zero-variance variables from df_orig_effect
for (col in 1:length(colnames(df_orig_effect))) {
  if (length(df_orig_effect > 0)) {
    if (length(unique(na.omit(df_orig_effect[, col]))) == 1) {
      df_orig_effect[,col] <- NULL
    }
  }
}
    
# convert effect coding to dummy coding
for (col in 1:length(colnames(df_orig_effect))) {
  if (length(df_orig_effect > 0)) {
    if (length(unique(na.omit(df_orig_effect[, col]))) == 2 & is.numeric(df_orig_effect[, col])) {
      df_orig_effect[,col] <- ifelse(df_orig_effect[,col] == -1, 0, ifelse(df_orig_effect[,col] == 1, 1, df_orig_effect[,col]))
    }
  }
}

# combine effect coded df with dummy coded df, depending on which exists
if (length(df_orig_dummy) > 0 & length(df_orig_effect) > 0) {
  df_orig_dummy <- cbind(df_orig_dummy, df_orig_effect)
} else if (length(df_orig_dummy) > 0 & length(df_orig_effect) == 0) {
  df_orig_dummy <- df_orig_dummy
} else if (length(df_orig_dummy) == 0 & length(df_orig_effect) > 0) {
  df_orig_dummy <- df_orig_effect
}

# turn all non-dummy-coded binary variables to factors
for (col in 1:length(colnames(df_orig_no_dummy))) {
  if (length(df_orig_no_dummy) > 0) {
    if (length(unique(na.omit(df_orig_no_dummy[, col]))) == 2 & is.numeric(df_orig_no_dummy[, col])) {
      df_orig_no_dummy[[col]] = as.factor(df_orig_no_dummy[[col]])
    }
  }
}

# turn any numeric variables to categorical as specified by front-end df upload
for (i in (convertToFactor)) {
  if (length(df_orig_no_dummy) > 0) {
    if (i %in% df_orig_no_dummy[[i]]) {
      df_orig_no_dummy[[i]] = as.factor(df_orig_no_dummy[[i]])
    }
  }
}

# combine changes into a single df
if (length(df_orig_dummy) > 0 & length(df_orig_no_dummy) > 0) {
  df <- cbind(df_orig_dummy,df_orig_no_dummy)
} else if (length(df_orig_dummy) > 0 & length(df_orig_no_dummy) == 0) {
  df <- df_orig_dummy
} else if (length(df_orig_dummy) == 0 & length(df_orig_no_dummy) > 0) {
  df <- df_orig_no_dummy
}
```

## Subset data

``` r
# select variables for analysis
inputs = c(predictor1, predictor2, covariate)[which(c(predictor1, predictor2, covariate) != FALSE)]
variables = c(outcome, inputs)

# create new dataframe with only selected variables, then drop missing values
df.analysis = df |> 
  dplyr::select(all_of(variables)) |> 
  drop_na() |> 
  mutate_if(is.character, as.factor)

# preserve un-centered variables for visualizations later
df.analysis.uncentered = df.analysis 
```

## Check categorical variables

``` r
# if an integer variable has only 2 values, recode it as a factor
for (i in 1:length(variables)) {
  
  var.levels = df.analysis[[variables[i]]] |> 
    as.factor() |> 
    levels() |> 
    length()
  
  if (var.levels == 2) {
    df.analysis[[variables[i]]] %<>% as.factor()
  }
  
}

# if any variable has only 1 value, print an error & have user pick a different variable
single_level = vector()
for (i in 1:length(variables)) {
  
  if(length(unique(df.analysis[[variables[i]]])) == 1) {
    single_level %<>% append(variables[i])
  }
  
}
if (length(single_level) > 0) {
  print("The following variables cannot be modeled because they have only 1 unique value. Please select a different variable.")
  print(single_level)
}

# for multinomial regression, if a categorical variable has more than 10 levels, print an error & have user pick a different variable
too_many_levels = vector()

for (i in 1:length(variables)) {
  if (is.factor(df.analysis[[variables[i]]])) {
    if (length(levels(df.analysis[[variables[i]]])) >= 10) {
      too_many_levels %<>% append(variables[i])
    }
  }
}

if (length(too_many_levels) > 0) {
  print("The following variables cannot be modeled because they have more than 10 categories. Please select a different variable.")
  print(too_many_levels)
}
```

We may need a script break at this point to allow for error printing &
for the user to go back to select new variables if necessary.

## Centre continuous variables

``` r
if (is.numeric(df.analysis[[predictor1]])) {
  df.analysis %<>%
    mutate_at(vars(all_of(predictor1)), function(x) {x - mean(x, na.rm = TRUE)})
}

if (predictor2 != FALSE) {
  if (is.numeric(df.analysis[[predictor2]])) {
    df.analysis %<>%
      mutate_at(vars(all_of(predictor2)), function(x) {x - mean(x, na.rm = TRUE)})
  }
}

if (covariate != FALSE) {
  if (is.numeric(df.analysis[[covariate]])) {
    df.analysis %<>%
      mutate_at(vars(all_of(covariate)), function(x) {x - mean(x, na.rm = TRUE)})
  }
}
```

## Determine analysis to run

``` r
outcome.type = class(df.analysis[[outcome]])

if (outcome.type == "factor") {
  if (length(levels(df.analysis[[outcome]])) == 2) {
    analysis = "logistic"
  } else if (length(levels(df.analysis[[outcome]])) > 2) {
    analysis = "multinomial_logistic"
  }
} else {
  analysis = "linear"
}
```

<!-- ======================================================================= -->

# Print descriptive information about data

Number of rows for analysis:

``` r
# number of rows for analysis
paste0("After removing missing data, there are ", nrow(df.analysis), " rows in the dataset.") |> 
  print()
```

    ## [1] "After removing missing data, there are 2500 rows in the dataset."

Range/levels of variables:

``` r
# range/levels of outcome variable
if (analysis == "linear") {
  paste0("The range of the outcome variable, ", outcome, ", is ", 
         min(df.analysis.uncentered[[outcome]]) |> round(2), " to ", 
         max(df.analysis.uncentered[[outcome]]) |> round(2), 
         ", and the average value is ", 
         mean(df.analysis.uncentered[[outcome]]) |> round(2), ".") |> 
    print()
} else {
  paste0("The levels of your outcome variable, ", outcome, ", are: ",
         paste(levels(df.analysis.uncentered[[outcome]]),
               collapse = ", "),
         ".") |> 
    print()
}
```

    ## [1] "The levels of your outcome variable, Gender, are: F, M."

``` r
# range/levels of predictor 1
if (is.numeric(df.analysis.uncentered[[predictor1]])) {
    paste0("The range of your first predictor, ", predictor1, ", is ",
           min(df.analysis.uncentered[[predictor1]]) |> round(2), " to ",
           max(df.analysis.uncentered[[predictor1]]) |> round(2), 
           ", and the average value is ", 
           mean(df.analysis.uncentered[[predictor1]]) |> round(2), ".") |> 
    print()
  } else {
    paste0("The levels of your outcome variable, ", outcome, ", are: ",
           paste(levels(df.analysis.uncentered[[outcome]]),
                 collapse = ", "),
           ".") |> 
      print()
}
```

    ## [1] "The levels of your outcome variable, Gender, are: F, M."

``` r
# range/levels of predictor 2
if (predictor2 != FALSE) {
  if (is.numeric(df.analysis.uncentered[[predictor2]])) {
    paste0("The range of your second predictor, ", predictor2, ", is ",
           min(df.analysis.uncentered[[predictor2]]) |> round(2), " to ",
           max(df.analysis.uncentered[[predictor2]]) |> round(2), 
           ", and the average value is ", 
           mean(df.analysis.uncentered[[predictor2]]) |> round(2), ".") |> 
      print()
  } else {
      paste0("The levels of your second predictor, ", predictor2, ", are: ",
             paste(levels(df.analysis.uncentered[[predictor2]]),
                   collapse = ", "),
             ".") |> 
        print()
  }
}
```

    ## [1] "The levels of your second predictor, Vehicle.Size, are: Large, Medsize, Small."

<!-- ======================================================================= -->

# Build model

Set up regression formula based on selected combination of predictors
and covariate:

``` r
if (predictor2 == FALSE | analysis == "multinomial_logistic") {
  formula = paste0(outcome, " ~ ", paste(inputs, collapse = " + "))
} else {
  formula = paste0(outcome, " ~ ", paste(inputs, collapse = " + "), 
                  " + ", predictor1, ":", predictor2)
}

formula %<>% as.formula()
```

Source appropriate analysis script:

``` r
if (analysis == "linear") {
  source("/Users/verns/Documents/dacture-r/dynamic_modeling/dev/statistical-modeling-beginner/univariate/2-linear-regression.R")
} else if (analysis == "logistic") {
  source("/Users/verns/Documents/dacture-r/dynamic_modeling/dev/statistical-modeling-beginner/univariate/2-logistic-regression.R")
} else if (analysis == "multinomial_logistic") {
  source("/Users/verns/Documents/dacture-r/dynamic_modeling/dev/statistical-modeling-beginner/univariate/2-multinomial-logistic-regression.R")
}
```

    ## [1] "Vehicle.Class did not statistically significantly predict Gender."
    ## [1] "Vehicle.Size did not statistically significantly predict Gender."
    ## [1] "The interaction between Vehicle.Class and Vehicle.Size was not statistically significant in predicting Gender."
    ## [1] "The pseudo R-squared value for your model is 0.007. This value ranges from 0 to 1, with values closer to 1 indicating a better model fit."
    ## [1] Sensitivity is the "true positive" rate, or the percentage of observations the model correctly predicted for the "positive" result. In this case, the "positive" result for your outcome variable, Gender, is M. The sensitivity of the model is 36.5%.
    ## [1] Specificity is the "true negative" rate, or the percentage of observations the model correctly predicted for the "negative" result. In this case, the "negative" result for your outcome variable, Gender, is F. The specificity of the model is 69.8%.
    ## [1] "Misclassification error is the percentage of observations the model predicted the outcome incorrectly for. The total misclassification error rate for your model is 49.9%."

<!-- ======================================================================= -->

# Export R script and workspace

``` r
rm(list=ls()[! ls() %in% c(
  "outcome", "predictor1", "predictor2", "covariate","pos_class","ref_level",
  "variables",
  "inputs",
  "analysis",
  "df.analysis",
  "df.analysis.uncentered",
  "model"
)])

save.image("/Users/verns/Documents/dacture-r/dynamic_modeling/dev/statistical-modeling-beginner/univariate/1-model-building.RData")
```

``` r
options(knitr.duplicate.label = "allow")
knitr::purl("1-model-building.Rmd", documentation = 1)
```

    ## 
    ## 
    ## processing file: 1-model-building.Rmd

    ##   |                                                           |                                                   |   0%  |                                                           |..                                                 |   3%                     |                                                           |...                                                |   7% [packages-26]       |                                                           |.....                                              |  10%                     |                                                           |.......                                            |  13% [data-27]           |                                                           |........                                           |  17%                     |                                                           |..........                                         |  20% [user_inputs-28]    |                                                           |............                                       |  23%                     |                                                           |..............                                     |  27% [unnamed-chunk-29]  |                                                           |...............                                    |  30%                     |                                                           |.................                                  |  33% [unnamed-chunk-30]  |                                                           |...................                                |  37%                     |                                                           |....................                               |  40% [unnamed-chunk-31]  |                                                           |......................                             |  43%                     |                                                           |........................                           |  47% [unnamed-chunk-32]  |                                                           |..........................                         |  50%                     |                                                           |...........................                        |  53% [unnamed-chunk-33]  |                                                           |.............................                      |  57%                     |                                                           |...............................                    |  60% [unnamed-chunk-34]  |                                                           |................................                   |  63%                     |                                                           |..................................                 |  67% [unnamed-chunk-35]  |                                                           |....................................               |  70%                     |                                                           |.....................................              |  73% [unnamed-chunk-36]  |                                                           |.......................................            |  77%                     |                                                           |.........................................          |  80% [unnamed-chunk-37]  |                                                           |..........................................         |  83%                     |                                                           |............................................       |  87% [unnamed-chunk-38]  |                                                           |..............................................     |  90%                     |                                                           |................................................   |  93% [unnamed-chunk-39]  |                                                           |.................................................  |  97%                     |                                                           |...................................................| 100% [session_info-40] 

    ## output file: 1-model-building.R

    ## [1] "1-model-building.R"

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
    ##  [1] lubridate_1.9.3        forcats_1.0.0          stringr_1.5.1         
    ##  [4] dplyr_1.1.4            purrr_1.0.2            readr_2.1.5           
    ##  [7] tidyr_1.3.0            tibble_3.2.1           tidyverse_2.0.0       
    ## [10] magrittr_2.0.3         yardstick_1.2.0        reshape2_1.4.4        
    ## [13] pscl_1.5.5.1           nnet_7.3-19            moments_0.14.1        
    ## [16] jsonlite_1.8.8         InformationValue_1.3.1 ggfortify_0.4.16      
    ## [19] ggplot2_3.4.4          car_3.1-2              carData_3.0-5         
    ## [22] broom_1.0.5           
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] tidyselect_1.2.0     timeDate_4032.109    fastmap_1.1.1       
    ##  [4] pROC_1.18.5          caret_6.0-94         digest_0.6.34       
    ##  [7] rpart_4.1.23         timechange_0.2.0     lifecycle_1.0.4     
    ## [10] survival_3.5-7       compiler_4.3.1       rlang_1.1.3         
    ## [13] tools_4.3.1          utf8_1.2.4           yaml_2.3.8          
    ## [16] data.table_1.14.10   knitr_1.45           plyr_1.8.9          
    ## [19] abind_1.4-5          withr_3.0.0          stats4_4.3.1        
    ## [22] grid_4.3.1           fansi_1.0.6          colorspace_2.1-0    
    ## [25] future_1.33.1        globals_0.16.2       scales_1.3.0        
    ## [28] iterators_1.0.14     MASS_7.3-60.0.1      cli_3.6.2           
    ## [31] rmarkdown_2.25       generics_0.1.3       rstudioapi_0.15.0   
    ## [34] future.apply_1.11.1  tzdb_0.4.0           splines_4.3.1       
    ## [37] parallel_4.3.1       vctrs_0.6.5          hardhat_1.3.0       
    ## [40] Matrix_1.6-5         hms_1.1.3            listenv_0.9.0       
    ## [43] foreach_1.5.2        gower_1.0.1          recipes_1.0.9       
    ## [46] glue_1.7.0           parallelly_1.36.0    codetools_0.2-19    
    ## [49] stringi_1.8.3        gtable_0.3.4         munsell_0.5.0       
    ## [52] pillar_1.9.0         htmltools_0.5.7      ipred_0.9-14        
    ## [55] lava_1.7.3           R6_2.5.1             evaluate_0.23       
    ## [58] lattice_0.22-5       backports_1.4.1      class_7.3-22        
    ## [61] Rcpp_1.0.12          gridExtra_2.3        nlme_3.1-164        
    ## [64] prodlim_2023.08.28   xfun_0.41            pkgconfig_2.0.3     
    ## [67] ModelMetrics_1.2.2.2
