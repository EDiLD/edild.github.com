---
layout: post
title: "Collinearity"
date: 2013-12-13 15:12
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: R
---
 


 
It has been quiet the last months here - main reason is that I'm working on my master's thesis.
I have already prepared some more examples from 'Quantitative Ecotoxicolgy', but I didn't come to publish them here.
 
 
This post is about collinearity and the implications for linear models. The best way to explore this is by simulations - where I create data with known properties and look what happens.
 
 
### Create correlated random variables
 
The first problem for this simulation was: How can we create correlated random variables?
 
This simulation is similar to the one in Dormann (2013), where it is also mentioned that one could use Cholesky Decompostion to create correlated variables: 
 
What this function function does:
 
* Create two normal random variables (X1, X2 ~ N(0, 1))
* Create desired correlation matrix
* Compute the Choleski factorization of the correlation matrix ( R )
* Apply the resulting matrix on the two variables (this rotates, shears and scales the variables so that they are correlated)
* Create a dependent variable `y` after the model:
 
`y ~ 5 + 7*X1 + 7*X2 + e` with `e ~ N(0, 1)`
 
 

{% highlight r %}
#############################################################################
# create two correlated variables and a dependent variable
# n : number of points
# p:correlation
 
datagen <- function(n , p){
  # random points N(0,1)
  x1 <- rnorm(n)
  x2 <- rnorm(n)
  X <- cbind(x1, x2)
  
  # desired correlation matrix
  R <- matrix(c(1, p, p, 1), ncol = 2)
  
  # use cholesky decomposition `t(U) %*% U = R`
  U <- chol(R)
  corvars <- X %*% U
  
  # create dependent variable after model:
  # y ~ 5 + 7 * X1 + 7 * X2 + e | E ~ N(0,10)
  y <- 5 + 7*corvars[,1] + 7*corvars[,2] + rnorm(n, 0, 1)
  df <- data.frame(y, corvars)
  return(df)
}
{% endhighlight %}
 
 
Let's see if it works. 
This creates two variables with 1000 observations with a correlation of 0.8 between them and a dependent variable.

{% highlight r %}
df1 <- datagen(n = 1000, p = 0.8)
{% endhighlight %}
 
The correlation between X1 and X2 is as desired nearly 0.8

{% highlight r %}
cor(df1)
{% endhighlight %}



{% highlight text %}
##          y      X1      X2
## y  1.00000 0.94290 0.94014
## X1 0.94290 1.00000 0.78396
## X2 0.94014 0.78396 1.00000
{% endhighlight %}
 
And the data follows the specified model.

{% highlight r %}
mod <- lm(y~X1+X2, data = df1)
summary(mod)
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = y ~ X1 + X2, data = df1)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -2.734 -0.724 -0.009  0.704  3.374 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   4.9554     0.0318     156   <2e-16 ***
## X1            7.0168     0.0527     133   <2e-16 ***
## X2            6.9797     0.0537     130   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1 on 997 degrees of freedom
## Multiple R-squared:  0.994,	Adjusted R-squared:  0.994 
## F-statistic: 8.02e+04 on 2 and 997 DF,  p-value: <2e-16
{% endhighlight %}



{% highlight r %}
pairs(df1)
{% endhighlight %}

![plot of chunk example_datagen3](/figures/example_datagen3-1.png) 
 
 
### Methods to spot collinearity
 
Dormann lists eight methods to spot collinearity (see their Table 1). I will only show how to calculate two of those (but see the Appendix of the Dormann paper for code to all methods):
 
#### Absolute value of correlation coefficients r

{% highlight r %}
( cormat <- cor(df1[,2:3]) )
{% endhighlight %}



{% highlight text %}
##         X1      X2
## X1 1.00000 0.78396
## X2 0.78396 1.00000
{% endhighlight %}
 
Dormann (2013) found that 'coefficients between predictor variables of r > 0.7 was an appropriate indicator for when collinearity begins to severely distort model estimation'.
 
 
#### Variance Inflation Factors

{% highlight r %}
require(car)
vif(mod)
{% endhighlight %}



{% highlight text %}
##     X1     X2 
## 2.5947 2.5947
{% endhighlight %}
 
Which is equivalent to (for variable X1):

{% highlight r %}
sum <- summary(lm(X1 ~ X2, data = df1))
1 / (1 - sum$r.squared)
{% endhighlight %}



{% highlight text %}
## [1] 2.5947
{% endhighlight %}
 
Unfortunately there are many 'rules of thumb' associated with VIF: > 10, > 4, ...
 
 
 
 
 
 
### Simulation 1: How does collinearity affect the precision of estimates? 
 
Here we simulate datasets with correlations ranging from -0.8 to 0.8:

{% highlight r %}
ps <- seq(-0.8, 0.8, 0.005)
n = 1000
 
sim1 <- lapply(ps, function(x) datagen(n=n, p=x))
{% endhighlight %}
 
 
Next we fit a linear model to each of the datasets and extract the coefficients table:

{% highlight r %}
# Function to fit and extract coefficients
regfun <- function(x){
  return(summary(lm(y~X1+X2, data = x))$coefficients)
}
# run regfun on every dataset
res1 <- lapply(sim1, regfun)
{% endhighlight %}
 
Finally we extract the Standard Errors and plot them against the degree of correlation:

{% highlight r %}
# extract Standard Errors from results
ses <- data.frame(ps, t(sapply(res1, function(x) x[,2])))
 
# plot
require(ggplot2)
require(reshape2)
ses_m <- melt(ses, id.vars='ps')
ggplot(ses_m, aes(x = ps, y = value)) +
  geom_point(size = 3) +
  facet_wrap(~variable) +
  theme_bw() +
  ylab('Std. Error') +
  xlab('Correlation')
{% endhighlight %}

![plot of chunk plot1](/figures/plot1-1.png) 
 
It can be clearly seen, that collinearity inflates the Standard Errors for the correlated variables. The intercept is not affected.
 
Having large standard errors parameter estimates are also variable, which is demonstrated in the next simulation:
 
 
### Simulation 2: Are estimated parameters stable under collinearity?
 
Here I simulate for different correlations 100 datasets each in order to see how stable the estimates are. 
 
The code creates the datasets, fits a linear model to each dataset and extracts the estimates.
 

{% highlight r %}
n_sims <- 100
ps <- seq(-0.8, 0.8, 0.05)
 
# function to generate n_sims datasets, for give p
sim2 <- function(p){
  sim <- lapply(1:n_sims, function(x) datagen(n=1000, p=p))
  res <- lapply(sim, regfun)
  est <- t(sapply(res, function(x) x[,1]))
  out <- data.frame(p, est)
  return(out)
}
 
res2 <- lapply(ps, function(x) {
  return(sim2(x))
})
{% endhighlight %}
 
 
The we create a plot of the estimates for the three coefficients, each boxplot represents 100 datasets.
 
 

{% highlight r %}
ests <- lapply(res2, melt, id.vars = 'p')
ests <- do.call(rbind, ests)
 
ggplot(ests, aes(y = value, x=factor(p))) +
  geom_boxplot() +
  facet_wrap(~variable, scales='free_y') +
  theme_bw() +
  geom_hline(aes(yintercept = 7), data = subset(ests, variable %in% c("X1","X2")), 
             col = 'red', lwd = 1) + 
  ylab('Estimate')+
  xlab('Correlation')
{% endhighlight %}

![plot of chunk plot2](/figures/plot2-1.png) 
The red line indicates the coefficients after the data has been generated (7 * X1 and 7 * X2). We see that the spread of estimates increases as correlation increases.
 
 
This is confirmed by looking at the standard deviation of the estimates:

{% highlight r %}
sds <- data.frame(ps, t(sapply(res2, function(x) apply(x[, 2:4], 2, sd))))
require(reshape2)
sds_m <- melt(sds, id.vars='ps')
ggplot(sds_m, aes(x = ps, y = value)) +
  geom_point(size = 3) +
  facet_wrap(~variable) +
  theme_bw() +
  ylab('SD of estimates from 100 simulations') +
  xlab('Correlation')
{% endhighlight %}

![plot of chunk plot3](/figures/plot3-1.png) 
 
 
If the standard errors are large enough it may happen that parameter estimates may be so variable that even their sign is changed.
 
 
 
 
 
 
