---
layout: post
title: "Quantitative Ecotoxicology, Page 223, Example 5.1"
date: 2014-08-25 12:10
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR R
---



This is example 5.1 on page 223 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - reproduced with R. 


#### Load and clean the data
Read the data into R:

{% highlight r %}
df <- read.table(header = TRUE, text = 'conc A B C D
0 1 1 0.9 0.9
32 0.8 0.8 1 0.8
64 0.9 1 1 1 
128 0.9 0.9 0.8 1
256 0.7 0.9 1 0.5
512 0.4 0.3 0.4 0.2')
df
{% endhighlight %}



{% highlight text %}
##   conc   A   B   C   D
## 1    0 1.0 1.0 0.9 0.9
## 2   32 0.8 0.8 1.0 0.8
## 3   64 0.9 1.0 1.0 1.0
## 4  128 0.9 0.9 0.8 1.0
## 5  256 0.7 0.9 1.0 0.5
## 6  512 0.4 0.3 0.4 0.2
{% endhighlight %}

Next we do some house-keeping: convert the data to long format and the concentration to factor.

{% highlight r %}
require(reshape2)
# to long
dfm <- melt(df, id.vars = 'conc', value.name = 'y', variable.name = 'tank')
# conc as factor
dfm$conc <- factor(dfm$conc)
head(dfm)
{% endhighlight %}



{% highlight text %}
##   conc tank   y
## 1    0    A 1.0
## 2   32    A 0.8
## 3   64    A 0.9
## 4  128    A 0.9
## 5  256    A 0.7
## 6  512    A 0.4
{% endhighlight %}

Let's have first look at the data:

{% highlight r %}
boxplot(y ~ conc, data = dfm, 
        xlab = 'conc', ylab = 'Proportion surv.')
{% endhighlight %}

![plot of chunk unnamed-chunk-4](../figures/source/2014-08-25-quant-ecotox20/unnamed-chunk-4-1.png) 

#### Transform response
Next we apply the arcsine transformation:

{% highlight r %}
dfm$y_asin <- ifelse(dfm$y == 1, 
                     asin(sqrt(dfm$y)) - asin(sqrt(1/40)), 
                     asin(sqrt(dfm$y)) 
                     )
{% endhighlight %}
This adds the transformed values as column `y_asin` to our data.frame. 
Survivals of 1 (100%) are transformed 

$$arcsin(sqrt(y)) - arcsin(\sqrt\frac{1}{4n}) =$$ 

$$arcsin(1) - arcsin(\sqrt\frac{1}{4 \cdot 10}) = $$

$$1.5708 - 0.1588 = $$

$$ 1.412$$

, where n = number of animals per replicate.

All other values are transformed using

$$ arcsin(\sqrt y) $$

If we would have had observations with 0% survival, these zero values should be transformed using

$$ arcsin(\sqrt\frac{1}{4n}) $$

by adding an extra `ifelse()`.

Let's look at the transformed values:

{% highlight r %}
head(dfm)
{% endhighlight %}



{% highlight text %}
##   conc tank   y  y_asin
## 1    0    A 1.0 1.41202
## 2   32    A 0.8 1.10715
## 3   64    A 0.9 1.24905
## 4  128    A 0.9 1.24905
## 5  256    A 0.7 0.99116
## 6  512    A 0.4 0.68472
{% endhighlight %}



{% highlight r %}
boxplot(y_asin ~ conc, data = dfm, 
        xlab = 'conc', ylab = 'Transformed prop. surv.')
{% endhighlight %}

![plot of chunk unnamed-chunk-6](../figures/source/2014-08-25-quant-ecotox20/unnamed-chunk-6-1.png) 

Doesn't look that different...

#### ANOVA
To fit a ANOVA to his data we use the `aov()` function:

{% highlight r %}
mod <- aov(y_asin ~ conc, data = dfm)
{% endhighlight %}

And `summary()` gives the anova table:

{% highlight r %}
summary(mod)
{% endhighlight %}



{% highlight text %}
##             Df Sum Sq Mean Sq F value   Pr(>F)    
## conc         5  1.575  0.3151    13.3 0.000016 ***
## Residuals   18  0.426  0.0237                     
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
{% endhighlight %}
The within-treatment variance is termed *Residuals* and the between-treatment variance is named according to the predictor *conc*. The total variance is simply the sum of those and not displayed.

R already performed an F-test for us, indicated by the *F value* (=ratio of the *Mean Sq*) and *Pr (>F)* columns. 


#### Multiple Comparisons
Now we know that there is a statistically significant treatment effect, we might be interested which treatments differ from the control group. 

The in the book mentioned Tukey contrasts (comparing each level with each other) can be easily done with the `multcomp` package:


{% highlight r %}
require(multcomp)
summary(glht(mod, linfct = mcp(conc = 'Tukey')))
{% endhighlight %}



{% highlight text %}
## 
## 	 Simultaneous Tests for General Linear Hypotheses
## 
## Multiple Comparisons of Means: Tukey Contrasts
## 
## 
## Fit: aov(formula = y_asin ~ conc, data = dfm)
## 
## Linear Hypotheses:
##                Estimate Std. Error t value Pr(>|t|)    
## 32 - 0 == 0     -0.1472     0.1088   -1.35   0.7530    
## 64 - 0 == 0      0.0407     0.1088    0.37   0.9989    
## 128 - 0 == 0    -0.0762     0.1088   -0.70   0.9795    
## 256 - 0 == 0    -0.2211     0.1088   -2.03   0.3633    
## 512 - 0 == 0    -0.7273     0.1088   -6.69   <0.001 ***
## 64 - 32 == 0     0.1879     0.1088    1.73   0.5326    
## 128 - 32 == 0    0.0709     0.1088    0.65   0.9850    
## 256 - 32 == 0   -0.0740     0.1088   -0.68   0.9820    
## 512 - 32 == 0   -0.5802     0.1088   -5.33   <0.001 ***
## 128 - 64 == 0   -0.1170     0.1088   -1.08   0.8849    
## 256 - 64 == 0   -0.2619     0.1088   -2.41   0.2054    
## 512 - 64 == 0   -0.7681     0.1088   -7.06   <0.001 ***
## 256 - 128 == 0  -0.1449     0.1088   -1.33   0.7643    
## 512 - 128 == 0  -0.6511     0.1088   -5.98   <0.001 ***
## 512 - 256 == 0  -0.5062     0.1088   -4.65   0.0024 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## (Adjusted p values reported -- single-step method)
{% endhighlight %}

However, this leads to 15 comparisons (and tests) and we may not be interested in all. Note that we are wrong in 1 out of 20 tests ($\alpha = 0.05$) (if we do not apply correction for multiple testing). 

An alternative would be just to compare the control group to the treatments. This is called *Dunnett contrasts* and leads to only 5 comparison.

The syntax is the same, just change Tukey to Dunnett:

{% highlight r %}
summary(glht(mod, linfct = mcp(conc = 'Dunnett')))
{% endhighlight %}



{% highlight text %}
## 
## 	 Simultaneous Tests for General Linear Hypotheses
## 
## Multiple Comparisons of Means: Dunnett Contrasts
## 
## 
## Fit: aov(formula = y_asin ~ conc, data = dfm)
## 
## Linear Hypotheses:
##              Estimate Std. Error t value Pr(>|t|)    
## 32 - 0 == 0   -0.1472     0.1088   -1.35     0.54    
## 64 - 0 == 0    0.0407     0.1088    0.37     0.99    
## 128 - 0 == 0  -0.0762     0.1088   -0.70     0.93    
## 256 - 0 == 0  -0.2211     0.1088   -2.03     0.20    
## 512 - 0 == 0  -0.7273     0.1088   -6.69   <0.001 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## (Adjusted p values reported -- single-step method)
{% endhighlight %}

The column *Estimate* gives use the difference in means between the control and the respective treatments and *Std. Error* the standard error from these estimates. Both are combined to a *t value* (=Estimate / Std. Error), from which we can get a p-value (*P(>t)*).

Note, that the p-values are already corrected for multiple testing, as indicated at the bottom of the output. 
If you want to change the correction method you can use:

{% highlight r %}
summary(glht(mod, linfct = mcp(conc = 'Dunnett')), test = adjusted('bonferroni'))
{% endhighlight %}

This applies Bonferroni-correction, see `?p.adjust` and `?adjusted` for other methods.

#### Outlook
Warton & Hui (2011) demonstrated that *the arcsine transform should not be used in either circumstance*. Similarly as O’Hara & Kotze (2010) showed that count data should not be log-transformed. 

I a future post I will show how to analyse this data without transformation using **Generalized Linear Models (GLM)** and perhabs some simulations showing that using GLM can lead to an increased statistical power for ecotoxicological data sets.

Note, that I couldn't find any reference to Generalized Linear Models in Newman (2012) and EPA (2002), although they have been around for 30 years now (Nelder & Wedderburn, 1972).

#### References

* Warton, D. I., & Hui, F. K. (2011). The arcsine is asinine: the analysis of proportions in ecology. Ecology, 92(1), 3-10.

* O’Hara, R. B., & Kotze, D. J. (2010). Do not log‐transform count data. Methods in Ecology and Evolution, 1(2), 118-122.

* Newman, M. C. (2012). Quantitative ecotoxicology. Taylor & Francis, Boca Raton, FL.

* EPA (2002). Methods for Measuring the Acute Toxicity of Effluents and Receiving Waters to Freshwater and Marine Organisms. 

* Nelder J.A., & Wedderburn R.W.M. (1972). Generalized Linear Models. Journal of the Royal Statistical Society Series A (General) 135:370–384. 

