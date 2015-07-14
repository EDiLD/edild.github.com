---
layout: post
title: "Quantitative Ecotoxicology, Page 159, Example 4.6, Control Mortality"
date: 2013-03-11 23:23
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR R
---


 
This is example 4.6 on page 159 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647). It is about how to deal with control mortalities.
 
 
First we need the data:

{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p160.csv",
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
NAP <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}

{% highlight r %}
head(NAP)
{% endhighlight %}



{% highlight text %}
##   CONC DEAD TOTAL
## 1    0    1    26
## 2    0    0    26
## 3    0    0    26
## 4    0    0    26
## 5    0    1    26
## 6    0    0    26
{% endhighlight %}
 
The data consists of number of dead animals (DEAD) from all animals (TOTAL) exposed to different concentrations (CONC).
First we create a new column with the proportion of dead animals:
 

{% highlight r %}
NAP$PROP <- NAP$DEAD / NAP$TOTAL
{% endhighlight %}
 
Here is a plot of the data. Note the use of `expression()` (greek letters in the axis labels).

{% highlight r %}
plot(NAP$CONC, NAP$PROP, 
     pch = 16, 
     xlab = expression(paste('Concentration (', mu, 'g/L)')),
     ylab = 'Proportion Dead',
     main = 'Raw data')
{% endhighlight %}

![plot of chunk plot_raw](/figures/plot_raw-1.png) 
 
 
### Control mortality
 
We can estimate the mean control mortality and the confidence interval for the mean using the `t.test` function:

{% highlight r %}
contr_m <- t.test(NAP$PROP[NAP$CONC==0])
contr_m
{% endhighlight %}



{% highlight text %}
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
{% endhighlight %}
 
These can be also easily extracted from the t.test object:
 

{% highlight r %}
## extract the values from t.test-object
# mean
contr_m$estimate
{% endhighlight %}



{% highlight text %}
## mean of x 
##  0.012821
{% endhighlight %}



{% highlight r %}
# CI
contr_m$conf.int
{% endhighlight %}



{% highlight text %}
## [1] -0.0080228  0.0336638
## attr(,"conf.level")
## [1] 0.95
{% endhighlight %}
 
This gives nearly the same values as in the book. I don't know what the SAS option `OPTC` is doing or computing, however it seems like it is the mean +- CI for the control group.
 
 
### Abbott’s formula
 
We could adjust for control mortality using Abbott's formula:
 
$$p_c = \frac{p-p_0}{1-p_0}$$
 
with $p_c$ = the corrected, p = original and $p_0$ = control mortality.
 
The mean control mortality can be calculated as:

{% highlight r %}
d_control <- mean(NAP$PROP[NAP$CONC == 0])
d_control
{% endhighlight %}



{% highlight text %}
## [1] 0.012821
{% endhighlight %}
 
And the corrected mortalities using Abbotts formula as:

{% highlight r %}
NAP$PROP_c <- (NAP$PROP - d_control) / (1 - d_control)
NAP$PROP_c
{% endhighlight %}



{% highlight text %}
##  [1]  0.025974 -0.012987 -0.012987 -0.012987  0.025974 -0.012987  0.064935
##  [8]  0.181818 -0.012987  0.311169  0.259740  0.181818  0.512266  0.454545
## [15]  0.610390  0.649351  0.688312  0.766234  0.774892  0.805195  0.805195
## [22]  1.000000  1.000000  0.922078  1.000000  0.961039  1.000000  1.000000
## [29]  1.000000  1.000000
{% endhighlight %}
 
 
### Dose-Response-Models
#### Ignoring control mortality
 
As in the previous example we can fit a dose-response-model to this data using the `drc` package:

{% highlight r %}
require(drc)
mod1 <- drm(PROP ~ CONC, data = NAP, fct = LL.2())
{% endhighlight %}
 
Comparing with other model this models performs quite good. Also the lack-of-fit test indicates a reasonable model:

{% highlight r %}
mselect(mod1, fctList= list(LL.3(), LL.4(), LL.5(), W1.2(), W1.3(), W1.4()))
{% endhighlight %}



{% highlight text %}
##      logLik      IC Lack of fit   Res var
## LL.2 47.803 -89.607    0.015649 0.0025908
## LL.3     NA      NA          NA        NA
## LL.4     NA      NA          NA        NA
## LL.5     NA      NA          NA        NA
## W1.2     NA      NA          NA        NA
## W1.3     NA      NA          NA        NA
## W1.4     NA      NA          NA        NA
{% endhighlight %}
 

{% highlight r %}
plot(mod1, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
mtext('Dose-Response-Model - LL2.2', 3)
{% endhighlight %}

![plot of chunk plot_mod1](/figures/plot_mod1-1.png) 
 
 
#### Using the corrected mortalities
 
We can also fit a model to the corrected mortalities `PROP_c`.
 
Abbotts correction resulted to some negative mortalities, therefore I set the control and all negative mortalities to zero:
 

{% highlight r %}
NAP$PROP_c[NAP$PROP_c < 0 | NAP$CONC == 0] <- 0
{% endhighlight %}
 
Then we fit a dose-response model:

{% highlight r %}
mod2 <- drm(PROP_c ~ CONC, data = NAP, fct = LL.2())
{% endhighlight %}
 
However a Weibull model fits slightly better the data, so I change to a two-parameter Weibull model (using the `update` function).
 

{% highlight r %}
mselect(mod2, fctList= list(LL.3(), LL.4(), LL.5(), W1.2(), W1.3(), W1.4()))
{% endhighlight %}



{% highlight text %}
##      logLik      IC Lack of fit   Res var
## LL.2 48.451 -90.902  3.7032e-71 0.0024813
## LL.3     NA      NA          NA        NA
## LL.4     NA      NA          NA        NA
## LL.5     NA      NA          NA        NA
## W1.2     NA      NA          NA        NA
## W1.3     NA      NA          NA        NA
## W1.4     NA      NA          NA        NA
{% endhighlight %}



{% highlight r %}
mod2 <- update(mod2, fct = W1.2())
{% endhighlight %}
 

{% highlight r %}
plot(mod2, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
mtext('Corrected mortalities - W1.2', 3)
{% endhighlight %}

![plot of chunk plot_mod2](/figures/plot_mod2-1.png) 
 
 
#### A model without fixed lower limit
 
The two-parameter log-logistic model from above (`mod1`) performs quite good. However its lower limit is fixed to 0 and the upper limit to 1.
Since we have a small amount of control mortality we could check if a model with varying lower limit (will be estimated) makes sense.
 
Let's fit a three parameter log-logistic function, where the lower limit is an additional parameter:
 

{% highlight r %}
mod3 <- drm(PROP ~ CONC, data = NAP, fct = LL.3u())
plot(mod3, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
mtext('Free (estimated) lower limit - LL3.u', 3)
{% endhighlight %}

![plot of chunk plot_mod3](/figures/plot_mod3-1.png) 
 
However looking at the summary we see that the lower limit is estimated as $0.007 \pm 0.02$ and is statistically not significant.

{% highlight r %}
summary(mod3)
{% endhighlight %}



{% highlight text %}
## 
## Model fitted: Log-logistic (ED50 as parameter) with upper limit at 1 (3 parms)
## 
## Parameter estimates:
## 
##                 Estimate Std. Error    t-value p-value
## b:(Intercept)  -13.71026    1.11140  -12.33603    0.00
## c:(Intercept)    0.00686    0.01958    0.35045    0.73
## e:(Intercept) 2099.76948   13.38536  156.87056    0.00
## 
## Residual standard error:
## 
##  0.051716 (27 degrees of freedom)
{% endhighlight %}
 
Since the lower limit (=control mortality) is so low we could also stick with `mod1`.

{% highlight r %}
mselect(mod3, fctList = list(LL.2(), LL2.3u()))
{% endhighlight %}



{% highlight text %}
##        logLik      IC Lack of fit   Res var
## LL.3u  47.872 -87.744    0.014383 0.0026746
## LL.2       NA      NA          NA        NA
## LL2.3u     NA      NA          NA        NA
{% endhighlight %}
 
 
All three considered models give nearly the same $LC_{50}$ around 2100:

{% highlight r %}
ED(mod1, 50, interval='delta')
{% endhighlight %}



{% highlight text %}
## 
## Estimated effective doses
## (Delta method-based confidence interval(s))
## 
##      Estimate Std. Error  Lower Upper
## 1:50   2097.1       10.9 2074.9  2119
{% endhighlight %}



{% highlight r %}
ED(mod2, 50, interval='delta')
{% endhighlight %}



{% highlight text %}
## 
## Estimated effective doses
## (Delta method-based confidence interval(s))
## 
##      Estimate Std. Error  Lower Upper
## 1:50   2088.2       10.2 2067.2  2109
{% endhighlight %}



{% highlight r %}
ED(mod3, 50, interval='delta')
{% endhighlight %}



{% highlight text %}
## 
## Estimated effective doses
## (Delta method-based confidence interval(s))
## 
##      Estimate Std. Error  Lower Upper
## 1:50   2099.8       13.4 2072.3  2127
{% endhighlight %}
 
