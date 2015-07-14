---
layout: post
title: "Dispersion-based weighting - a new function in vegan"
date: 2013-07-12 11:00
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: R vegan
---

### Intro

When analysing multivariate community data the first step is often to transform the abundance data, in order to downweight contributions of high abundance taxa to dissimilarity. There are a lot of transformations available, for example:

* log(x + 1), but see [O’Hara &amp; Kotze (2010)](http://dx.doi.org/10.1111/j.2041-210X.2010.00021.x)
* ln(2x + 1), which is used often with mesocosm data
* $x^{0.5}$
* $x^{0.25}$
* Presence/Absence
* and manymany more

However these are all global transformations (applied to every species in the dataset).

In an email discussion with Bob Clarke he pointed me to one of his papers [Clarke et al. 2006](http://dx.doi.org/10.3354/meps320011):

>Clarke, K. R., M. G. Chapman, P. J. Somerfield, and H. R. Needham. 2006. “Dispersion-based Weighting of Species Counts in Assemblage Analyses.” Marine Ecology. Progress Series 320: 11–27. [klick](http://www.int-res.com/abstracts/meps/v320/p11-27/)

I somehow missed this paper and it looks like this hasn't been implemented in R yet. 
So I wrote it up for myself and the function has been merged into vegan this week (see [Changelog](https://raw.github.com/jarioksa/vegan/master/inst/ChangeLog)).

### Get it
Currently the function is only available in the dev-version of vegan. You can it easily install it from [github](https://github.com/jarioksa/vegan) using the devtools-package. You can also take a look at the implementation [here](https://github.com/jarioksa/vegan/blob/master/R/dispweight.R).


{% highlight r %}
require(devtools)
install_github('vegan', 'jarioksa')
require(vegan)
{% endhighlight %}


The new function is named `dispweight` and takes three arguments:


{% highlight r %}
dispweight(comm, group, nsimul = 1000)
{% endhighlight %}

The community data matrix, a vector describing the group structure and the number of permutations to use for the permutation test.

It returns a list of four elements:

* D: The dispersion index
* p: p-value of the permutation test
* weights: The weights applied to species
* transformed: The transformed abundance matrix


### Use it

Here it take the dune dataset and `Management` as grouping variable.

First we take look at the data via NMDS:


{% highlight r %}
require(vegan)
data(dune)
data(dune.env)

# NMDS on fouth-root transformed abundances
un_nmds <- metaMDS(dune^0.25, distance = 'bray')

# plot it
cols = rainbow(4)
plot(un_nmds, display = 'sites', type = 'n')
points(un_nmds, col = cols[dune.env$Management], pch = 16, cex = 1.5)
ordihull(un_nmds, groups = dune.env$Management, lty = 'dotted')
ordispider(un_nmds, groups = dune.env$Management, label = TRUE)
{% endhighlight %}

![plot of chunk plot_raw](../figures/source/2013-06-09-disp-weight/plot_raw-1.png) 


Lets run the NMDS on dispersion-weighted abundances:

{% highlight r %}
# calculate weights
dpw <- dispweight(dune, dune.env$Management, nsimul = 100)

# NMDS on dispersion weighted community data
disp_nmds <- metaMDS(dpw, distance = 'bray')

# plot
plot(disp_nmds, display = 'sites', type = 'n')
points(disp_nmds, col = cols[dune.env$Management], pch = 16, cex = 1.5)
ordihull(disp_nmds, groups = dune.env$Management, lty = 'dotted')
ordispider(disp_nmds, groups = dune.env$Management, label = TRUE)
{% endhighlight %}

![plot of chunk plot_disp](../figures/source/2013-06-09-disp-weight/plot_disp-1.png) 

In this example there is not a big difference, but as they write in their paper:

>[...] we should not expect a dispersion-weighting approach to radically alter ordination and test results. Indeed, the improvements shown in the real examples of this paper are sometimes small and subtle [...].

I haven't tested the function very much, so let me know if you find any bugs! Note that the code isn't optimised for speed and follows largely the paper.

### Refs

<ul>
<li>KR Clarke, MG Chapman, PJ Somerfield, HR Needham,   (2006) Dispersion-Based Weighting of Species Counts in Assemblage Analyses.  <em>Marine Ecology Progress Series</em>  <strong>320</strong>  11-27  <a href="http://dx.doi.org/10.3354/meps320011">10.3354/meps320011</a></li>
<li>Robert B. O’Hara, D. Johan Kotze,   (2010) do Not Log-Transform Count Data.  <em>Methods in Ecology And Evolution</em>  <strong>1</strong>  118-122  <a href="http://dx.doi.org/10.1111/j.2041-210X.2010.00021.x">10.1111/j.2041-210X.2010.00021.x</a></li>
</ul>
