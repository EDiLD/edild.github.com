---
layout: post
title: "Quantitative Ecotoxicology, page 39, example 2.3, Kaplan–Meier estimates"
date: 2012-12-01 22:47
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR
---
Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p39.csv) and read it into R:




{% highlight r %}
SULFATE <- read.table("p39.csv", header = TRUE, sep = ";")
{% endhighlight %}


Convert left to right censored data:

{% highlight r %}
SULFATE$FLIP <- abs(SULFATE$SO4 - 8)
SULFATE
{% endhighlight %}

{% highlight r %}
##    SO4 FLAG FLIP
## 1  7.9    1  0.1
## 2  7.7    1  0.3
## 3  7.1    1  0.9
## 4  6.9    1  1.1
## 5  6.5    1  1.5
## 6  6.2    1  1.8
## 7  6.1    1  1.9
## 8  5.7    1  2.3
## 9  5.6    1  2.4
## 10 5.2    1  2.8
## 11 4.5    1  3.5
## 12 4.1    1  3.9
## 13 4.0    1  4.0
## 14 3.6    1  4.4
## 15 3.5    1  4.5
## 16 3.5    1  4.5
## 17 3.3    1  4.7
## 18 2.6    1  5.4
## 19 2.5    0  5.5
## 20 2.5    0  5.5
{% endhighlight %}


The Kaplan-Meier estimates can be calculated using survfit() from the survival package:

{% highlight r %}
require(survival)
fit <- survfit(Surv(FLIP, FLAG) ~ 1, data = SULFATE, conf.type = "plain")
fit
{% endhighlight %}

{% highlight r %}
## Call: survfit(formula = Surv(FLIP, FLAG) ~ 1, data = SULFATE, conf.type = "plain")
## 
## records   n.max n.start  events  median 0.95LCL 0.95UCL 
##   20.00   20.00   20.00   18.00    3.15    1.80    4.50
{% endhighlight %}


I set conf.type="plain" to be concordant with 'CONFTYPE=LINEAR' from SAS.

The median of 3.15, 95% CI [1.8, 4.5] is the same as with SAS.

Finally a quick plot:

{% highlight r %}
plot(fit)
{% endhighlight %}

![plot of chunk p39](/figures/p39.png) 



Code and data are available at my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under filename 'p39'.
