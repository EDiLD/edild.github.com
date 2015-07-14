---
layout: post
title: "Quantitative Ecotoxicology, Page 45, Example 2.5, Gehan-Test"
date: 2012-12-01 22:52
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR
---

Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p45.csv) and read it into R:




{% highlight r %}
CADMIUM <- read.table("p45.csv", header = TRUE, sep = ";")
{% endhighlight %}


'Flip' the data:

{% highlight r %}
CADMIUM$FLIP <- abs(CADMIUM$CD - 100)
CADMIUM
{% endhighlight %}

{% highlight r %}
##      CD SITE FLAG FLIP
## 1  81.3    A    1 18.7
## 2   4.9    A    1 95.1
## 3   4.6    A    1 95.4
## 4   3.5    A    1 96.5
## 5   3.4    A    1 96.6
## 6   3.0    A    1 97.0
## 7   2.9    A    1 97.1
## 8   1.4    B    1 98.6
## 9   0.8    B    1 99.2
## 10  0.7    B    1 99.3
## 11  0.6    A    1 99.4
## 12  0.6    A    1 99.4
## 13  0.6    B    0 99.4
## 14  0.4    B    1 99.6
## 15  0.4    B    1 99.6
## 16  0.4    B    1 99.6
## 17  0.4    B    0 99.6
## 18  0.3    B    0 99.7
## 19  0.2    A    0 99.8
{% endhighlight %}


And test for differences using survdiff from the survival package:

{% highlight r %}
require(survival)
# log-rank
fit <- survdiff(Surv(FLIP, FLAG) ~ SITE, data = CADMIUM, rho = 0)
fit
{% endhighlight %}

{% highlight r %}
## Call:
## survdiff(formula = Surv(FLIP, FLAG) ~ SITE, data = CADMIUM, rho = 0)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## SITE=A 10        9     4.99      3.23      5.53
## SITE=B  9        6    10.01      1.61      5.53
## 
##  Chisq= 5.5  on 1 degrees of freedom, p= 0.0187
{% endhighlight %}

{% highlight r %}
# Peto & Peto modification of the Gehan-Wilcoxon test
fit2 <- survdiff(Surv(FLIP, FLAG) ~ SITE, data = CADMIUM, rho = 1)
fit2
{% endhighlight %}

{% highlight r %}
## Call:
## survdiff(formula = Surv(FLIP, FLAG) ~ SITE, data = CADMIUM, rho = 1)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## SITE=A 10     6.84     3.55      3.05      7.02
## SITE=B  9     2.84     6.13      1.76      7.02
## 
##  Chisq= 7  on 1 degrees of freedom, p= 0.00808
{% endhighlight %}




Code and data are available at my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p45'.
