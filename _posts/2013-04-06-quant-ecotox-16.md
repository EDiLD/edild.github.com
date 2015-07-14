---
layout: post
title: "Quantitative Ecotoxicology, Page 178, Example 4.9, Time-to-death"
date: 2013-04-06 23:08
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR R
---




This is example 4.9 on page 178 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - time-to-death data.

Thankfully, Prof. Newman provided me the data for this example. You can get it from the github-repo ([TOXICTY.csv](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/TOXICITY.csv)).


{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/TOXICITY.csv",
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
TOXICITY <- read.table(text = url, header = TRUE)
head(TOXICITY)
{% endhighlight %}



{% highlight text %}
##   TTD TANK  PPT WETWT STDLGTH
## 1   8    1 15.8 0.112     1.9
## 2   8    1 15.8 0.050     1.5
## 3   8    1 15.8 0.029     1.2
## 4   8    1 15.8 0.045     1.4
## 5   8    2 15.8 0.097     1.8
## 6   8    2 15.8 0.048     1.4
{% endhighlight %}



{% highlight r %}
summary(TOXICITY)
{% endhighlight %}



{% highlight text %}
##       TTD            TANK           PPT           WETWT      
##  Min.   : 8.0   Min.   : 1.0   Min.   : 0.0   Min.   :0.024  
##  1st Qu.:24.0   1st Qu.: 4.0   1st Qu.:10.3   1st Qu.:0.068  
##  Median :72.0   Median : 7.0   Median :11.6   Median :0.092  
##  Mean   :63.3   Mean   : 7.5   Mean   :11.7   Mean   :0.135  
##  3rd Qu.:97.0   3rd Qu.:11.0   3rd Qu.:15.8   3rd Qu.:0.129  
##  Max.   :97.0   Max.   :14.0   Max.   :20.1   Max.   :1.489  
##                                               NA's   :70     
##     STDLGTH    
##  Min.   :1.20  
##  1st Qu.:1.60  
##  Median :1.70  
##  Mean   :1.87  
##  3rd Qu.:2.00  
##  Max.   :4.90  
##  NA's   :70
{% endhighlight %}

The data consists of 5 columns:

* TTD   :     Time to death
* TANK  :     Tank
* PPT   :     NaCl Concentration
* WETWT :     wet weight
* STDLGTH :   Standard length

Columns 4 and 5 have 70 NA's (no data available due to measurement error), but we won't use these in this example. The observations with TTD = 97 are 'survivors', since the experiment run only 96 hours.


First we need to create a column `FLAG` for the status of the animal (dead/alive):

{% highlight r %}
TOXICITY$FLAG <- ifelse(TOXICITY$TTD > 96, 1, 2)
{% endhighlight %}
So 1 denotes alive and 2 dead.

Then we can plot the data. Each line is a tank and colors denote the NaCl concentrations.

{% highlight r %}
require(survival)
mod <- survfit(Surv(TTD, FLAG) ~ PPT + strata(TANK), data = TOXICITY)
plot(mod, col = rep(1:7, each=2), mark.time=FALSE, xlab = 'Hours', ylab = '% Survival')
legend('bottomleft', legend = sort(unique(TOXICITY$PPT)), col=1:7, lty = 1)
{% endhighlight %}

![plot of chunk plot_surv](../figures/source/2013-04-06-quant-ecotox-16/plot_surv-1.png) 

We see a clear relationship between concentration and the survival curves. In  this example we are interested in differences between the duplicates. We see that the two curves for the 11.6 g/L concentration are quite similar, while there is more divergence between tanks in the 13.2 g/L treatment.

We can test for differences using the `survdiff` function. With the `rho` argument we can specify the type of test: `rho = 0` is a log-rank test and `rho = 1` is equivalent to the Peto & Peto modification of the Gehan-Wilcoxon test.


First the log-rank test for each concentration:

{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==10.3, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     10.3, ], rho = 0)
## 
##          N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=9  40        6     9.02      1.01      2.22
## TANK=10 38       11     7.98      1.14      2.22
## 
##  Chisq= 2.2  on 1 degrees of freedom, p= 0.136
{% endhighlight %}



{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==10.8, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     10.8, ], rho = 0)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=7 41       10     11.6     0.226     0.503
## TANK=8 39       12     10.4     0.253     0.503
## 
##  Chisq= 0.5  on 1 degrees of freedom, p= 0.478
{% endhighlight %}



{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==11.6, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     11.6, ], rho = 0)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=5 37       20     18.9    0.0623     0.129
## TANK=6 41       20     21.1    0.0559     0.129
## 
##  Chisq= 0.1  on 1 degrees of freedom, p= 0.719
{% endhighlight %}



{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==13.2, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     13.2, ], rho = 0)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=3 38       34     40.6      1.06       3.1
## TANK=4 40       37     30.4      1.41       3.1
## 
##  Chisq= 3.1  on 1 degrees of freedom, p= 0.0781
{% endhighlight %}



{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==15.8, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     15.8, ], rho = 0)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=1 39       39     44.5     0.674      3.09
## TANK=2 40       40     34.5     0.868      3.09
## 
##  Chisq= 3.1  on 1 degrees of freedom, p= 0.0789
{% endhighlight %}


We could also run this in a `for` loop (here the Wilcoxon test):

{% highlight r %}
for(i in sort(unique(TOXICITY$PPT)[-c(2,7)])) {
  cat('\n', i, '\n')
  print(survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==i, ], rho = 1))
}
{% endhighlight %}



{% highlight text %}
## 
##  10.3 
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     i, ], rho = 1)
## 
##          N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=9  40     5.41     8.21     0.952      2.28
## TANK=10 38    10.09     7.29     1.071      2.28
## 
##  Chisq= 2.3  on 1 degrees of freedom, p= 0.131 
## 
##  10.8 
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     i, ], rho = 1)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=7 41     8.75    10.33     0.240     0.596
## TANK=8 39    10.81     9.24     0.269     0.596
## 
##  Chisq= 0.6  on 1 degrees of freedom, p= 0.44 
## 
##  11.6 
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     i, ], rho = 1)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=5 37     15.4     14.8    0.0205    0.0524
## TANK=6 41     15.9     16.5    0.0184    0.0524
## 
##  Chisq= 0.1  on 1 degrees of freedom, p= 0.819 
## 
##  13.2 
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     i, ], rho = 1)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=3 38     17.8     23.5      1.37      5.02
## TANK=4 40     24.9     19.2      1.68      5.02
## 
##  Chisq= 5  on 1 degrees of freedom, p= 0.0251 
## 
##  15.8 
## Call:
## survdiff(formula = Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT == 
##     i, ], rho = 1)
## 
##         N Observed Expected (O-E)^2/E (O-E)^2/V
## TANK=1 39     22.6     27.0     0.707      3.15
## TANK=2 40     27.8     23.4     0.814      3.15
## 
##  Chisq= 3.1  on 1 degrees of freedom, p= 0.076
{% endhighlight %}

Basically we get the same results as in the book: 

None of the log-rank tests is statistically significant (at the 0.05 level).
The wilcoxon test for the 13.2 g/L treatment shows a p < 0.05. 
This is also in agreement with the plot.

The $\chi^2$ values differ slightly but share the same trend - I suspect this is due to different data used.

With this dataset we can do much more. We already saw that there might be a relationship between survival time and concentration, but more on this later (example 4.10).

Code and data are available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p176'.



