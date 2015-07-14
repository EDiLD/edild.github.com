---
layout: post
title: "Quantitative Ecotoxicology, Page 101, Example 3.6, Langmuir"
date: 2013-02-24 00:25
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR R
---





This is example 3.6 on page 101 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - reproduced with R. This example is about adsorption and how to fit an adsorption model to data.

Get the data from [here](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p101.csv) and read it into R: 


{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/p101.csv",
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
ZINC <- read.table(text = url, header = TRUE, sep = ";")
{% endhighlight %}

{% highlight r %}
head(ZINC)
{% endhighlight %}



{% highlight text %}
##      N     C
## 1 0.75 0.030
## 2 1.40 0.069
## 3 1.95 0.118
## 4 2.51 0.166
## 5 3.03 0.217
## 6 3.53 0.270
{% endhighlight %}

So we have a data.frame with two columns,
where N = amount adsorbed (mmol) per unit mass (g) and  C = equilibrium concentration in the aqueous phase (mmol/ml).

We want fit a Langmuir Model (Equation 3.28 in the book) to this data. 

The three methods described are:

* Nonlinear Regression
* linear transformation
* linear transformation with weighting



#### Nonlinear Regression

{% highlight r %}
mod_nls <- nls(N ~ (K*C*M)/(1+K*C), data = ZINC, 
           start = list(K = 3, M = 9), lower = 0, 
           algorithm = 'port')
{% endhighlight %}
This fits the model 

$$ N = \frac{KCM}{1+KC} $$ 

to the data. 

We supplied some starting values and specified the lower bonds for K and M as 0 (bonds can only be used with the port algorithm).

This gives us the estimates for K and M as:

{% highlight r %}
summary(mod_nls)
{% endhighlight %}



{% highlight text %}
## 
## Formula: N ~ (K * C * M)/(1 + K * C)
## 
## Parameters:
##   Estimate Std. Error t value Pr(>|t|)    
## K    2.097      0.188    11.1  3.8e-06 ***
## M    9.899      0.521    19.0  6.1e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.0929 on 8 degrees of freedom
## 
## Algorithm "port", convergence message: relative convergence (4)
{% endhighlight %}

* $K = 2.097 \pm 0.188$
* $M = 9.899 \pm 0.521$

The t and p-values of this output are not of interest for us (tests if the parameters deviate from 0).

We can plot the raw data and the model easily using the predict-function:

{% highlight r %}
plot(ZINC$C, ZINC$N, xlab = 'C', ylab = 'N')
# generate C-values to predict
x_n <- seq(min(ZINC$C), max(ZINC$C), length.out=200)
# add predicts to plot
lines(x_n, predict(mod_nls, newdata = data.frame(C = x_n)))
{% endhighlight %}

<img src="../figures/source/2013-02-24-quant-ecotox-10/plot-nls-1.png" title="plot of chunk plot-nls" alt="plot of chunk plot-nls" width="400px" />



#### Linear model of transformation
We use were the reciprocal transformation, so C/N versus C.
First we create a the transformed y-variable:

{% highlight r %}
ZINC$Y <- ZINC$C / ZINC$N
{% endhighlight %}

Fitting a linear model to this data is done with lm():

{% highlight r %}
mod_lm <- lm(Y ~ C, data = ZINC)
plot(ZINC$C, ZINC$Y, ylab = 'C/N', xlab = 'C')
abline(mod_lm)
{% endhighlight %}

<img src="../figures/source/2013-02-24-quant-ecotox-10/plot-lm-1.png" title="plot of chunk plot-lm" alt="plot of chunk plot-lm" width="400px" />

{% highlight r %}
summary(mod_lm)
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = Y ~ C, data = ZINC)
## 
## Residuals:
##       Min        1Q    Median        3Q       Max 
## -0.006926 -0.001708  0.000268  0.003081  0.003706 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.04351    0.00225    19.3  5.3e-08 ***
## C            0.11400    0.00754    15.1  3.6e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.0037 on 8 degrees of freedom
## Multiple R-squared:  0.966,	Adjusted R-squared:  0.962 
## F-statistic:  229 on 1 and 8 DF,  p-value: 3.62e-07
{% endhighlight %}
We get from this K and M as:

* $K = \frac{slope}{intercept} = \frac{0.114}{0.043} = 2.62$
* $M = \frac{1}{slope} = \frac{1}{0.114} = 8.77$

The R^2 is 0.966.


#### Linear model of transformation with weights
Newman used N^4 / C^2 weighting. So first we need to calculate the weights:

{% highlight r %}
ZINC$WGT = ZINC$N^4 / ZINC$C^2
{% endhighlight %}

And fit the linear model with weighting:

{% highlight r %}
mod_wgt <- lm(Y ~ C, data = ZINC, weights = ZINC$WGT)
summary(mod_wgt)
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = Y ~ C, data = ZINC, weights = ZINC$WGT)
## 
## Weighted Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.1911 -0.0834  0.0291  0.0580  0.0858 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.04708    0.00199    23.6  1.1e-08 ***
## C            0.10373    0.00568    18.3  8.3e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.105 on 8 degrees of freedom
## Multiple R-squared:  0.977,	Adjusted R-squared:  0.974 
## F-statistic:  333 on 1 and 8 DF,  p-value: 8.32e-08
{% endhighlight %}
The R^2 is slightly higher: 0.977.

The result for K is:

{% highlight r %}
coef(mod_wgt)[2] / coef(mod_wgt)[1]
{% endhighlight %}



{% highlight text %}
##      C 
## 2.2033
{% endhighlight %}

and for M:

{% highlight r %}
1 / coef(mod_wgt)[2]
{% endhighlight %}



{% highlight text %}
##      C 
## 9.6403
{% endhighlight %}

#### Are the models appropiate?

We can inspect the residuals of both models:



{% highlight r %}
par(mfrow = c(1,2))
# lm
plot(mod_lm, which = 1, main='linear model without weights')
# nls
plot(fitted(mod_nls), residuals(mod_nls), xlab = 'fitted', ylab = 'Residuals', main = 'nonlinear regression')
abline(h = 0, lty = 'dotted')
{% endhighlight %}

<img src="../figures/source/2013-02-24-quant-ecotox-10/plot-resid-1.png" title="plot of chunk plot-resid" alt="plot of chunk plot-resid" width="500px" />

The linear model clearly shows an arc-pattern in the residuals - so the data may not follow a linear relationship.
The nonlinear model performs better.



Once again we reproduced the same results as in the book using R :)
Code and data are available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p101'.


