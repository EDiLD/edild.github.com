---
layout: post
title: "Quantitative Ecotoxicology, Page 35, Robust Regression on Order Statistics"
date: 2012-12-01 22:37
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR
---

Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p35.csv) and read it into R:




{% highlight r %}
SO4 <- read.table("p35.csv", header = TRUE, sep = ";")
{% endhighlight %}



First we need to convert the vector indicating if an observation is censored to TRUE/FALSE:
I store it in a new colum called 'rem2' (you could also overwrite rem):

{% highlight r %}
SO4$rem2 <- ifelse(SO4$rem == "<", TRUE, FALSE)
SO4
{% endhighlight %}

{% highlight r %}
##    value rem  rem2
## 1    2.5   <  TRUE
## 2    2.5   <  TRUE
## 3    2.6   X FALSE
## 4    3.3   X FALSE
## 5    3.5   X FALSE
## 6    3.5   X FALSE
## 7    3.6   X FALSE
## 8    4.0   X FALSE
## 9    4.1   X FALSE
## 10   4.5   X FALSE
## 11   5.2   X FALSE
## 12   5.6   X FALSE
## 13   5.7   X FALSE
## 14   6.1   X FALSE
## 15   6.2   X FALSE
## 16   6.5   X FALSE
## 17   6.9   X FALSE
## 18   7.1   X FALSE
## 19   7.7   X FALSE
## 20   7.9   X FALSE
## 21   9.9   X FALSE
{% endhighlight %}


Then we can run the Robust Regression on Order Statistics with the ros() function from the NADA package:

{% highlight r %}
require(NADA)
rs <- ros(SO4$value, SO4$rem2)
print(rs)
{% endhighlight %}

{% highlight r %}
##      n  n.cen median   mean     sd 
## 21.000  2.000  5.200  5.158  2.071
{% endhighlight %}


Which gives the same mean and standard deviation as the SAS-Makro (5.16 and 2.07).

Code and data are available at my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under filename 'p35'.
