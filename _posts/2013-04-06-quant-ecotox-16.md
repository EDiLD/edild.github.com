---
layout: post
title: "Quantitative Ecotoxicology, Page 178, Example 4.9, Time-to-death"
date: 2013-04-06 23:08
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: QETXR, R
---
 


 
This is example 4.9 on page 178 of [Quantitative Ecotoxicology](http://www.crcpress.com/product/isbn/9781439835647) - time-to-death data.
 
Thankfully, Prof. Newman provided me the data for this example. You can get it from the github-repo ([TOXICTY.csv](https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/TOXICITY.csv)).
 

{% highlight r %}
require(RCurl)
url <- getURL("https://raw.github.com/EDiLD/r-ed/master/quantitative_ecotoxicology/data/TOXICITY.csv",
ssl.verifypeer = FALSE, .opts=curlOptions(followlocation=TRUE))
{% endhighlight %}



{% highlight text %}
## Error in function (type, msg, asError = TRUE) : Could not resolve host: raw.github.com
{% endhighlight %}



{% highlight r %}
TOXICITY <- read.table(text = url, header = TRUE)
head(TOXICITY)
{% endhighlight %}



{% highlight text %}
##   DEAD X..TOTAL...CONC.
## 1   16    ;"76";"10.30"
## 2   22    ;"79";"10.80"
## 3   40    ;"77";"11.60"
## 4   69    ;"76";"13.20"
## 5   78    ;"78";"15.80"
## 6   77    ;"77";"20.10"
{% endhighlight %}



{% highlight r %}
summary(TOXICITY)
{% endhighlight %}



{% highlight text %}
##       DEAD           X..TOTAL...CONC.
##  Min.   :16.0   ;"76";"10.30":1      
##  1st Qu.:26.5   ;"76";"13.20":1      
##  Median :54.5   ;"77";"11.60":1      
##  Mean   :50.3   ;"77";"20.10":1      
##  3rd Qu.:75.0   ;"78";"15.80":1      
##  Max.   :78.0   ;"79";"10.80":1
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



{% highlight text %}
## Error in `$<-.data.frame`(`*tmp*`, "FLAG", value = logical(0)): replacement has 0 rows, data has 6
{% endhighlight %}
So 1 denotes alive and 2 dead.
 
Then we can plot the data. Each line is a tank and colors denote the NaCl concentrations.

{% highlight r %}
require(survival)
mod <- survfit(Surv(TTD, FLAG) ~ PPT + strata(TANK), data = TOXICITY)
{% endhighlight %}



{% highlight text %}
## Error in Surv(TTD, FLAG): object 'TTD' not found
{% endhighlight %}



{% highlight r %}
plot(mod, col = rep(1:7, each=2), mark.time=FALSE, xlab = 'Hours', ylab = '% Survival')
{% endhighlight %}



{% highlight text %}
## Warning in plot.window(...): "mark.time" is not a graphical parameter
{% endhighlight %}



{% highlight text %}
## Warning in plot.xy(xy, type, ...): "mark.time" is not a graphical parameter
{% endhighlight %}



{% highlight text %}
## Warning in box(...): "mark.time" is not a graphical parameter
{% endhighlight %}



{% highlight text %}
## Warning in title(...): "mark.time" is not a graphical parameter
{% endhighlight %}



{% highlight text %}
## Warning in plot.xy(xy.coords(x, y), type = type, ...): "mark.time" is not a
## graphical parameter
{% endhighlight %}

![plot of chunk plot_surv](/figures/plot_surv-1.png) 

{% highlight r %}
legend('bottomleft', legend = sort(unique(TOXICITY$PPT)), col=1:7, lty = 1)
{% endhighlight %}



{% highlight text %}
## Warning in is.na(x): is.na() applied to non-(list or vector) of type 'NULL'
{% endhighlight %}



{% highlight text %}
## Error in legend("bottomleft", legend = sort(unique(TOXICITY$PPT)), col = 1:7, : 'legend' is of length 0
{% endhighlight %}
 
We see a clear relationship between concentration and the survival curves. In  this example we are interested in differences between the duplicates. We see that the two curves for the 11.6 g/L concentration are quite similar, while there is more divergence between tanks in the 13.2 g/L treatment.
 
We can test for differences using the `survdiff` function. With the `rho` argument we can specify the type of test: `rho = 0` is a log-rank test and `rho = 1` is equivalent to the Peto & Peto modification of the Gehan-Wilcoxon test.
 
 
First the log-rank test for each concentration:

{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==10.3, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Error in Surv(TTD, FLAG): object 'TTD' not found
{% endhighlight %}



{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==10.8, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Error in Surv(TTD, FLAG): object 'TTD' not found
{% endhighlight %}



{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==11.6, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Error in Surv(TTD, FLAG): object 'TTD' not found
{% endhighlight %}



{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==13.2, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Error in Surv(TTD, FLAG): object 'TTD' not found
{% endhighlight %}



{% highlight r %}
survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==15.8, ], rho = 0)
{% endhighlight %}



{% highlight text %}
## Error in Surv(TTD, FLAG): object 'TTD' not found
{% endhighlight %}
 
 
We could also run this in a `for` loop (here the Wilcoxon test):

{% highlight r %}
for(i in sort(unique(TOXICITY$PPT)[-c(2,7)])) {
  cat('\n', i, '\n')
  print(survdiff(Surv(TTD, FLAG) ~ TANK, data = TOXICITY[TOXICITY$PPT==i, ], rho = 1))
}
{% endhighlight %}



{% highlight text %}
## Warning in is.na(x): is.na() applied to non-(list or vector) of type 'NULL'
{% endhighlight %}
 
Basically we get the same results as in the book: 
 
None of the log-rank tests is statistically significant (at the 0.05 level).
The wilcoxon test for the 13.2 g/L treatment shows a p < 0.05. 
This is also in agreement with the plot.
 
The $\chi^2$ values differ slightly but share the same trend - I suspect this is due to different data used.
 
With this dataset we can do much more. We already saw that there might be a relationship between survival time and concentration, but more on this later (example 4.10).
 
Code and data are available on my [github-repo](https://github.com/EDiLD/r-ed/tree/master/quantitative_ecotoxicology) under file name 'p176'.
 
 
 
