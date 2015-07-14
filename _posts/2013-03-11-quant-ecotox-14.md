---
layout: post
title: "Quantitative Ecotoxicology, Page 159, Example 4.6, Control Mortality"
date: 2013-03-11 23:23
author: Eduard Szöcs
published: true
status: process
draft: false
tags: QETXR R
---



This is example 4.6 on page 159 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647). It is about how to deal with control mortalities. 


First we need the data:

{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p160.csv",
ssl.verifypeer = FALSE)
NAP <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}



{% highlight text %}
## Error in read.table(text = url, header = TRUE, sep = ";"): no lines available in input
{% endhighlight %}

{% highlight r %}
head(NAP)
{% endhighlight %}



{% highlight text %}
## Error in head(NAP): object 'NAP' not found
{% endhighlight %}

The data consists of number of dead animals (DEAD) from all animals (TOTAL) exposed to different concentrations (CONC).
First we create a new column with the proportion of dead animals:


{% highlight r %}
NAP$PROP <- NAP$DEAD / NAP$TOTAL
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'NAP' not found
{% endhighlight %}

Here is a plot of the data. Note the use of `expression()` (greek letters in the axis labels).

{% highlight r %}
plot(NAP$CONC, NAP$PROP, 
     pch = 16, 
     xlab = expression(paste('Concentration (', mu, 'g/L)')),
     ylab = 'Proportion Dead',
     main = 'Raw data')
{% endhighlight %}



{% highlight text %}
## Error in plot(NAP$CONC, NAP$PROP, pch = 16, xlab = expression(paste("Concentration (", : object 'NAP' not found
{% endhighlight %}


### Control mortality

We can estimate the mean control mortality and the confidence interval for the mean using the `t.test` function:

{% highlight r %}
contr_m <- t.test(NAP$PROP[NAP$CONC==0])
{% endhighlight %}



{% highlight text %}
## Error in t.test(NAP$PROP[NAP$CONC == 0]): object 'NAP' not found
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
{% endhighlight %}



{% highlight text %}
## Error in mean(NAP$PROP[NAP$CONC == 0]): object 'NAP' not found
{% endhighlight %}



{% highlight r %}
d_control
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd_control' not found
{% endhighlight %}

And the corrected mortalities using Abbotts formula as:

{% highlight r %}
NAP$PROP_c <- (NAP$PROP - d_control) / (1 - d_control)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'NAP' not found
{% endhighlight %}



{% highlight r %}
NAP$PROP_c
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'NAP' not found
{% endhighlight %}


### Dose-Response-Models
#### Ignoring control mortality

As in the previous example we can fit a dose-response-model to this data using the `drc` package:

{% highlight r %}
require(drc)
mod1 <- drm(PROP ~ CONC, data = NAP, fct = LL.2())
{% endhighlight %}



{% highlight text %}
## Error in is.data.frame(data): object 'NAP' not found
{% endhighlight %}

Comparing with other model this models performs quite good. Also the lack-of-fit test indicates a reasonable model:

{% highlight r %}
mselect(mod1, fctList= list(LL.3(), LL.4(), LL.5(), W1.2(), W1.3(), W1.4()))
{% endhighlight %}



{% highlight text %}
## Error in identical(object$type, "continuous"): object 'mod1' not found
{% endhighlight %}


{% highlight r %}
plot(mod1, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
{% endhighlight %}



{% highlight text %}
## Error in plot(mod1, broken = TRUE, type = "all", bp = 500, xt = seq(500, : object 'mod1' not found
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



{% highlight text %}
## Error in NAP$PROP_c[NAP$PROP_c < 0 | NAP$CONC == 0] <- 0: object 'NAP' not found
{% endhighlight %}

Then we fit a dose-response model:

{% highlight r %}
mod2 <- drm(PROP_c ~ CONC, data = NAP, fct = LL.2())
{% endhighlight %}



{% highlight text %}
## Error in is.data.frame(data): object 'NAP' not found
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
## Error in update(mod2, fct = W1.2()): object 'mod2' not found
{% endhighlight %}


{% highlight r %}
plot(mod2, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
{% endhighlight %}



{% highlight text %}
## Error in plot(mod2, broken = TRUE, type = "all", bp = 500, xt = seq(500, : object 'mod2' not found
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
{% endhighlight %}



{% highlight text %}
## Error in is.data.frame(data): object 'NAP' not found
{% endhighlight %}



{% highlight r %}
plot(mod3, broken = TRUE, type = 'all', bp = 500, xt = seq(500,3000,500))
{% endhighlight %}



{% highlight text %}
## Error in plot(mod3, broken = TRUE, type = "all", bp = 500, xt = seq(500, : object 'mod3' not found
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
## Error in summary(mod3): object 'mod3' not found
{% endhighlight %}

Since the lower limit (=control mortality) is so low we could also stick with `mod1`.

{% highlight r %}
mselect(mod3, fctList = list(LL.2(), LL2.3u()))
{% endhighlight %}



{% highlight text %}
## Error in identical(object$type, "continuous"): object 'mod3' not found
{% endhighlight %}


All three considered models give nearly the same $LC_{50}$ around 2100:

{% highlight r %}
ED(mod1, 50, interval='delta')
{% endhighlight %}



{% highlight text %}
## Error in ED(mod1, 50, interval = "delta"): object 'mod1' not found
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
## Error in ED(mod3, 50, interval = "delta"): object 'mod3' not found
{% endhighlight %}

