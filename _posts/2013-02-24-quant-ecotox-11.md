---
layout: post
title: "Quantitative Ecotoxicology, Page 108, Example 3.7, Accumulation"
date: 2013-02-24 21:22
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR, R
---


 
 
This is example 3.7 on page 108 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - reproduced with R. This example is about accumulation in mosquitofish (*Gambusia holbrooki*).  
 
Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p108.csv) and read it into R:
 

{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p108.csv",
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
MERCURY <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}

{% highlight r %}
head(MERCURY)
{% endhighlight %}



{% highlight text %}
##   DAY  HG
## 1   0   0
## 2   1 380
## 3   2 540
## 4   3 570
## 5   4 670
## 6   6 780
{% endhighlight %}
 
This is pretty much like the previous examples: 
 
We fit a nonlinear model to our data
.
The model is given in equation 3.42 of the book:
 
$$C_t = \frac{k_u}{k_e} C_1 (1-e^{-k_e t})$$
 

{% highlight r %}
plot(MERCURY)
{% endhighlight %}

<img src="/figures/plot_raw-1.png" title="plot of chunk plot_raw" alt="plot of chunk plot_raw" width="400px" />
 
We can specify the model as follows:

{% highlight r %}
mod <- nls(HG ~ KU / KE * 0.24 * (1 - exp(-KE * DAY)), 
           data = MERCURY, 
           start = list(KU = 1000, KE = 0.5))
{% endhighlight %}
 
This equals to equation 3.42:
 
* $HG = C_t$
* $KU = k_u$
* $KE = k_e$
* $0.24 = C_1$
* $DAY = t$
 
 
Unlike in the book I did not specify bounds here (see the previous posts how to do this).
 
This results in:

{% highlight r %}
summary(mod)
{% endhighlight %}



{% highlight text %}
## 
## Formula: HG ~ KU/KE * 0.24 * (1 - exp(-KE * DAY))
## 
## Parameters:
##    Estimate Std. Error t value Pr(>|t|)   
## KU 1866.700    241.784    7.72   0.0015 **
## KE    0.589      0.106    5.55   0.0051 **
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 43.7 on 4 degrees of freedom
## 
## Number of iterations to convergence: 7 
## Achieved convergence tolerance: 2.1e-06
{% endhighlight %}
So the parameter estimates are:
 
* $k_e = 0.589 \pm 0.106$
* $k_u = 1866.7 \pm 241.784$
 
The BCF is given as $BCF = \frac{k_u}{k_e} = 3171.4$

{% highlight r %}
BCF = coef(mod)[1] / coef(mod)[2]
BCF
{% endhighlight %}



{% highlight text %}
##     KU 
## 3171.4
{% endhighlight %}
 
From this we can predict the fish concentration as $$C_{fish}=BCF \cdot C_1=761.14$$

{% highlight r %}
BCF * 0.24
{% endhighlight %}



{% highlight text %}
##     KU 
## 761.14
{% endhighlight %}
 
Finally we plot the data and our model:

{% highlight r %}
DAY_pred <- seq(0, 6, by = 0.1) 
# Raw data
plot(MERCURY)
# add model
lines(DAY_pred, predict(mod, newdata = data.frame(DAY = DAY_pred)))
# add model-equation
text(3, 100, bquote(HG == .(BCF*0.24)%.%(1-exp(-.(coef(mod)[2])%.%DAY))))
{% endhighlight %}

<img src="/figures/plot_model-1.png" title="plot of chunk plot_model" alt="plot of chunk plot_model" width="400px" />
 
 
Once again we reproduced the results as in the book using R :)
The differences for BCF and $C_{fish}$ are due to rounding errors.
 
 
Code and data are available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p108'.
