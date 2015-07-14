---
layout: post
title: "Quantitative Ecotoxicology, Page 159, Example 4.6, Control Mortality"
date: 2013-03-11 23:23
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR, R
---


 
This is example 4.6 on page 159 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647). It is about how to deal with control mortalities.
 
 
First we need the data:

{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p160.csv",
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
{% endhighlight %}



{% highlight text %}
## Error in function (type, msg, asError = TRUE) : Could not resolve host: raw.github.com
{% endhighlight %}



{% highlight r %}
NAP <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}

{% highlight r %}
head(NAP)
{% endhighlight %}



{% highlight text %}
##   DEAD TOTAL CONC
## 1   16    76 10.3
## 2   22    79 10.8
## 3   40    77 11.6
## 4   69    76 13.2
## 5   78    78 15.8
## 6   77    77 20.1
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
{% endhighlight %}



{% highlight text %}
## Error in t.test.default(NAP$PROP[NAP$CONC == 0]): not enough 'x' observations
{% endhighlight %}



{% highlight r %}
contr_m
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'contr_m' not found
{% endhighlight %}
 
These can be also easily extracted from the t.test object:
 

{% highlight r %}
## extract the values from t.test-object
# mean
contr_m$estimate
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'contr_m' not found
{% endhighlight %}



{% highlight r %}
# CI
contr_m$conf.int
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'contr_m' not found
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
## [1] NaN
{% endhighlight %}
 
And the corrected mortalities using Abbotts formula as:

{% highlight r %}
NAP$PROP_c <- (NAP$PROP - d_control) / (1 - d_control)
NAP$PROP_c
{% endhighlight %}



{% highlight text %}
## [1] NaN NaN NaN NaN NaN NaN
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
##      logLik      IC Lack of fit    Res var
## LL.2 14.613 -23.226          NA 0.00067322
## LL.3     NA      NA          NA         NA
## LL.4     NA      NA          NA         NA
## LL.5     NA      NA          NA         NA
## W1.2     NA      NA          NA         NA
## W1.3     NA      NA          NA         NA
## W1.4     NA      NA          NA         NA
{% endhighlight %}
 

{% highlight r %}
plot(mod1, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
{% endhighlight %}



{% highlight text %}
## Error in plot.drc(mod1, broken = TRUE, type = "all", bp = 500, xt = seq(500, : Argument 'conLevel' is set too high
{% endhighlight %}



{% highlight r %}
mtext('Dose-Response-Model - LL2.2', 3)
{% endhighlight %}



{% highlight text %}
## Error in mtext("Dose-Response-Model - LL2.2", 3): plot.new has not been called yet
{% endhighlight %}
 
 
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



{% highlight text %}
## Error in na.fail.default(structure(list(PROP_c = c(NaN, NaN, NaN, NaN, : missing values in object
{% endhighlight %}
 
However a Weibull model fits slightly better the data, so I change to a two-parameter Weibull model (using the `update` function).
 

{% highlight r %}
mselect(mod2, fctList= list(LL.3(), LL.4(), LL.5(), W1.2(), W1.3(), W1.4()))
{% endhighlight %}



{% highlight text %}
## Error in identical(object$type, "continuous"): object 'mod2' not found
{% endhighlight %}



{% highlight r %}
mod2 <- update(mod2, fct = W1.2())
{% endhighlight %}



{% highlight text %}
## Error in update(mod2, fct = W1.2()): error in evaluating the argument 'object' in selecting a method for function 'update': Error: object 'mod2' not found
{% endhighlight %}
 

{% highlight r %}
plot(mod2, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
{% endhighlight %}



{% highlight text %}
## Error in plot(mod2, broken = TRUE, type = "all", bp = 500, xt = seq(500, : error in evaluating the argument 'x' in selecting a method for function 'plot': Error: object 'mod2' not found
{% endhighlight %}



{% highlight r %}
mtext('Corrected mortalities - W1.2', 3)
{% endhighlight %}



{% highlight text %}
## Error in mtext("Corrected mortalities - W1.2", 3): plot.new has not been called yet
{% endhighlight %}
 
 
#### A model without fixed lower limit
 
The two-parameter log-logistic model from above (`mod1`) performs quite good. However its lower limit is fixed to 0 and the upper limit to 1.
Since we have a small amount of control mortality we could check if a model with varying lower limit (will be estimated) makes sense.
 
Let's fit a three parameter log-logistic function, where the lower limit is an additional parameter:
 

{% highlight r %}
mod3 <- drm(PROP ~ CONC, data = NAP, fct = LL.3u())
plot(mod3, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
{% endhighlight %}



{% highlight text %}
## Error in plot.drc(mod3, broken = TRUE, type = "all", bp = 500, xt = seq(500, : Argument 'conLevel' is set too high
{% endhighlight %}



{% highlight r %}
mtext('Free (estimated) lower limit - LL3.u', 3)
{% endhighlight %}



{% highlight text %}
## Error in mtext("Free (estimated) lower limit - LL3.u", 3): plot.new has not been called yet
{% endhighlight %}
 
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
##               Estimate Std. Error  t-value p-value
## b:(Intercept) -18.6783     0.6366 -29.3421       0
## c:(Intercept)   0.1394     0.0114  12.2045       0
## e:(Intercept)  11.7583     0.0266 442.0403       0
## 
## Residual standard error:
## 
##  0.00587 (3 degrees of freedom)
{% endhighlight %}
 
Since the lower limit (=control mortality) is so low we could also stick with `mod1`.

{% highlight r %}
mselect(mod3, fctList = list(LL.2(), LL2.3u()))
{% endhighlight %}



{% highlight text %}
##        logLik      IC Lack of fit     Res var
## LL.3u  24.393 -40.786          NA 0.000034457
## LL.2       NA      NA          NA          NA
## LL2.3u     NA      NA          NA          NA
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
##      Estimate Std. Error   Lower Upper
## 1:50  11.4768     0.0575 11.3171  11.6
{% endhighlight %}



{% highlight r %}
ED(mod2, 50, interval='delta')
{% endhighlight %}



{% highlight text %}
## Error in ED(mod2, 50, interval = "delta"): object 'mod2' not found
{% endhighlight %}



{% highlight r %}
ED(mod3, 50, interval='delta')
{% endhighlight %}



{% highlight text %}
## 
## Estimated effective doses
## (Delta method-based confidence interval(s))
## 
##      Estimate Std. Error   Lower Upper
## 1:50  11.7583     0.0266 11.6736  11.8
{% endhighlight %}
 
