# rrtmodavg
*Marco Girardello* (marco.girardello@gmail.com) 

An R package for model selection and multi-model inference for Randomized Response Logistic Regression 
Models. The package includes functions for model selection and model-averaging for RRlog models, computed using the RRreg package.

The package can be installed by typing:

```r
# the devtools package is needed in order to be able to install the package
# install.packages("devtools")
library(devtools)
install_github("drmarcogir/rrtmodavg")
library(rrtmodavg)
``` 

# Example usage
Below are some simple examples of how this package can be used to analyse data collected using a randomized response design. 

## Required Packages

```{r packages, message=FALSE}
library(RRreg);library(rrtmodavg)
library(gtools);library(MuMIn)
library(tidyr);library(dplyr)
library(stringi)
```

## Automated model selection
The function `modtable()` generates all possible model combinations and ranks the models using the AIC or BIC.
The function can optionally generate specific model combinations e.g. models containing all 
3-variable combinations, 4-variable combinations etc. 

```r
#' @ y = character vector specifying the name of column containing data for the response variable (binary) 
#' @ x = character vector specifying column names containing data for the predictor variables
#' @ df =dataframe name containing predictor and response variables
#' @ combos = integer specifying desired variable combinations. e.g. 3 for models containing
#' @ all 3-variable combinations. For all possible model combinations specify "all".
#' @ inp = randomization probabilities. A numerical vector specifying randomization probabilities (e.g. c(0.1,0.1). If not specificied default is c(0.1,0.1). 

# fit all model combinations to a dummy dataset
modtab<- modtable(y = "response", x = c("pred1","pred2","pred3","pred4"),df=dummy, combos ="all")

# print first six rows of results
head(modtab)
    modID (Intercept)      pred1      pred2         pred3     pred4      AIC      BIC  deltaBIC    weightBIC deltaAIC  weightAIC
1 mod_4_1   -4.333207 0.04043779 0.07148832 -6.515456e-05 0.4344719 435.9528 455.0234 14.537721 0.0004554068 3.956570 0.04097879
2 mod_3_1   -3.617275 0.28487327 0.08646057 -1.407753e-04        NA 438.4724 453.7289 13.243189 0.0008699683 6.476169 0.01162613
3 mod_3_2   -4.337472 0.04026171 0.07152153            NA 0.4345677 433.9532 449.2097  8.724007 0.0083335762 1.956987 0.11136869
4 mod_3_3   -4.512808 0.06138911         NA -2.914202e-04 0.4914335 436.7506 452.0071 11.521423 0.0020576913 4.754403 0.02749868
5 mod_3_4   -4.157229         NA 0.07205292 -3.372099e-05 0.4599640 433.9961 449.2526  8.766909 0.0081567161 1.999889 0.10900516
6 mod_2_1   -3.628225 0.28483653 0.08654198            NA        NA 436.4745 447.9169  7.431177 0.0159061784 4.478287 0.03156964

```
## Model averaging based on an information criterion
The function `modavgrrt()` performs model averaging following the methods proposed by
Burnham & Anderson (2003). First a 95% confidence set of models is selected by summing model weights from the largest to the smallest until the sum is 0.95. Model-averaged coefficients are calculated as the weighted average of the coefficients across the confidence set, where weights are Aikaike or BIC weights.


```r
#' @ intable = model selection table calculated with the function modtable
#' @ index = information criterion to be used i.e. AIC or BIC

# perform model averaging
modavgres<-modavgrrt(intable=modtab,index="BIC")

# print results
head(modavgres)

     variable   Coefficient          SE     Lower CI     Upper CI
1 (Intercept) -4.0654192753 1.365341617 -6.741439672 -1.389398879
2       pred1  0.1975238577 0.226100510 -0.245624999  0.640672715
3       pred2  0.0796533786 0.044124767 -0.006829576  0.166136333
4       pred3 -0.0001932845 0.003133355 -0.006334548  0.005947979
5       pred4  0.5153739607 0.174383022  0.173589519  0.857158403

```

## Relative importance of predictors

The function `imprrt()` calculates variable importance following the methods developed
by Burnham & Anderson (2003) and Cade (2015).First a 95% confidence set of models is selected by summing model weights from the largest to the smallest until the sum is 0.95. Variable importance is then calculated as the sum of the model weights over all models including the explanatory variable (Burnham & Anderson 2003) or as the average of the ratio of absolute values of the t statistic  over all models including the explanatory variable, weighted by Akaike or BIC weights (Cade 2015).

```r
#' @ intable = model selection table calculated with the function modtable
#' @ index = information criterion to be used i.e. AIC or BIC
#' @ method = "burand" to use Burnham & Anderson's method or "cade" to use Cade's method

# calculate relative importance of predictors
impres<-imprrt(intable=modtab,index="BIC",method="cade")

# print results
head(impres)
    var        imp
1   pred4 1.00000000
2   pred3 0.03710603
3   pred2 0.58083567
4   pred1 0.47866592

```


## Compute model-averaged fitted values
The function `predrrt()` calculates model-averaged fitted values. First a 95% confidence set of models is selected by summing model weights from the largest to the smallest until the sum is 0.95. Model-averaged fitted values are calculated as the weighted average of the fitted values of all the models of the confidence set, where weights are Aikaike or BIC weights. 

```r
#' @ intable = model selection table calculated with the function modtable
#' @ index = information criterion to be used i.e. AIC or BIC

# calculate model-averaged fitted values
fitres<-predrrt(intable=modtab,index="BIC")

# print the first six values
head(fitres)
[1] 0.3674067 0.2862663 0.2492794 0.1649464 0.2559161 0.2531186

```

References

Burnham, K. P., & Anderson, D. R. (2003). Model selection and multimodel inference: a practical information-theoretic approach. Springer Science & Business Media.

Cade, B. S. (2015). Model averaging and muddled multimodel inferences. Ecology, 96(9), 2370-2382.

