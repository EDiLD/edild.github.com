---
layout: post
title: "Quantitative Ecotoxicology, Page 108, Example 3.7, Accumulation"
date: 2013-02-24 21:22
author: Eduard Szöcs
published: true
status: process
draft: false
tags: QETXR R
---




This is example 3.7 on page 108 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - reproduced with R. This example is about accumulation in mosquitofish (*Gambusia holbrooki*). 

Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p108.csv) and read it into R:


{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p108.csv",
ssl.verifypeer = FALSE)
MERCURY <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}



{% highlight text %}
## Error in read.table(text = url, header = TRUE, sep = ";"): no lines available in input
{% endhighlight %}

{% highlight r %}
head(MERCURY)
{% endhighlight %}



{% highlight text %}
## Error in head(MERCURY): object 'MERCURY' not found
{% endhighlight %}

This is pretty much like the previous examples: 

We fit a nonlinear model to our data
.
The model is given in equation 3.42 of the book:

$$C_t = \frac{k_u}{k_e} C_1 (1-e^{-k_e t})$$


{% highlight r %}
plot(MERCURY)
{% endhighlight %}



{% highlight text %}
## Error in plot(MERCURY): object 'MERCURY' not found
{% endhighlight %}

We can specify the model as follows:

{% highlight r %}
mod <- nls(HG ~ KU / KE * 0.24 * (1 - exp(-KE * DAY)), 
           data = MERCURY, 
           start = list(KU = 1000, KE = 0.5))
{% endhighlight %}



{% highlight text %}
## Error in nls(HG ~ KU/KE * 0.24 * (1 - exp(-KE * DAY)), data = MERCURY, : object 'MERCURY' not found
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
## Error in summary(mod): object 'mod' not found
{% endhighlight %}
So the parameter estimates are:

* $k_e = 0.589 \pm 0.106$
* $k_u = 1866.7 \pm 241.784$

The BCF is given as $BCF = \frac{k_u}{k_e} = 3171.4$

{% highlight r %}
BCF = coef(mod)[1] / coef(mod)[2]
{% endhighlight %}



{% highlight text %}
## Error in coef(mod): object 'mod' not found
{% endhighlight %}



{% highlight r %}
BCF
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'BCF' not found
{% endhighlight %}

From this we can predict the fish concentration as $$C_{fish}=BCF \cdot C_1=761.14$$

{% highlight r %}
BCF * 0.24
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'BCF' not found
{% endhighlight %}

Finally we plot the data and our model:

{% highlight r %}
DAY_pred <- seq(0, 6, by = 0.1) 
# Raw data
plot(MERCURY)
{% endhighlight %}



{% highlight text %}
## Error in plot(MERCURY): object 'MERCURY' not found
{% endhighlight %}



{% highlight r %}
# add model
lines(DAY_pred, predict(mod, newdata = data.frame(DAY = DAY_pred)))
{% endhighlight %}



{% highlight text %}
## Error in predict(mod, newdata = data.frame(DAY = DAY_pred)): object 'mod' not found
{% endhighlight %}



{% highlight r %}
# add model-equation
text(3, 100, bquote(HG == .(BCF*0.24)%.%(1-exp(-.(coef(mod)[2])%.%DAY))))
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'BCF' not found
{% endhighlight %}


Once again we reproduced the results as in the book using R :)
The differences for BCF and $C_{fish}$ are due to rounding errors.


Code and data are available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p108'.
