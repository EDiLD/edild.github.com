---
layout: post
title: "Web scraping pesticides data with R"
date: 2014-11-08 11:56
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: R web-scraping ecotoxicology
---



**Update:**
The function allanwood() has been integrated into the [webchem package](https://github.com/ropensci/webchem) and the function removed from the esmisc package!


This week I had to find the [CAS-numbers](http://en.wikipedia.org/wiki/CAS_Registry_Number) for a bunch of pesticides. 
Moreover, I also needed information about the major groups of these pesticides (e.g. herbicides, fungicides, ...) and some of them were in German language.

[ETOX](http://webetox.uba.de/webETOX/index.do) is quite useful to find the CAS-numbers, even for German names, as they have also synonyms in their database.

Another, useful source is the [Compendium of Pesticide Common Names](http://www.alanwood.net/pesticides/index.html) by Allan Wood.

Since I had > 500 compounds in my list, this was feasible to be done manually. 
So I wrote two small functions (`etox_to_cas()` and `allanwood()`) to search and retrieve information from these two websites.

Both are available from my [esmisc](https://github.com/EDiLD/esmisc#cas-from-etox) package on github.com.
These are small functions using the `RCurl` and `XML` packages for scraping. 
They have not been tested very much and may not be very robust.


#### Query CAS from ETOX 

{% highlight r %}
require(esmisc)
etox_to_cas('2,4-D')
{% endhighlight %}



{% highlight text %}
## [1] "88-85-7"
{% endhighlight %}

If you have a bunch of compounds you can use 'sapply()' to feed `etox_to_cas`:

{% highlight r %}
sapply(c('2,4-D', 'DDT', 'Diclopfop'), etox_to_cas)
{% endhighlight %}



{% highlight text %}
##     2,4-D       DDT Diclopfop 
## "88-85-7" "50-29-3"        NA
{% endhighlight %}


#### Query CAS and pesticide group from Allan Wood

**Update:**
The function allanwood() has been integrated into the [webchem package](https://github.com/ropensci/webchem) and the function removed from the esmisc package!


{% highlight r %}
allanwood('Fluazinam')
sapply(c('Fluazinam', 'Diclofop', 'DDT'), allanwood)
{% endhighlight %}





