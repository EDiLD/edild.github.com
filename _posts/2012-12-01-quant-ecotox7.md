---
layout: post
title: "Quantitative Ecotoxicology, Page 85, Example 3.2, Nonlinear Regression"
date: 2012-12-01 23:04
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR
---


Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p85.csv) and read it into R:

{% highlight r %}
OYSTERZN <- read.table("p85.csv", header = TRUE, sep = ";")
head(OYSTERZN)
{% endhighlight %}

{% highlight r %}
##   DAY ZINC
## 1   1  700
## 2   1  695
## 3   1  675
## 4   1  630
## 5   1  606
## 6   1  540
{% endhighlight %}


{% highlight r %}
plot(ZINC ~ DAY, data = OYSTERZN)
{% endhighlight %}

![plot of chunk p85_raw](/figures/p85_raw.jpeg) 


First we fit a **nonlinear Regression without weighting**.

{% highlight r %}
mod_noweight <- nls(ZINC ~ INITACT * exp((-(KE + 0.00283)) * DAY), data = OYSTERZN, 
    start = list(KE = 0.004, INITACT = 500))
{% endhighlight %}


So we fit the model

$$C_t = C_0 e^{-(k_{e1} + k_{e2}) \cdot t}$$

to our data.

In the R formula **ZINC** corresponds to $C_t$,   
**INITACT** to $$C_0$$,   
**KE** to $$k_{e1}$$,   
**0.00283** is the decay rate constant for 65-Zn $k_{e2}$,   
and **DAY** to $t$.


Were are going to estimate **KE** and **INITACT** and also supplied some start-values for the algorithm.

We can look a the summary to get the estimates and standard error:

{% highlight r %}
summary(mod_noweight)
{% endhighlight %}

{% highlight r %}
## 
## Formula: ZINC ~ INITACT * exp((-(KE + 0.00283)) * DAY)
## 
## Parameters:
##         Estimate Std. Error t value Pr(>|t|)    
## KE      2.68e-03   6.68e-04    4.01  0.00015 ***
## INITACT 4.65e+02   2.04e+01   22.86  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 98.8 on 71 degrees of freedom
## 
## Number of iterations to convergence: 3 
## Achieved convergence tolerance: 5.57e-06
{% endhighlight %}





The resulting estimates of $$k_{e1}$$ and $$C_0$$ are $$0.00268 \pm 0.00067$$ and $$465 \pm 20$$.

We can investigate the residuals, which show a clear pattern:


{% highlight r %}
res_mod_noweight <- resid(mod_noweight)
plot(OYSTERZN$DAY, res_mod_noweight)
abline(h = 0)
{% endhighlight %}

![plot of chunk p85_residuals_nls](/figures/p85_residuals_nls.jpeg) 


Secondly, we run a **nonlinear regression with day-squared weighting**:

We use day^2 as weights and add there a column to our data:

{% highlight r %}
OYSTERZN$WNLIN <- OYSTERZN$DAY^2
{% endhighlight %}


We run again nls, but now we supply this new column as weights:

{% highlight r %}
mod_weight <- nls(ZINC ~ INITACT * exp((-(KE + 0.00283)) * DAY), data = OYSTERZN, 
    weights = OYSTERZN$WNLIN, start = list(KE = 0.004, INITACT = 500))
summary(mod_weight)
{% endhighlight %}

{% highlight r %}
## 
## Formula: ZINC ~ INITACT * exp((-(KE + 0.00283)) * DAY)
## 
## Parameters:
##         Estimate Std. Error t value Pr(>|t|)    
## KE      2.44e-03   3.47e-04    7.03  1.1e-09 ***
## INITACT 4.55e+02   3.82e+01   11.89  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 7550 on 71 degrees of freedom
## 
## Number of iterations to convergence: 4 
## Achieved convergence tolerance: 5.42e-07
{% endhighlight %}




The estimates $$0.00244 \pm 0.00035$$ and  $$455 \pm 38$$ are quite similar to the non-weighted regression.

We could plot the two models and the data:


{% highlight r %}
# extract the fitted values
fit_mod_noweight <- fitted(mod_noweight)
fit_mod_weight <- fitted(mod_weight)
# plot data
plot(ZINC ~ DAY, data = OYSTERZN)
# add fitted values
lines(OYSTERZN$DAY, fit_mod_noweight)
lines(OYSTERZN$DAY, fit_mod_weight, lty = "dashed")
# add legend
legend("topright", legend = c("nonweighted", "weighted"), lty = c("solid", "dashed"))
{% endhighlight %}

![plot of chunk p85_nls_fitted](/figures/p85_nls_fitted.jpeg) 



Finally we can also fit a **linear model** to the transformed Zinc-Concentrations:

First we ln-transform the concentrations:

{% highlight r %}
OYSTERZN$LZINC <- log(OYSTERZN$ZINC)
{% endhighlight %}


We see that the data has now linear trend:

{% highlight r %}
plot(LZINC ~ DAY, OYSTERZN)
{% endhighlight %}

![plot of chunk p85_raw2](/figures/p85_raw2.jpeg) 


And fit a linear regression:

{% highlight r %}
mod_lm <- lm(LZINC ~ DAY, data = OYSTERZN)
sum_lm <- summary(mod_lm)
sum_lm
{% endhighlight %}

{% highlight r %}
## 
## Call:
## lm(formula = LZINC ~ DAY, data = OYSTERZN)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.8974 -0.2448 -0.0709  0.2958  0.5715 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  6.073833   0.056834   106.9   <2e-16 ***
## DAY         -0.005314   0.000243   -21.9   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 0.354 on 71 degrees of freedom
## Multiple R-squared: 0.871,  Adjusted R-squared: 0.869 
## F-statistic:  478 on 1 and 71 DF,  p-value: <2e-16
{% endhighlight %}


which is fitting the model
$$ln(Zn) = a \cdot day + intercept$$,
with a = `-0.0053` and intercept = `6.07`

Now plot data and model, as well as the residuals:

{% highlight r %}
# fitted values
fit_mod_lm <- fitted(mod_lm)

# data + fitted
plot(LZINC ~ DAY, OYSTERZN)
lines(OYSTERZN$DAY, fit_mod_lm)
{% endhighlight %}

![plot of chunk p85_lm_fitted](/figures/p85_lm_fitted1.jpeg) 

{% highlight r %}

# residuals
plot(mod_lm, which = 1)
{% endhighlight %}

![plot of chunk p85_lm_fitted](/figures/p85_lm_fitted2.jpeg) 


The mean square error can be calculated from the summary:

{% highlight r %}
# MSE
sum_lm$sigma^2
{% endhighlight %}

{% highlight r %}
## [1] 0.12516
{% endhighlight %}

From which we can get an unbiased estimate of $$C_0$$:

{% highlight r %}
# unbiased estimate for C_0: exp(MSE/2) * exp(Intercept)
exp(sum_lm$sigma^2/2) * exp(sum_lm$coefficients[1, 1])
{% endhighlight %}

{% highlight r %}
## [1] 462.39
{% endhighlight %}

where 

{% highlight r %}
sum_lm$coefficients[1, 1]
{% endhighlight %}

extracts the intercept from the summary.

The estimated $$k$$ in the summary output is $$-0.00531 \pm 0.00024$$, and 
$$k_e = k - decayrate = 0.00531 - 0.00283 = 0.00248$$

This result is similar to the weighted and non weighted nonlinear regression.
Again we have the same results as with SAS :) [Small deviations may be due to rounding error]

Code and data are available at my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p85'.
