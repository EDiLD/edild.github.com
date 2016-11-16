---
layout: post
title:  "Quantitative Ecotoxicology, Page 189, Example 4.12, QICAR-Model"
date: 2013-06-19 23:03
author: Eduard Szöcs
published: true
status: published
draft: false
tags: QETXR R
---


 
This is example 4.12 on page 189 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - reproduced with R.
 
### Introduction
Quantitative Ion Character-Activity Relationships (QICAR) are models that are used to predict toxicity from metal ion characteristics. 
 
One such metal ion characteristic is the *'softness'*:
 
metal ions can be classified into 
 
* hard (e.g., Be2, Al3, Fe3)
* soft (e.g., Cu, Ag, Hg, Pt2)
* borderline (e.g., Fe2, Co2 , Ni2, Zn2,Cu2) metal ions.
 
Hard acids preferentially bind to O or N, soft acids to S, and the borderline ions form stable complexes with S, O, or N (Ownby and Newman, 2003). 
 
In this example, the softness of 20 metal ions is given, as well as associated toxicity data (EC50 values from a bacterial assay). 
 
We want to relate softness to toxicity by a linear model.
 
 
### Analysis
 
Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p189.csv) and read it into R:
 

{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p189.csv",
              .opts = curlOptions(followlocation=TRUE), ssl.verifypeer = FALSE)
QICAR <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}
 
As always our first step is to look at the data:
 
Data consists of a three column table: TOTLEC is the log10 of EC50 values and SOFTCON is a measure of metal ion softness.
Plotting the data reveals that there is a strong relationship between both.
 

{% highlight r %}
head(QICAR)
{% endhighlight %}



{% highlight text %}
##   METAL TOTLEC SOFTCON
## 1  HG2+ -0.037    1.16
## 2  CA2+  4.976   -0.99
## 3  CD2+  1.424    0.17
## 4  CU2+  0.208    0.65
## 5  MG2+  4.941   -1.02
## 6  MN2+  3.196   -0.20
{% endhighlight %}



{% highlight r %}
plot(TOTLEC ~ SOFTCON, data = QICAR)
{% endhighlight %}

![plot of chunk plot_data](/figures/plot_data-1.png)
 
To build a linear model we use the `lm()` function.
We specify the model via the formula notation `repose ~ predictor` and store it as an object names `mod`.

{% highlight r %}
mod <- lm(TOTLEC ~ SOFTCON, data = QICAR)
{% endhighlight %}
 
Since we are interested in the model properties we take a look at the model summary:

{% highlight r %}
(mod_sum <- summary(mod))
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = TOTLEC ~ SOFTCON, data = QICAR)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -1.3338 -0.5736  0.0007  0.6673  1.1423 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    2.617      0.185    14.2  3.3e-11 ***
## SOFTCON       -2.931      0.272   -10.8  2.8e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.799 on 18 degrees of freedom
## Multiple R-squared:  0.866,	Adjusted R-squared:  0.858 
## F-statistic:  116 on 1 and 18 DF,  p-value: 2.81e-09
{% endhighlight %}
This output shows us the intercept (2.617) and slope (SOFTCON, - 2.931) of our model, the R-Square (0.866) as well as some other useful information.
 
To make a quick plot of our data and model we can use `abline`:

{% highlight r %}
plot(TOTLEC ~ SOFTCON, data = QICAR)
abline(mod)
{% endhighlight %}

![plot of chunk unnamed-chunk-5](/figures/unnamed-chunk-5-1.png)
 
 
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
Moreoever, the axis labels were customized. All `plotmath` annotations have to be wrapped into expressions, `sigma[con]` is equal to the greek letter sigma subscript with con.
 
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
 
 
![plot of chunk plot_model2](/figures/plot_model2-1.png)
 
Once again we reproduced the same results as in the book using R :)
Code and data are available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p189'.
 
### References
 
[1] D. Ownby and M. Newman. "Advances in Quantitative Ion
Character-Activity Relationships (QICARs): Using Metal-Ligand
Binding Characteristics to Predict Metal Toxicity". In: _QSAR \&
Combinatorial Science_ 22.2 (Apr. 2003), pp. 241-246. DOI:
10.1002/qsar.200390018. <URL:
http://dx.doi.org/10.1002/qsar.200390018>.
