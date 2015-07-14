---
layout: post
title:  "Quantitative Ecotoxicology, Page 189, Example 4.12, QICAR-Model"
date: 2013-06-19 23:03
author: Eduard Szöcs
published: true
status: process
draft: false
tags: QETXR R
---

{% highlight text %}
## Error in BibOptions(check.entries = check.entries, style = style, hyperlink = hyperlink, : Invalid name specified, see ?BibOptions
{% endhighlight %}


This is example 4.12 on page 189 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - reproduced with R. 

### Introduction
Quantitative Ion Character-Activity Relationships (QICAR) are models that are used to predict toxicity from metal ion characteristics. 

One such metal ion characteristic is the *'softness'*: 

metal ions can classified into 
* hard (e.g., Be2, Al3, Fe3)
* soft (e.g., Cu, Ag, Hg, Pt2)
* borderline (e.g., Fe2, Co2 , Ni2, Zn2,Cu2) metal ions.

Hard acids preferentially bind to O or N, soft acids to S, and the borderline ions form stable complexes with S, O, or N (Ownby and Newman, 2003). 

In this example the softness of 20 metal ions is given, as well as associated toxicity data (EC50 values from a bacterial assay). 

We want to relate softness to toxicity by a linear model.


### Analysis

Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p189.csv) and read it into R:


{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p189.csv",
ssl.verifypeer = FALSE)
QICAR <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}



{% highlight text %}
## Error in read.table(text = url, header = TRUE, sep = ";"): no lines available in input
{% endhighlight %}

As always our first step is to look at the data:
 
Data consists of a three column table: TOTLEC is the log10 of EC50 values and SOFTCON is a measure of metal ion softness.
Plotting the data reveals that there is a strong relationship between both.


{% highlight r %}
head(QICAR)
{% endhighlight %}



{% highlight text %}
## Error in head(QICAR): object 'QICAR' not found
{% endhighlight %}



{% highlight r %}
plot(TOTLEC ~ SOFTCON, data = QICAR)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'QICAR' not found
{% endhighlight %}

To build a linear model we use the `lm()` function.
We specify the model via the formula notation `repose ~ predictor` and store it as an object names `mod`.

{% highlight r %}
mod <- lm(TOTLEC ~ SOFTCON, data = QICAR)
{% endhighlight %}



{% highlight text %}
## Error in is.data.frame(data): object 'QICAR' not found
{% endhighlight %}

Since we are interested in the model properties we take a look at the model summary:

{% highlight r %}
(mod_sum <- summary(mod))
{% endhighlight %}



{% highlight text %}
## Error in summary(mod): object 'mod' not found
{% endhighlight %}
This output shows us the intercept (2.617) and slope (SOFTCON, - 2.931) of our model, the R-Square (0.866) as well as some other usefull information.

To make a quick plot of our data and model we can use `abline`:

{% highlight r %}
plot(TOTLEC ~ SOFTCON, data = QICAR)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'QICAR' not found
{% endhighlight %}



{% highlight r %}
abline(mod)
{% endhighlight %}



{% highlight text %}
## Error in abline(mod): object 'mod' not found
{% endhighlight %}


### Polishing the plot

The above plot isn't very nice, so let's try to reproduce the plot from Figure 4.15.

To get greek symbols, sub- and superscripts etc into R plots we have to use some special mathematical annotation (see `?plotmath` for more information).



{% highlight r %}
plot(TOTLEC ~ SOFTCON, data = QICAR, 
     pch = 16, 
     xlab = expression(sigma[con]),
     ylab = expression(paste(Log[10], ' EC50 (', mu, 'M/L)')),
     cex = 1.4
     )
abline(mod, lty = 'dashed')
{% endhighlight %}
As above we plot the data and our model. The model is display as a dashed line (`lty = 'dashed'`), the raw data as solid (`pch = 16`) and bigger (`cex = 1.4`) points. 
Moreoever the axis labels were customized. All `plotmath` annotations have to be wrapped into expressions, `sigma[con]` is equal to the greek letter sigma subscript with con.

We can also add the model equation to this plot via `text()` which is slightly trickier:


{% highlight r %}
coefs <- round(coef(mod), 2)
text(0, 5, labels = bquote('log EC50' == .(coefs[2])~sigma[con] + .(coefs[1])), 
     pos = 4, cex = 0.9)
text(0, 4.5, labels = bquote(r^2 == .(round(mod_sum$r.squared, 2))), 
     pos = 4, cex = 0.9)
{% endhighlight %}
First we extract the model coefficients via `coef()`. 
If we want to access these numbers (and not type them manually) in the equation, we have to embed the equation into `bqoute()`. `bqoute()` works like `expression()` above, except that  objects wrapped in `.()` will be replaced by their respective values.



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'QICAR' not found
{% endhighlight %}



{% highlight text %}
## Error in abline(mod, lty = "dashed"): object 'mod' not found
{% endhighlight %}



{% highlight text %}
## Error in coef(mod): object 'mod' not found
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'coefs' not found
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'mod_sum' not found
{% endhighlight %}

Once again we reproduced the same results as in the book using R :)
Code and data are available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p189'.

### References


{% highlight text %}
## Error in mget(nom, envir = .BibOptions): invalid first argument
{% endhighlight %}
