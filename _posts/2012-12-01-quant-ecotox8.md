---
layout: post
title: "Quantitative Ecotoxicology, Page 89, Example 3.3, Backstripping"
date: 2012-12-01 23:52
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR
---

Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p89.csv) and read it into R:


{% highlight r %}
MERCURY <- read.table("p89.csv", header = TRUE, sep = ";")
head(MERCURY)
{% endhighlight %}

{% highlight r %}
##   DAY   PCI
## 1   3 27400
## 2   6 17601
## 3  10 12950
## 4  14 11157
## 5  22  9410
## 6  31  9150
{% endhighlight %}


{% highlight r %}
plot(PCI ~ DAY, data = MERCURY)
abline(v = 22)
{% endhighlight %}

![plot of chunk p89_raw](/figures/p89_raw.jpeg) 


We can identify two compartments on this plot: A slow one after day 22 and a fast one before day 22.

First we estimate $$C_B$$ and $$k_B$$ for the slow compartment, using linear regression of ln-transformed activity against day and predict from this slow compartment the activity over the whole period:


{% highlight r %}
# ln-transformation
MERCURY$LPCI <- log(MERCURY$PCI)
# fit linear model for day 31 to 94
mod_slow <- lm(LPCI ~ DAY, data = MERCURY[MERCURY$DAY > 22, ])
sum_mod_slow <- summary(mod_slow)
sum_mod_slow
{% endhighlight %}

{% highlight r %}
## 
## Call:
## lm(formula = LPCI ~ DAY, data = MERCURY[MERCURY$DAY > 22, ])
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.2502 -0.0657  0.0605  0.0822  0.1386 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  9.43096    0.13987   67.42  2.6e-12 ***
## DAY         -0.01240    0.00213   -5.82   0.0004 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 0.135 on 8 degrees of freedom
## Multiple R-squared: 0.809,  Adjusted R-squared: 0.785 
## F-statistic: 33.9 on 1 and 8 DF,  p-value: 0.000395
{% endhighlight %}

{% highlight r %}
exp(coef(mod_slow)[1])
{% endhighlight %}

{% highlight r %}
## (Intercept) 
##       12468
{% endhighlight %}

So this gives us the model $$C = 12468 \cdot e^{-0.0124 * Day}$$ for the slow component.

We do a bias correction and predict the activity for the whole data-range:

{% highlight r %}
# bias-correction
corr_mod_slow <- exp(sum_mod_slow$sigma^2/2)
# add bias corrected predicts to data.frame predict takes the whole
# data.frame es newdata, so we get predicts for every day.
MERCURY$PRED <- exp(predict(mod_slow, newdata = MERCURY)) * corr_mod_slow
# save C_B and k_B as objects (used later...)
CB <- exp(coef(mod_slow)[1]) * corr_mod_slow
KCB <- abs(coef(mod_slow)[2])
{% endhighlight %}


The residuals from these predictions for day 3 to 22 are associated with the fast compartment.
And we fit a linear regression to the ln-transformed residuals for the fast compartment.


{% highlight r %}
plot(LPCI ~ DAY, data = MERCURY)
points(MERCURY$DAY[1:4], MERCURY$LPCI[1:4], pch = 16)
abline(a = log(CB), b = -KCB)
for (i in 1:4) {
    lines(c(MERCURY$DAY[i], MERCURY$DAY[i]), c(log(MERCURY$PRED[i]), MERCURY$LPCI[i]), 
        lwd = 2)
}
{% endhighlight %}

![plot of chunk p89_residuals](/figures/p89_residuals.jpeg) 



{% highlight r %}
# extract residuals
MERCURY$ERROR <- MERCURY$PCI - MERCURY$PRED
MERCURY$ERROR[1:4]
{% endhighlight %}

{% highlight r %}
## [1] 15276.20  5920.03  1834.41   579.42
{% endhighlight %}

{% highlight r %}
# fit linear model to ln(residuals) for Day 3 to 22
mod_fast <- lm(log(ERROR) ~ DAY, data = MERCURY[MERCURY$DAY < 22, ])
sum_mod_fast <- summary(mod_fast)
sum_mod_fast
{% endhighlight %}

{% highlight r %}
## 
## Call:
## lm(formula = log(ERROR) ~ DAY, data = MERCURY[MERCURY$DAY < 22, 
##     ])
## 
## Residuals:
##       1       2       3       4 
##  0.0278 -0.0304 -0.0157  0.0183 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 10.49602    0.03755   279.5 0.000013 ***
## DAY         -0.29659    0.00407   -72.9  0.00019 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 0.0337 on 2 degrees of freedom
## Multiple R-squared:    1,	Adjusted R-squared: 0.999 
## F-statistic: 5.32e+03 on 1 and 2 DF,  p-value: 0.000188
{% endhighlight %}

{% highlight r %}
exp(coef(mod_fast)[1])
{% endhighlight %}

{% highlight r %}
## (Intercept) 
##       36171
{% endhighlight %}


So the model for the fast component is: $$C = 36171 \cdot e^{-0.297 * Day}$$


{% highlight r %}
# bias correction
corr_mod_fast <- exp(sum_mod_fast$sigma^2/2)
# save C_A and k_A as objects
CA <- exp(coef(mod_fast)[1]) * corr_mod_fast
KCA <- abs(coef(mod_fast)[2])
{% endhighlight %}



Now we have two models: one for the fast component and one for the slow component, and we can make a plot similar to Figure 8.1 in Newman and Clements (2008, pp. 119–120). 


{% highlight r %}
plot(LPCI ~ DAY, data = MERCURY)
abline(mod_slow)
abline(mod_fast, lty = "dotted")
legend("topright", c("slow", "backstripped-fast"), lty = c("solid", "dashed"), 
    cex = 0.8)
{% endhighlight %}

![plot of chunk p89_backstripped](/figures/p89_backstripped.jpeg) 

{% highlight r %}
# Estimates
c(CA, KCA, CB, KCB)
{% endhighlight %}

{% highlight r %}
## (Intercept)         DAY (Intercept)         DAY 
##  3.6192e+04  2.9659e-01  1.2583e+04  1.2403e-02
{% endhighlight %}


We can use this estimates as start-values to fit a non-linear Model to the data (therefore we stored them into objects).

We want to fit the following model:
$$C=C_A e^{-k_A \cdot Day} + C_B e^{-k_B \cdot Day}$$


{% highlight r %}
nls_mod1 <- nls(PCI ~ CA * exp(-KCA * DAY) + CB * exp(-KCB * DAY), 
                data = MERCURY, 
                algorithm = "port",    # to use the bonds
                start = list(KCB = KCB, KCA = KCA, CB = CB, CA = CA),
                lower = c(0, 0, 5000, 20000), 
                upper = c(1, 1, 20000, 45000))
sum_nls_mod1 <- summary(nls_mod1)
sum_nls_mod1
{% endhighlight %}

{% highlight r %}
## 
## Formula: PCI ~ CA * exp(-KCA * DAY) + CB * exp(-KCB * DAY)
## 
## Parameters:
##     Estimate Std. Error t value Pr(>|t|)    
## KCB 1.19e-02   1.41e-03    8.45  3.9e-06 ***
## KCA 2.97e-01   4.47e-02    6.66  3.6e-05 ***
## CB  1.22e+04   8.58e+02   14.27  1.9e-08 ***
## CA  3.79e+04   4.82e+03    7.86  7.7e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 701 on 11 degrees of freedom
## 
## Algorithm "port", convergence message: relative convergence (4)
{% endhighlight %}

* $$C_A$$ is estimated as $$37909 \pm 4824$$
* $$C_B$$ is estimated as $$12245 \pm 858$$
* $$k_A$$ is estimated as $$0.297 \pm 0.045$$
* $$k_A$$ is estimated as $$0.012 \pm 0.001$$


And finally we plot data and model.

{% highlight r %}
plot(PCI ~ DAY, data = MERCURY, type = "n")
points(MERCURY$DAY, MERCURY$PCI, pch = ifelse(MERCURY$DAY <= 22, 16, 17))
# smooth line
pred_nls_mod1 <- predict(nls_mod1, newdata = data.frame(DAY = seq(0, 100, 1)))
lines(seq(0, 100, 1), pred_nls_mod1)
legend("topright", c("Fast", "slow"), pch = c(16, 17))
{% endhighlight %}

![plot of chunk p89_nls](/figures/p89_nls.jpeg) 



Again we get nearly the same results with R, except for some differences in the linear models.

This is probably due to the bias-correction in slow-component-model.
We have a MSE of

{% highlight r %}
sum_mod_slow$sigma^2
{% endhighlight %}

{% highlight r %}
## [1] 0.018349
{% endhighlight %}

which is identical to the book. From the previous example, the bias can be estimated as
$$e^{MSE/2}$$:

{% highlight r %}
exp(sum_mod_slow$sigma^2/2)
{% endhighlight %}

{% highlight r %}
## [1] 1.0092
{% endhighlight %}

which is different to the book (1.002). However we only use this as starting values for nls() and the results of the non-linear regression are the same in R and SAS.

I have no SAS at hand, so I cannot check this with SAS. However let me know if there is an error
in my calculations.


**Refs**

> Newman, Michael C., and William Henry Clements. Ecotoxicology: A Comprehensive Treatment. Boca Raton: Taylor /& Francis, 2008. 



Code and data are available at my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p89'.
