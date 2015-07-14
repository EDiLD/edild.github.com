---
layout: post
title: "Quantitative Ecotoxicology, Page 147, Example 4.3, LC50"
date: 2013-02-25 18:22
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR R
---


 
This is about example 4.3 on page 147 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647). This example is about Calculations of $LC_{50}$ values.
In this post I won't reproduce the SAS-Code since I do not have any experience with SAS PROC PROBIT and I do not fully understand whats happening there.
 
Instead I will fit Dose-Response-Models using the `drc`-package to this data.
 
 
Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p147.csv) and read it into R:
 

{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p147.csv",
ssl.verifypeer = FALSE)
salt <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}



{% highlight text %}
## Error in read.table(text = url, header = TRUE, sep = ";"): no lines available in input
{% endhighlight %}

{% highlight r %}
head(salt)
{% endhighlight %}



{% highlight text %}
## Error in head(salt): object 'salt' not found
{% endhighlight %}
 
So the data consists of number of dead animals (DEAD) from all animals (TOTAL) exposed to a concentration (CONC).
First we create a new column with the proportion of dead animals:
 

{% highlight r %}
salt$prop <- salt$DEAD / salt$TOTAL
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'salt' not found
{% endhighlight %}
 
Lets have a look at the raw data (note that I use a logarithmic scale for the x-axis):

{% highlight r %}
plot(salt$CONC, salt$prop, xlab = 'Concentration', ylab = 'Proportion dead', log='x')
{% endhighlight %}



{% highlight text %}
## Error in plot(salt$CONC, salt$prop, xlab = "Concentration", ylab = "Proportion dead", : object 'salt' not found
{% endhighlight %}
 
 
I will use the drc-package of Christian Ritz and Jens Strebig to fit dose-response-curves to this data. The main function of this package is `drm`:
 
 
Here I fit a two-parameter log-logistic model to the data (see Ritz (2010) for a review of dose-response-models used in ecotoxicology):

{% highlight r %}
require(drc)
mod <- drm(prop ~ CONC, data = salt, fct =  LL.2())
{% endhighlight %}



{% highlight text %}
## Error in is.data.frame(data): object 'salt' not found
{% endhighlight %}
 
So the usage is similar to `lm()` or `nls()`, except the `fct` argument. This argument defines the model that is fitted to the data.
 
We can compare this model with other models using the AIC (the smaller the better). 
 
Here I compare the 2-parameter log-logistic model with a two-parameter Weibull and a 2-parameter Gompertz model. 
 
`drc` has for this purpose the `mselect()` function that shows some diagnostics for every model:
 
* likelihood (the higher the better; however use this only when your models are nested)
* AIC (the lower the better)
* residual variance (the lower the better)
 

{% highlight r %}
mselect(mod, fctList=list(W1.2(),G.2()))
{% endhighlight %}



{% highlight text %}
## Error in identical(object$type, "continuous"): object 'mod' not found
{% endhighlight %}
 
The LL.2-model has the lowest AIC so I will keep this. 
Lets see how the model looks like:

{% highlight r %}
# raw data
plot(prop ~ CONC, data = salt, xlim = c(9,21), ylim = c(0,1), ylab = 'Proportion dead', xlab = 'Concentration')
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'salt' not found
{% endhighlight %}



{% highlight r %}
conc_pred <- seq(9, 21, 0.1)
lines(conc_pred, predict(mod, newdata = data.frame(CONC = conc_pred)))
{% endhighlight %}



{% highlight text %}
## Error in predict(mod, newdata = data.frame(CONC = conc_pred)): object 'mod' not found
{% endhighlight %}
 
We can get the $LC_{50}$ with confidence interval from the model using the `ED()` function:

{% highlight r %}
ED(mod, 50, interval='delta')
{% endhighlight %}



{% highlight text %}
## Error in ED(mod, 50, interval = "delta"): object 'mod' not found
{% endhighlight %}
 
Code and data is available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p147'.
 
#### References
 

{% highlight text %}
## Error in mget(nom, envir = .BibOptions): invalid first argument
{% endhighlight %}
 
