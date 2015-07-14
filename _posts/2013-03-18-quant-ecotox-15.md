---
layout: post
title: "Quantitative Ecotoxicology, Page 162, Example 4.7, Duplicate Treatments"
date: 2013-03-18 00:24
author: Eduard Szöcs
published: true
status: process
draft: false
tags: QETXR R
---

{% highlight text %}
## Error in BibOptions(check.entries = check.entries, style = style, hyperlink = hyperlink, : Invalid name specified, see ?BibOptions
{% endhighlight %}


This is a short one (example 4.7 on page 1621 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647)). 

First we create the data as matrix:

{% highlight r %}
TEST <- matrix(c(1,19,6,14), byrow=TRUE, ncol = 2, 
               dimnames=list(c('Tank_A', 'Tank_B'), c('Number_Dead', 'Number_Surviving')))
TEST
{% endhighlight %}



{% highlight text %}
##        Number_Dead Number_Surviving
## Tank_A           1               19
## Tank_B           6               14
{% endhighlight %}


The we can easily run fisher's Exact test for this 2x2 table using the function `fisher.test()`:

{% highlight r %}
fisher.test(TEST)
{% endhighlight %}



{% highlight text %}
## 
## 	Fisher's Exact Test for Count Data
## 
## data:  TEST
## p-value = 0.091
## alternative hypothesis: true odds ratio is not equal to 1
## 95 percent confidence interval:
##  0.0025451 1.2461419
## sample estimates:
## odds ratio 
##    0.12883
{% endhighlight %}



{% highlight r %}
fisher.test(TEST, alternative='greater')
{% endhighlight %}



{% highlight text %}
## 
## 	Fisher's Exact Test for Count Data
## 
## data:  TEST
## p-value = 1
## alternative hypothesis: true odds ratio is greater than 1
## 95 percent confidence interval:
##  0.0051465       Inf
## sample estimates:
## odds ratio 
##    0.12883
{% endhighlight %}



{% highlight r %}
fisher.test(TEST, alternative='less')
{% endhighlight %}



{% highlight text %}
## 
## 	Fisher's Exact Test for Count Data
## 
## data:  TEST
## p-value = 0.046
## alternative hypothesis: true odds ratio is less than 1
## 95 percent confidence interval:
##  0.00000 0.96589
## sample estimates:
## odds ratio 
##    0.12883
{% endhighlight %}

The results are identical to the one in the book.
