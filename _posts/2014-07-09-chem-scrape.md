---
layout: post
title: "Web scraping chemical data with R"
date: 2014-07-09 19:38
author: Eduard Szöcs
published: true
status: publish
draft: false
tags: R web-scraping
---




**Update:**
These functions have been integrated into the [webchem package](https://github.com/ropensci/webchem) and the functions removed from the esmisc package!


I recently came across the problem to convert [CAS numbers](http://en.wikipedia.org/wiki/CAS_registry_number) into [SMILES](Simplified molecular-input line-entry system) and retrieve additional information about the compound.

The are several sources around the web that provide chemical informations, eg.
[PubChem](https://pubchem.ncbi.nlm.nih.gov/), [ChemSpider](www.chemspider.com) and the [Chemical Identifier Resolver](http://cactus.nci.nih.gov/chemical/structure).

I wrote up some functions to interact from R with these servers. You can find them in my [esmisc](https://github.com/EDiLD/esmisc) package:


{% highlight r %}
install.packages('devtools')
require(devtools)
install_github('esmisc', 'EDiLD')
require(esmisc)
{% endhighlight %}



These functions are very crude and need some further development (if you want to improve, fork the package!), however, here's a short summary:

### Covert CAS to SMILES
Suppose we have some CAS numbers and want to convert them to SMILES:

{% highlight r %}
casnr <- c("107-06-2", "107-13-1", "319-86-8")
{% endhighlight %}

##### Via Cactus

{% highlight r %}
cactus(casnr, output = 'smiles')
{% endhighlight %}

##### Via ChemSpider
Note, that ChemSpider requires a security token. To obtain a token please [register](http://www.chemspider.com/Register.aspx) at ChemSpider. 



{% highlight r %}
csid <- get_csid(casnr, token = token)
csid_to_smiles(csid, token)
{% endhighlight %}

##### Via PubChem

{% highlight r %}
cid <- get_cid(casnr)
cid_to_smiles(cid)
{% endhighlight %}



### Retrieve other data from CAS

All these web resources provide additional data. Here is an example retrieving the molecular weights:

##### Via cactus

{% highlight r %}
cactus(casnr, output = 'mw')
{% endhighlight %}

##### Via ChemSpider

{% highlight r %}
csid_to_ext(csid, token)$MolecularWeight
{% endhighlight %}

##### Via Pubchem

{% highlight r %}
cid_to_ext(cid)$mw
{% endhighlight %}
ChemSpider and PubChem return the same values, however the results from cactus are slightly different.


### Retrieve partitioning coefficients
Partition coefficients are another useful property. [LOGKOW](http://logkow.cisti.nrc.ca/logkow/intro.html) is a databank that contains experimental data, retrieved from the literature, on over 20,000 organic compounds. 

`get_kow()` extracts the 'Recommended values' for a given CAS:


{% highlight r %}
get_kow(casnr)
{% endhighlight %}

This function is very crude. For example, it returns only the first hit if multiple hits are found in the database - a better way would be to ask for user input, as we did the [taxize](https://github.com/ropensci/taxize) package.


### Outlook

Currently I have no time to extensively develop these functions. 
I would be happy if someone picks up this work - it's [fairly easy](https://help.github.com/articles/fork-a-repo#contributing-to-a-project): just fork the repo and start.

In future this could be turned into a [ROpenSci](http://ropensci.org/) package as it is within their scope.
