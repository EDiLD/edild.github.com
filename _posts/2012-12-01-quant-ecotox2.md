---
layout: post
title: "Quantitative Ecotoxicology, Page 33, Example 2.1, Winsorization"
date: 2012-12-01 22:36
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR
---
Get the data (Sulfate Concentrations from Savannah River (South Carolina) in mg / L)) from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p33.csv) and read it into R:

{% highlight r %}
ALL <- read.table("p33.csv", header = TRUE, sep = ";")
{% endhighlight %}

So we have a data.frame with one variable and 21 observations:

{% highlight r %}
str(ALL)
{% endhighlight %}

{% highlight r %}
## 'data.frame':  21 obs. of  1 variable:
##  $ SO4: num  1.3 2.3 2.6 3.3 3.5 3.5 3.6 4 4.1 4.5 ...
{% endhighlight %}

{% highlight r %}
ALL$SO4
{% endhighlight %}

{% highlight r %}
##  [1] 1.3 2.3 2.6 3.3 3.5 3.5 3.6 4.0 4.1 4.5 5.2 5.6 5.7 6.1 6.2 6.5 6.9
## [18] 7.1 7.7 7.9 9.9
{% endhighlight %}



Winsorization replaces extreme data values with less extreme values. I have written a small function to run the winsorisation:

{% highlight r %}
winsori <- function(x, width = 2) {
    # check if sorted
    if (is.unsorted(x)) 
        stop("Values must be sorted!")
    # get number of observations
    n <- length(x)
    # Replace lowest
    x[1:width] <- x[width + 1]
    # replace highest
    x[(n - width + 1):n] <- x[(n - width)]
    x
}
{% endhighlight %}


The function takes a ordered vector and replaces the 2 highest and 2 lowest values (can be changed by the 'width'-Argument by their neighbors.

We can apply this function to our data and safe it as new column:

{% highlight r %}
ALL$SO4_win <- winsori(ALL$SO4)
# display the first and 5 last rows
ALL[c(1:5, 17:21), ]
{% endhighlight %}

{% highlight r %}
##    SO4 SO4_win
## 1  1.3     2.6
## 2  2.3     2.6
## 3  2.6     2.6
## 4  3.3     3.3
## 5  3.5     3.5
## 17 6.9     6.9
## 18 7.1     7.1
## 19 7.7     7.7
## 20 7.9     7.7
## 21 9.9     7.7
{% endhighlight %}


Worked as expected.
The Winsorized mean and standard-deviation is:

{% highlight r %}
# mean
mean(ALL$SO4_win)
{% endhighlight %}

{% highlight r %}
## [1] 5.081
{% endhighlight %}

{% highlight r %}
# standard deviation
sd(ALL$SO4_win)
{% endhighlight %}

{% highlight r %}
## [1] 1.792
{% endhighlight %}


For the Winsorized Standard Deviation we need again a homemade function:

{% highlight r %}
sw <- function(x, width = 2) {
    n <- length(x)
    sd(x) * (n - 1)/(n - 2 * width - 1)
}
sw(ALL$SO4_win)
{% endhighlight %}

{% highlight r %}
## [1] 2.24
{% endhighlight %}


And lastly we calculate the mean for the trimmed data (remove two observation from each tail):

{% highlight r %}
mean(ALL$SO4, trim = 2/21)
{% endhighlight %}

{% highlight r %}
## [1] 5.065
{% endhighlight %}


Code and data are available at my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under filename 'p33'.

