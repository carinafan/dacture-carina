# Dacture examples 
Example outputs from my work as a machine learning engineer at Dacture.

Dacture’s product was a software platform that allowed people to easily run and interpret machine learning models without any code or ML experience. 
My role was to code end-to-end flows in Python and R to automatically run and interpret the appropriate model given the user’s research question, 
and then provide an output that was easily interpretable by someone largely unfamiliar with ML models.

This repo contains example outputs from some of the end-to-end flows that I built. html files were generated using R. Source code is hidden but the output of the evaluated code is available.

## Data exploration

Includes a correlation heat map and single-variable plots (i.e., histogram or bar plot).

## Regression

Automatically detects the type of the user's target variable, and runs the appropriate regression (e.g., linear, logistic). 
Allows users to input hypothetical values of the predictor values to predict a value for the target variable.

## Uplift modelling

Given data from an A/B test, predicts the best treatment for each individual. 
This flow was coded in Python and is not complete, but example of the figures generated from this analysis are available.