---
layout: post
title: "Quantitative Ecotoxicology, Page 223, Example 5.1, Using GLM"
date: 2015-01-19 22:08
author: Eduard Szöcs
published: true
status: published
draft: false
tags: QETXR R
---
 


 
[Previously](http://edild.github.io/quant-ecotox20/), I showed how to analyse the fish survival data using the arcsine square root transformation.
Warton & Hui (2011) demonstrated that *the arcsine transform should not be used in either circumstance*, but instead use Generalized Linear Models.
This post is about how to analyse example 5.1 on page 223 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) using Generalised Linear Models.
 
 
### Introduction
 
Data of this type (*x out of n*) can be directly modelled using a [binomial distribution](http://en.wikipedia.org/wiki/Binomial_distribution).
 
<!--more-->
 
The binomial distribution is described by two parameters:
 
$$Bin(N, \pi)$$
 
where N is the number of fish and $\pi$ is the probability of survival.
 
 
Here I plotted the binomial distribution for ten fish at different probabilities ($\pi = 0.5, 0.75, 0.95, 0.05$), to give you a feeling how this distribution looks like at difference $\pi$:
 

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
  geom_bar(stat = 'identity', position = "dodge") +
  labs(x = expression(pi), y = expression(p(pi))) +
  scale_fill_discrete("Distribution", 
                    labels = c("Bin(10, 0.5)", "Bin(10, 0.75)", "Bin(10, 0.95)", "Bin(10, 0.05)")) +
  theme_bw()
{% endhighlight %}

![plot of chunk plot_binomial](/figures/plot_binomial-1.png) 
 
 
 
### The Model
 
We can now build a model, where the probability of success is a function of treatment:
 
$$  y \sim Bin(N, \pi) \\
  logit~(\pi) = \alpha + \beta x \\
  var(y) =  \pi (1 - \pi) / N $$
  
  
Let's explain this a little bit:
 
$$ y \sim Bin(N, \pi) $$, basically says: We assume that the number of dead fish ($y$) binomially distributed, where N = exposed animals and $\pi$ i is the probability of survival, which together give the expected number of surviving fish ($E(y) = N \times \pi$).
 
$$ logit~(\pi) = \alpha + \beta x $$, basically says: We are modelling the probability of survival as function of treatment (x) [note the right-hand side of the formula is similar to linear regression]. However, we need to ensure that $$0 < \pi < 1$$, and therefore the relationship is on the logit scale. The estimated parameters ($\beta) are directly interpretable as changes in log odds between treatments.
 
$$ var(y) =  \pi (1 - \pi) / N $$, says that variance of the binomial distribution
is a quadratic function of the mean. Note, that in linear regression we assume a constant variance.
 
 
 
### How to do it in R?
 
First I read again the data into R and prepare it:
 

{% highlight r %}
df <- read.table(header = TRUE, text = 'conc A B C D
0 1 1 0.9 0.9
32 0.8 0.8 1 0.8
64 0.9 1 1 1 
128 0.9 0.9 0.8 1
256 0.7 0.9 1 0.5
512 0.4 0.3 0.4 0.2')
require(reshape2)
dfm <- melt(df, id.vars = 'conc', value.name = 'y', variable.name = 'tank')
dfm$conc <- factor(dfm$conc)
{% endhighlight %}
 
 
The compute a binomial GLM in R we can use the `glm()` function:
 

{% highlight r %}
modglm <- glm(y ~ conc , family = binomial(link = 'logit'), data = dfm, 
              weights = rep(10, nrow(dfm)))
{% endhighlight %}
 
Note, that I use the `weights` argument to specify that in each tank we had 10 fish (N in the above formulas).
The summary gives the estimated parameters:
 

{% highlight r %}
summary(modglm)
{% endhighlight %}



{% highlight text %}
## 
## Call:
## glm(formula = y ~ conc, family = binomial(link = "logit"), data = dfm, 
##     weights = rep(10, nrow(dfm)))
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -1.898  -0.572   0.000   0.787   2.258  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept)    2.944      0.725    4.06  4.9e-05 ***
## conc32        -1.210      0.850   -1.42    0.155    
## conc64         0.719      1.246    0.58    0.564    
## conc128       -0.747      0.897   -0.83    0.405    
## conc256       -1.708      0.818   -2.09    0.037 *  
## conc512       -3.675      0.800   -4.59  4.4e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 88.672  on 23  degrees of freedom
## Residual deviance: 23.889  on 18  degrees of freedom
## AIC: 72.86
## 
## Number of Fisher Scoring iterations: 5
{% endhighlight %}
 
The estimates for `(Intercept)` are the log odds to survive in the treatment:
 
 

{% highlight r %}
# mean proportion surviving in control
p_0 <- mean(dfm$y[dfm$conc == 0])
# log odds
(logodd_0 <- log(p_0 / (1 - p_0)))
{% endhighlight %}



{% highlight text %}
## [1] 2.9444
{% endhighlight %}
 
Similar the log odds for the highest treatment:

{% highlight r %}
# mean proportion surviving in control
p_512 <- mean(dfm$y[dfm$conc == 512])
# log odds
(logodd_512 <- log(p_512 / (1 - p_512)))
{% endhighlight %}



{% highlight text %}
## [1] -0.73089
{% endhighlight %}
 
The estimate for `conc512` (-3.675) gives you the difference in the log odds, as can be seen here:

{% highlight r %}
logodd_512 - logodd_0
{% endhighlight %}



{% highlight text %}
## [1] -3.6753
{% endhighlight %}
 
Note, that this kind of interpretation is not possible with the arcsine transformation.
 
#### Hypothesis tests
Similar to the previous post we can perform an F-Test:
 

{% highlight r %}
drop1(modglm, test = 'F')
{% endhighlight %}



{% highlight text %}
## Warning in drop1.glm(modglm, test = "F"): F test assumes 'quasibinomial'
## family
{% endhighlight %}



{% highlight text %}
## Single term deletions
## 
## Model:
## y ~ conc
##        Df Deviance   AIC F value  Pr(>F)    
## <none>        23.9  72.9                    
## conc    5     88.7 127.6    9.76 0.00012 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
{% endhighlight %}
 
Or do multiple comparisons:
 

{% highlight r %}
require(multcomp)
summary(glht(modglm, linfct = mcp(conc = 'Dunnett')))
{% endhighlight %}



{% highlight text %}
## 
## 	 Simultaneous Tests for General Linear Hypotheses
## 
## Multiple Comparisons of Means: Dunnett Contrasts
## 
## 
## Fit: glm(formula = y ~ conc, family = binomial(link = "logit"), data = dfm, 
##     weights = rep(10, nrow(dfm)))
## 
## Linear Hypotheses:
##              Estimate Std. Error z value Pr(>|z|)    
## 32 - 0 == 0    -1.210      0.850   -1.42     0.40    
## 64 - 0 == 0     0.719      1.246    0.58     0.94    
## 128 - 0 == 0   -0.747      0.897   -0.83     0.81    
## 256 - 0 == 0   -1.708      0.818   -2.09     0.12    
## 512 - 0 == 0   -3.675      0.800   -4.59   <0.001 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## (Adjusted p values reported -- single-step method)
{% endhighlight %}
 
 
 
 
### Which one is better?
I would say GLMs! 
They have greater power (Warton & Hui, 2011), are simpler to interpret and are readily available in most software packages.
 
 
#### References
 
* Warton, D. I., & Hui, F. K. (2011). The arcsine is asinine: the analysis of proportions in ecology. Ecology, 92(1), 3-10.
