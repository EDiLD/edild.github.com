---
layout: post
title: "Quantitative Ecotoxicology, Page 223, Example 5.1, with GLM"
date: 2015-01-19 22:08
author: Eduard Szöcs
published: true
status: publish
draft: true
tags: QETXR R
---


 
[Previously](http://edild.github.io/quant-ecotox20/), I showed how to analyze the fish survival data using the arcsine square root transformation.
Warton & Hui (2011) demonstrated that *the arcsine transform should not be used in either circumstance*, but instead use Generalized Linear Models.
This post is about how to analyse example 5.1 on page 223 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) using Generalised Linear Models.
 
 
#### Introduction
 
Data of this type (*x out of n*) can be directly modelled using a [binomial distribution](http://en.wikipedia.org/wiki/Binomial_distribution).
 
The binomial distribution is described by two parameters:
 
$$Bin(N, \pi)$$
 
where N is the number of fish and $\pi$ is the probabilty of survial.
 
 
Here I plotted the binomial distribution for then fish at different probabilities ($\pi = 0.5, 0.75, 0.95, 0.05$):

{% highlight r %}
require(ggplot2)
require(reshape2)
df <- data.frame(k = 0:10, 
                 p1 = dbinom(0:10, size = 10, prob = 0.5),
                 p2 = dbinom(0:10, size = 10, prob = 0.75),
                 p3 = dbinom(0:10, size = 10, prob = 0.95),
                 p4 = dbinom(0:10, size = 10, prob = 0.05))
dfm <- melt(df, id = 'k')
 
ggplot(dfm, aes(x = factor(k), y = value, fill = variable)) +
  geom_bar(stat = 'identity', position = "dodge")+
  labs(x = expression(pi), y = expression(p(pi))) +
  scale_fill_discrete("Distribution", 
                    labels = c("Bin(10, 0.5)", "Bin(10, 0.75)", "Bin(10, 0.95)", "Bin(10, 0.05)")) +
  theme_bw()
{% endhighlight %}

![plot of chunk unnamed-chunk-2](/figures/unnamed-chunk-2-1.png) 
 
We can now build a model, where the probability of success is a function of treatment:
 
$$  y_i \sim Bin(N, \pi_i) \\
  logit~(\pi_i) = \alpha + \beta x_i \\
  var(y_i) =  \pi_i (1 - \pi_i) / N $$
  
Let's explain this a little bit:
 
 
#### Which one is better?
I would say GLMs! They have greater power (Warton & Hui, 2011) and are simpler to interpret.
Nowadays, there is no need to transform the data as GLMs are readily available in most software packages.
 
 
#### References
 
* Warton, D. I., & Hui, F. K. (2011). The arcsine is asinine: the analysis of proportions in ecology. Ecology, 92(1), 3-10.