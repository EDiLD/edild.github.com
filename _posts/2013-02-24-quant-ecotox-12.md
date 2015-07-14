---
layout: post
title: "Quantitative Ecotoxicology, Page 109, Example 3.8, Bioaccumulation"
date: 2013-02-24 23:14
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR R
---


 
This is example 3.8 on page 109 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - reproduced with R. This example is about accumulation and elimination of bromophos from water in a guppy (*Poecilia reticulata*).
 
There are two data files for this example - one for the [accumulation](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p109_accum.csv) and on for the [elimination](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p109_elimin.csv).
 
 
### Accumulation
First we will look at the accumulation phase:

{% highlight r %}
require(RCurl)
# Accumulation
url_accum <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p109_accum.csv",
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
ACCUM <- read.table(text = url_accum, header = TRUE, sep = ";")
{% endhighlight %}

{% highlight r %}
head(ACCUM)
{% endhighlight %}



{% highlight text %}
##   HOUR BRPHOS
## 1  0.5   1900
## 2  1.0   3000
## 3  2.0   5200
## 4  4.0   6900
## 5  8.0  24000
## 6 24.0  50000
{% endhighlight %}
 
Again we have two columns: One for the time and one for the concentration.
 
 
We fit can same model as in [example 3.7](http://edild.github.com/blog/2013/02/24/quant-ecotox-11/) to this data. The uptake $(k_u)$ and elimination $(k_e)$ constants are estimated simultaneously (at the same time):
 
 

{% highlight r %}
mod_accum <- nls(BRPHOS ~ KU / KE * 10.5 * (1 - exp(-KE * HOUR)),
           data = ACCUM, 
           start = list(KU = 100, KE = 0.01))
{% endhighlight %}
Note that I used different starting values than in the SAS-Code (must be a typo in the book). Also I didn't specify any bounds.

{% highlight r %}
summary(mod_accum)
{% endhighlight %}



{% highlight text %}
## 
## Formula: BRPHOS ~ KU/KE * 10.5 * (1 - exp(-KE * HOUR))
## 
## Parameters:
##     Estimate Std. Error t value Pr(>|t|)    
## KU 344.79786   31.85529   10.82  4.7e-06 ***
## KE   0.00525    0.00103    5.09  0.00094 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18900 on 8 degrees of freedom
## 
## Number of iterations to convergence: 7 
## Achieved convergence tolerance: 3.81e-06
{% endhighlight %}
 

{% highlight r %}
HOUR_pred <- seq(min(ACCUM$HOUR), max(ACCUM$HOUR), by = 0.1) 
# Raw data
plot(ACCUM, main = 'Accumulation')
# add model
lines(HOUR_pred, predict(mod_accum, newdata = data.frame(HOUR = HOUR_pred)))
{% endhighlight %}

<img src="/figures/plot_accum_model-1.png" title="plot of chunk plot_accum_model" alt="plot of chunk plot_accum_model" width="400px" />
 
So from the accumulation data we estimated the uptake and elimination constants as:
 
* $k_e = 0.0053 \pm 0.0010$
* $k_u = 344.798 \pm 31.855$
 
 
 
 
### Sequential estimation
However we could also estimate the elimination constant $(k_e)$ from the elimination phase and then use this estimate for our accumulation data. 
 
* First estimate $k_e$ from a linear model (linear transformation)
* Plug this estimated $k_e$ into a nonlinear model to estimate $k_u$
 
 

{% highlight r %}
# Elimination data
url_elimin <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p109_elimin.csv",
ssl.verifypeer = FALSE)
ELIMIN <- read.table(text = url_elimin, header = TRUE, sep = ";")
{% endhighlight %}



{% highlight text %}
## Error in read.table(text = url_elimin, header = TRUE, sep = ";"): no lines available in input
{% endhighlight %}

{% highlight r %}
head(ELIMIN)
{% endhighlight %}



{% highlight text %}
## Error in head(ELIMIN): object 'ELIMIN' not found
{% endhighlight %}



{% highlight r %}
plot(ELIMIN)
{% endhighlight %}



{% highlight text %}
## Error in plot(ELIMIN): error in evaluating the argument 'x' in selecting a method for function 'plot': Error: object 'ELIMIN' not found
{% endhighlight %}
 
 
We will estimate $k_e$ from a linear model like in [previous examples](http://edild.github.com/blog/2013/02/24/quant-ecotox-10/). We could also use nls for this.
 
First we need to transform the bromophos-concentration to linearize the relationship.

{% highlight r %}
ELIMIN$LBROMO <- log(ELIMIN$BRPHOS)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'ELIMIN' not found
{% endhighlight %}
 
The we can use lm() to fit the linear model:

{% highlight r %}
mod_elimin_lm <- lm(LBROMO ~ HOUR, data = ELIMIN)
{% endhighlight %}



{% highlight text %}
## Error in is.data.frame(data): object 'ELIMIN' not found
{% endhighlight %}



{% highlight r %}
summary(mod_elimin_lm)
{% endhighlight %}



{% highlight text %}
## Error in summary(mod_elimin_lm): error in evaluating the argument 'object' in selecting a method for function 'summary': Error: object 'mod_elimin_lm' not found
{% endhighlight %}
 
So we get an estimate of $k_e$ as $0.0147 \pm 0.0003$.
 
This is quite different to the $k_e$ estimated simultaneous from the accumulation data!
Our linear model fits very good (R^2 = 0.998, no pattern in the residuals), so something is strange here...

{% highlight r %}
par(mfrow = c(1, 2))
# plot linearized model
plot(LBROMO ~ HOUR, data = ELIMIN, main = 'Data + Model')
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'ELIMIN' not found
{% endhighlight %}



{% highlight r %}
# add regression line
abline(mod_elimin_lm)
{% endhighlight %}



{% highlight text %}
## Error in abline(mod_elimin_lm): object 'mod_elimin_lm' not found
{% endhighlight %}



{% highlight r %}
# plot residuals
plot(fitted(mod_elimin_lm), residuals(mod_elimin_lm), main = 'Residuals')
{% endhighlight %}



{% highlight text %}
## Error in plot(fitted(mod_elimin_lm), residuals(mod_elimin_lm), main = "Residuals"): error in evaluating the argument 'x' in selecting a method for function 'plot': Error in fitted(mod_elimin_lm) : object 'mod_elimin_lm' not found
{% endhighlight %}



{% highlight r %}
abline(h = 0, lty = 'dotted')
{% endhighlight %}



{% highlight text %}
## Error in int_abline(a = a, b = b, h = h, v = v, untf = untf, ...): plot.new has not been called yet
{% endhighlight %}
 
 
### Plug $k_e$ from the elimination phase into the accumulation model
 
Lets take $k_e$ from the elimination phase and plug it into our accumulation model and investigate the differences:
 

{% highlight r %}
mod_accum2 <- nls(BRPHOS ~ KU / -coef(mod_elimin_lm)[2] * 10.5 * (1 - exp(coef(mod_elimin_lm)[2] * HOUR)),
           data = ACCUM, 
           start = list(KU = 100))
{% endhighlight %}



{% highlight text %}
## Error in nls(BRPHOS ~ KU/-coef(mod_elimin_lm)[2] * 10.5 * (1 - exp(coef(mod_elimin_lm)[2] * : parameters without starting value in 'data': mod_elimin_lm
{% endhighlight %}



{% highlight r %}
summary(mod_accum2)
{% endhighlight %}



{% highlight text %}
## Error in summary(mod_accum2): error in evaluating the argument 'object' in selecting a method for function 'summary': Error: object 'mod_accum2' not found
{% endhighlight %}
 
This estimates $k_u = 643.9 \pm 40.4$ which differs greatly from our initial results!
Lets plot this model and the residuals:

{% highlight r %}
par(mfrow=c(1,2))
HOUR_pred <- seq(min(ACCUM$HOUR), max(ACCUM$HOUR), by = 0.1) 
# Raw data
plot(ACCUM, main = 'Accumulation')
# add model
lines(HOUR_pred, predict(mod_accum2, newdata = data.frame(HOUR = HOUR_pred)))
{% endhighlight %}



{% highlight text %}
## Error in predict(mod_accum2, newdata = data.frame(HOUR = HOUR_pred)): object 'mod_accum2' not found
{% endhighlight %}



{% highlight r %}
plot(fitted(mod_accum2), residuals(mod_accum2))
{% endhighlight %}



{% highlight text %}
## Error in plot(fitted(mod_accum2), residuals(mod_accum2)): error in evaluating the argument 'x' in selecting a method for function 'plot': Error in fitted(mod_accum2) : object 'mod_accum2' not found
{% endhighlight %}

<img src="/figures/plot_accum_model2-1.png" title="plot of chunk plot_accum_model2" alt="plot of chunk plot_accum_model2" width="400px" />
 
 
The residuals show a clear curve pattern. But we could also look at the residual sum of squares and the AIC to see which model fit better to the accumulation data:
 

{% highlight r %}
# Residual sum of squares
mod_accum$m$deviance()
{% endhighlight %}



{% highlight text %}
## [1] 2870355583
{% endhighlight %}



{% highlight r %}
mod_accum2$m$deviance()
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'mod_accum2' not found
{% endhighlight %}



{% highlight r %}
# AIC
AIC(mod_accum)
{% endhighlight %}



{% highlight text %}
## [1] 229.13
{% endhighlight %}



{% highlight r %}
AIC(mod_accum2)
{% endhighlight %}



{% highlight text %}
## Error in AIC(mod_accum2): object 'mod_accum2' not found
{% endhighlight %}
 
So the first model seem to better fit to the data. However see the discussion in the book for this example!
 
Once again we reproduced the results as in the book using R :)
 
Code and data are available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p109'.
 
