---
layout: post
title: "Quantitative Ecotoxicology, Page 159, Example 4.6, Control Mortality"
date: 2013-03-11 23:23
author: Eduard Szöcs
published: true
status: published
draft: false
tags: QETXR R
---


 
This is example 4.6 on page 159 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647). It is about how to deal with control mortalities.
 
 
First we need the data:

```r
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p160.csv",
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
NAP <- read.table(text = url, header = TRUE, sep = ";")
```

```r
head(NAP)
```

```
##   CONC DEAD TOTAL
## 1    0    1    26
## 2    0    0    26
## 3    0    0    26
## 4    0    0    26
## 5    0    1    26
## 6    0    0    26
```
 
The data consists of number of dead animals (DEAD) from all animals (TOTAL) exposed to different concentrations (CONC).
First we create a new column with the proportion of dead animals:
 

```r
NAP$PROP <- NAP$DEAD / NAP$TOTAL
```
 
Here is a plot of the data. Note the use of `expression()` (greek letters in the axis labels).

```r
plot(NAP$CONC, NAP$PROP, 
     pch = 16, 
     xlab = expression(paste('Concentration (', mu, 'g/L)')),
     ylab = 'Proportion Dead',
     main = 'Raw data')
```

![plot of chunk plot_raw](/figures/plot_raw-1.png)
 
 
### Control mortality
 
We can estimate the mean control mortality and the confidence interval for the mean using the `t.test` function:

```r
contr_m <- t.test(NAP$PROP[NAP$CONC == 0])
contr_m
```

```
## 
## 	One Sample t-test
## 
## data:  NAP$PROP[NAP$CONC == 0]
## t = 1.58, df = 5, p-value = 0.17
## alternative hypothesis: true mean is not equal to 0
## 95 percent confidence interval:
##  -0.0080228  0.0336638
## sample estimates:
## mean of x 
##  0.012821
```
 
These can be also easily extracted from the t.test object:
 

```r
## extract the values from t.test-object
# mean
contr_m$estimate
```

```
## mean of x 
##  0.012821
```

```r
# CI
contr_m$conf.int
```

```
## [1] -0.0080228  0.0336638
## attr(,"conf.level")
## [1] 0.95
```
 
This gives nearly the same values as in the book. I don't know what the SAS option `OPTC` is doing or computing, however it seems like it is the mean +- CI for the control group.
 
 
### Abbott’s formula
 
We could adjust for control mortality using Abbott's formula:
 
$$p_c = \frac{p-p_0}{1-p_0}$$
 
with $p_c$ = the corrected, p = original and $p_0$ = control mortality.
 
The mean control mortality can be calculated as:

```r
d_control <- mean(NAP$PROP[NAP$CONC == 0])
d_control
```

```
## [1] 0.012821
```
 
And the corrected mortalities using Abbotts formula as:

```r
NAP$PROP_c <- (NAP$PROP - d_control) / (1 - d_control)
NAP$PROP_c
```

```
##  [1]  0.025974 -0.012987 -0.012987 -0.012987  0.025974 -0.012987  0.064935
##  [8]  0.181818 -0.012987  0.311169  0.259740  0.181818  0.512266  0.454545
## [15]  0.610390  0.649351  0.688312  0.766234  0.774892  0.805195  0.805195
## [22]  1.000000  1.000000  0.922078  1.000000  0.961039  1.000000  1.000000
## [29]  1.000000  1.000000
```
 
 
### Dose-Response-Models
 
#### Ignoring control mortality
 
As in the previous example we can fit a dose-response-model to this data using the `drc` package:

```r
require(drc)
mod1 <- drm(DEAD/TOTAL ~ CONC, weight = TOTAL, data = NAP, fct = LL.2(), type = 'binomial')
```
 
Comparing with other model this models performs quite good:
 

```r
mselect(mod1, fctList = list(LL.3(), LL.4(), LL.5(), W1.2(), W1.3()))
```

```
##       logLik     IC Lack of fit
## LL.2 -87.435 178.87     0.48919
## LL.3      NA     NA          NA
## LL.4      NA     NA          NA
## LL.5      NA     NA          NA
## W1.2      NA     NA          NA
## W1.3      NA     NA          NA
```
 

```r
plot(mod1, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
mtext('Dose-Response-Model - LL2.2', 3)
```

![plot of chunk plot_mod1](/figures/plot_mod1-1.png)
 
 
#### Using the corrected mortalities
 
We can also fit a model to the corrected mortalities `PROP_c`.
 
Abbotts correction resulted to some negative mortalities, therefore I set the control and all negative mortalities to zero:
 

```r
NAP$PROP_c[NAP$PROP_c < 0 | NAP$CONC == 0] <- 0
```
 
Then we fit a dose-response model:

```r
mod2 <- drm(PROP_c ~ CONC, weights = TOTAL, data = NAP, fct = LL.2(), type = 'binomial')
```
 
 

```r
plot(mod2, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
mtext('Corrected mortalities - LL.2', 3)
```

![plot of chunk plot_mod2](/figures/plot_mod2-1.png)
 
 
#### A model without fixed lower limit
 
The two-parameter log-logistic model from above (`mod1`) performs quite good. However its lower limit is fixed to 0 and the upper limit to 1.
Since we have a small amount of control mortality we could check if a model with varying lower limit (will be estimated) makes sense.
 
Let's fit a three parameter log-logistic function, where the lower limit is an additional parameter:
 

```r
mod3 <- drm(PROP ~ CONC, weights = TOTAL, data = NAP, fct = LL.3u(), type = 'binomial')
plot(mod3, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
mtext('Free (estimated) lower limit - LL3.u', 3)
```

![plot of chunk plot_mod3](/figures/plot_mod3-1.png)
 
However looking at the summary we see that the lower limit (`c`) is basically zero.

```r
summary(mod3)
```

```
## 
## Model fitted: Log-logistic (ED50 as parameter) with upper limit at 1 (3 parms)
## 
## Parameter estimates:
## 
##                 Estimate Std. Error    t-value p-value
## b:(Intercept)  -13.12559    1.56664   -8.37819    0.00
## c:(Intercept)   -0.00105    0.06066   -0.01728    0.99
## e:(Intercept) 2101.42620   34.69350   60.57118    0.00
```
 
Since the lower limit (=control mortality) is so low we could also stick with the simpler model:
 

```r
anova(mod1, mod3)
```

```
## 
## 1st model
##  fct:      LL.2()
## 2nd model
##  fct:      LL.3u()
```

```
## ANOVA-like table
## 
##           ModelDf Loglik Df LR value p value
## 1st model       2  -87.4                    
## 2nd model       3  -87.4  1   0.0106    0.92
```
 
suggest that there is no statistically significant better fit of the more complicated model.
 
 
 
 
All three considered models give nearly the same $LC_{50}$ around 2100:

```r
ED(mod1, 50, interval='delta')
```

```
## 
## Estimated effective doses
## (Delta method-based confidence interval(s))
## 
##      Estimate Std. Error  Lower Upper
## 1:50   2102.0       18.8 2065.1  2139
```

```r
ED(mod2, 50, interval='delta')
```

```
## 
## Estimated effective doses
## (Delta method-based confidence interval(s))
## 
##      Estimate Std. Error  Lower Upper
## 1:50   2107.5       18.6 2070.9  2144
```

```r
ED(mod3, 50, interval='delta')
```

```
## 
## Estimated effective doses
## (Delta method-based confidence interval(s))
## 
##      Estimate Std. Error  Lower Upper
## 1:50   2101.4       34.7 2033.4  2169
```
 
