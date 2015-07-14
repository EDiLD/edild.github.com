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
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
salt <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}

{% highlight r %}
head(salt)
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
 
So the data consists of number of dead animals (DEAD) from all animals (TOTAL) exposed to a concentration (CONC).
First we create a new column with the proportion of dead animals:
 

{% highlight r %}
salt$prop <- salt$DEAD / salt$TOTAL
{% endhighlight %}
 
Lets have a look at the raw data (note that I use a logarithmic scale for the x-axis):

{% highlight r %}
plot(salt$CONC, salt$prop, xlab = 'Concentration', ylab = 'Proportion dead', log='x')
{% endhighlight %}

![plot of chunk p147_plot_raw](/figures/p147_plot_raw-1.png) 
 
 
I will use the drc-package of Christian Ritz and Jens Strebig to fit dose-response-curves to this data. The main function of this package is `drm`:
 
 
Here I fit a two-parameter log-logistic model to the data (see Ritz (2010) for a review of dose-response-models used in ecotoxicology):

{% highlight r %}
require(drc)
mod <- drm(prop ~ CONC, data = salt, fct =  LL.2())
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
##      logLik      IC Lack of fit    Res var
## LL.2 14.613 -23.226          NA 0.00067322
## W1.2     NA      NA          NA         NA
## G.2      NA      NA          NA         NA
{% endhighlight %}
 
The LL.2-model has the lowest AIC so I will keep this. 
Lets see how the model looks like:

{% highlight r %}
# raw data
plot(prop ~ CONC, data = salt, xlim = c(9,21), ylim = c(0,1), ylab = 'Proportion dead', xlab = 'Concentration')
 
conc_pred <- seq(9, 21, 0.1)
lines(conc_pred, predict(mod, newdata = data.frame(CONC = conc_pred)))
{% endhighlight %}

![plot of chunk p147_plot_mod](/figures/p147_plot_mod-1.png) 
 
We can get the $LC_{50}$ with confidence interval from the model using the `ED()` function:

{% highlight r %}
ED(mod, 50, interval='delta')
{% endhighlight %}



{% highlight text %}
## 
## Estimated effective doses
## (Delta method-based confidence interval(s))
## 
##      Estimate Std. Error   Lower Upper
## 1:50  11.4768     0.0575 11.3171  11.6
{% endhighlight %}
 
Code and data is available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p147'.
 
#### References
 

{% highlight text %}
## Error in mget(nom, envir = .BibOptions): invalid first argument
{% endhighlight %}
 
