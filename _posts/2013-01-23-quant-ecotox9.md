---
layout: post
title: "Quantitative Ecotoxicology, page 94, example 3.5"
date: 2013-01-23 21:32
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR
---




Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p94.csv) and read it into R:


{% highlight r %}
LEAD <- read.table("p94.csv", header = TRUE, sep = ";")
{% endhighlight %}





{% highlight r %}
head(LEAD)
{% endhighlight %}


{% highlight r %}
##    DAY LEAD
## 1 0.16 41.0
## 2 0.16 31.0
## 3 0.16 25.3
## 4 1.00 30.5
## 5 1.00 22.7
## 6 1.00 22.0
{% endhighlight %}




As always we first take a look at the data:

{% highlight r %}
plot(LEAD ~ DAY, LEAD)
{% endhighlight %}


![plot of chunk p94_raw](/figures/p94_raw.png) 


A simple power model may fit the data:

$$C_t = C_1~t^{−P}$$

We could fit such model as in example 3.3 via Nonlinear Least Squares or we could try to linearize the relationship by a ln-transform  of both DAY and LEAD:


{% highlight r %}
LEAD$LLEAD <- log(LEAD$LEAD)
LEAD$LDAY <- log(LEAD$DAY)
plot(LLEAD ~ LDAY, LEAD)
{% endhighlight %}


![plot of chunk p94_linear](/figures/p94_linear.png) 


Now we can us lm() to estimate the coefficients and check our model:


{% highlight r %}
# fit model
mod <- lm(LLEAD ~ LDAY, data = LEAD)
{% endhighlight %}



The residuals show no pattern:

{% highlight r %}
plot(mod, which = 1)
{% endhighlight %}


![plot of chunk p94_residuals](/figures/p94_residuals.png) 


From the model-output:

{% highlight r %}
mod_sum <- summary(mod)
mod_sum
{% endhighlight %}


{% highlight r %}
## 
## Call:
## lm(formula = LLEAD ~ LDAY, data = LEAD)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.4568 -0.1789  0.0372  0.1689  0.4169 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   3.0008     0.0641   46.80  < 2e-16 ***
## LDAY         -0.2715     0.0313   -8.67  1.5e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 0.238 on 22 degrees of freedom
## Multiple R-squared: 0.773,	Adjusted R-squared: 0.763 
## F-statistic: 75.1 on 1 and 22 DF,  p-value: 1.53e-08
{% endhighlight %}




We see that out fitted model hast the formula:
$$Ln(LEAD) = 3.0008 - 0.272 ln(DAY)$$
with an R-squared of 0.77 and is statistically significant. The standard errors for the two parameters are 0.064 and 0.031.

So our backtransformed model would be:
$$ LEAD = exp(3.0008)~Day^{-0.272} = 20.68~Day^{-0.272}$$

Finally we can also plot our model:

{% highlight r %}
plot(LLEAD ~ LDAY, LEAD)
abline(mod)
{% endhighlight %}


![plot of chunk p94_model](/figures/p94_model.png) 



Code and data are available at my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under filename 'p94'.

