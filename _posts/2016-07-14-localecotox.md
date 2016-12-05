---
title: "Build a local version of the EPA ECOTOX database"
author: "Eduard Szöcs"
date: "2016-8-14 20:00"
layout: post
published: true
status: published
tags: R ecotoxicology
draft: false
---
<img src="http://vg03.met.vgwort.de/na/cfbef2b8a1d64eca9d08698ed03ae1b4" width="1" height="1" alt="">
 

 

 
## Introduction
 
Databases of toxicity data (like EC50 of a species towards a chemical) play an important role in ecotoxicology: Their data is used to construct [SSDs](http://edild.github.io/ssd/), to calculate Toxic Units (TU) or for prior information to experimental design.
 
Several such databases are available, maintained by different institutions and with a possible overlap.
The most comprehensive database (with more than 600,000 tests) may be the [US EPA ECOTOX database](https://cfpub.epa.gov/ecotox/).
Although you can query the ECOTOX database via the web interface, this is limited by the number of rows, your internet connection and maybe you need a more customized query.
Luckily, the US EPA provides all the data as download [(here)](https://cfpub.epa.gov/ecotox/data_download.cfm) and also updates it regularly (every 3 months).
 
In this post I will describe how you can build a local (= on your own computer / server) version of the US EPA ECOTOX database. 
To follow this post you will need 
 
1. basic R knowledge - I write everything as an R script because later on, I will also use R to analyse this data.
2. basic SQL knowledge - I will store the data in a PostgreSQL database. Basically, this is all SQL (wrapped into R)
 
 
 
 
 
## Preliminaries
 
1) Download the US EPA ECOTOX data from [here](https://cfpub.epa.gov/ecotox/data_download.cfm).
2) Execute the .exe file (=Unzip the data to a folder). This will result in a folder (`ecotox_ascii_06_15_2016`) with the following contents:
 

{% highlight r %}
├── AsciiDownloadHelp5.pdf
├── chemical_carriers.txt
├── dose_response_details.txt
├── dose_response_links.txt
├── dose_responses.txt
├── doses.txt
├── ECOTOX_ASCII_dd.html
├── media_characteristics.txt
├── release_notes_06_15_2016.txt
├── results.txt
├── tests.txt
└── validation
  |── <lots of txt files>
{% endhighlight %}
 
 
The unpacked folder contains documentation of the downloaded data (`AsciiDownloadHelp5.pdf` and `ECOTOX_ASCII_dd.html`),
release notes (`release_notes_06_15_2016.txt`) and the data tables:
 
1. `chemical_carriers.txt` : Information pertaining to the carrier and/or positive control chemicals
reported for the test.
2. `dose_response_details.txt`: Detail dose response records
3. `dose_response_links.txt`: Ties dose response to its endpoint
4. `dose_responses.txt`: Parent dose response record containing sample size, effect measurement,
response site, observation duration etc.
5. `doses.txt`: Information pertaining to the dose-response dose.
6. `media_characteristics.txt`: Water chemistry and media characteristics parameters.
7. `results.txt` : Information pertaining to the endpoint or non-endpoint result or dose-
response summary
8. `tests.txt`: Information pertaining to the experimental design.
 
The subfolder `~/validation` contains lookup tables. 
For more information see the documentation files.
 
 
3. Get a running PostgreSQL instance: 
 
I use PostgreSQL as database, but the following can be easily rewritten to other database systems.
You can install a local database on your computer or a server, see [here](http://www.postgresqltutorial.com/install-postgresql/) or [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04) for installation tutorials.
Create a database named `'ecotox'`.
 
 
## Copy the data
 
I will copy all downloaded tables to the database.
For this, I will use the `RPostgreSQL` package (which saves me writting up all the `CREATE TABLE` statements...).
 
First we specify the path to the downloaded and extracted folder (`ecotox_ascii_06_15_2016`)

{% highlight r %}
datadir <- '/home/edisz/Downloads/ecotox_ascii_06_15_2016/'
{% endhighlight %}
 
Next we set the server details:

{% highlight r %}
require(RPostgreSQL)
DBname <- 'ecotox'
DBhost <- '<yourIP>'   # or localhost)
DBport <- '<yourPort>' # default is 5432
DBuser <- '<yourUsername>'
{% endhighlight %}
 
And connect to the database from R:
 

{% highlight r %}
drv <- dbDriver("PostgreSQL")
con <- dbConnect(drv, user = DBuser, 
                 dbname = DBname, 
                 host = DBhost, 
                 port = DBport)
{% endhighlight %}
 
Now we are ready to copy the data:
 
### Data tables
First the 8 data tables.

{% highlight r %}
# list all .txt files
files <- list.files(datadir, pattern = "*.txt", full.names = TRUE)
# exlcude the release notes
files <- files[!grepl('release', files)]
# extract the file/table names
names <- gsub(".txt", "", basename(files))
# for every file, read into R and copy to postgresql
for(i in seq_along(files)){
  message("Read File: ", files[i], "\n")
  df <- read.table(files[i], header = TRUE, sep = '|', comment.char = '', 
    quote = '')
  dbWriteTable(con, names[i], value = df, row.names = FALSE)
}
{% endhighlight %}
 
This and the next steps may take some time... 
Now that we have the tables in our database, I add primary keys to them (see also documentation):
 

{% highlight r %}
# Add primary keys
dbSendQuery(con, "ALTER TABLE chemical_carriers ADD PRIMARY KEY (carrier_id)")
dbSendQuery(con, "ALTER TABLE dose_response_details ADD PRIMARY KEY (dose_resp_detail_id)")
dbSendQuery(con, "ALTER TABLE dose_response_links ADD PRIMARY KEY (dose_resp_link_id)")
dbSendQuery(con, "ALTER TABLE dose_responses ADD PRIMARY KEY (dose_resp_id)")
dbSendQuery(con, "ALTER TABLE doses ADD PRIMARY KEY (dose_id)")
dbSendQuery(con, "ALTER TABLE media_characteristics ADD PRIMARY KEY (result_id)")
dbSendQuery(con, "ALTER TABLE results ADD PRIMARY KEY (result_id)")
dbSendQuery(con, "ALTER TABLE tests ADD PRIMARY KEY (test_id)")
{% endhighlight %}
 
For better performance of queries I also add (lots of) indexes to columns that may be used for joins or filters:
 

{% highlight r %}
# add indexes
dbSendQuery(con, "CREATE INDEX idx_chemical_carriers_test_id ON chemical_carriers(test_id)")
dbSendQuery(con, "CREATE INDEX idx_chemical_carriers_cas ON chemical_carriers(cas_number)")
dbSendQuery(con, "CREATE INDEX idx_dose_response_details_dose_resp_id ON dose_response_details(dose_resp_id)")
dbSendQuery(con, "CREATE INDEX idx_dose_response_details_dose_id ON dose_response_details(dose_id)")
dbSendQuery(con, "CREATE INDEX idx_dose_response_links_results_id ON dose_response_links(result_id)")
dbSendQuery(con, "CREATE INDEX idx_dose_response_links_dose_resp ON dose_response_links(dose_resp_id)")
dbSendQuery(con, "CREATE INDEX idx_dose_responses_test_id ON dose_responses(test_id)")
dbSendQuery(con, "CREATE INDEX idx_dose_responses_effect_code ON dose_responses(effect_code)")
dbSendQuery(con, "CREATE INDEX idx_doses_test_id ON doses(test_id)")
dbSendQuery(con, "CREATE INDEX idx_results_endpoint ON results(endpoint)")
dbSendQuery(con, "CREATE INDEX idx_results_test_id ON results(test_id)")
dbSendQuery(con, "CREATE INDEX idx_results_effect ON results(effect)")
dbSendQuery(con, "CREATE INDEX idx_results_conc1_unit ON results(conc1_unit)")
dbSendQuery(con, "CREATE INDEX idx_results_obs_duration_mean ON results(obs_duration_mean)")
dbSendQuery(con, "CREATE INDEX idx_results_obs_duration_unit ON results(obs_duration_unit)")
dbSendQuery(con, "CREATE INDEX idx_results_measurement ON results(measurement)")
dbSendQuery(con, "CREATE INDEX idx_results_response_site ON results(response_site)")
dbSendQuery(con, "CREATE INDEX idx_results_chem_analysis ON results(chem_analysis_method)")
dbSendQuery(con, "CREATE INDEX idx_results_significance_code ON results(significance_code)")
dbSendQuery(con, "CREATE INDEX idx_results_trend ON results(trend)")
dbSendQuery(con, "CREATE INDEX idx_test_cas ON tests(test_cas)")
dbSendQuery(con, "CREATE INDEX idx_test_species_number ON tests(species_number)")
dbSendQuery(con, "CREATE INDEX idx_test_media_type ON tests(media_type)")
dbSendQuery(con, "CREATE INDEX idx_test_location ON tests(test_location)")
dbSendQuery(con, "CREATE INDEX idx_test_type ON tests(test_type)")
dbSendQuery(con, "CREATE INDEX idx_test_study_type ON tests(study_type)")
dbSendQuery(con, "CREATE INDEX idx_test_method ON tests(test_method)")
dbSendQuery(con, "CREATE INDEX idx_test_lifestage ON tests(organism_lifestage)")
dbSendQuery(con, "CREATE INDEX idx_test_gender ON tests(organism_gender)")
dbSendQuery(con, "CREATE INDEX idx_test_source ON tests(organism_source)")
dbSendQuery(con, "CREATE INDEX idx_test_exposure ON tests(exposure_type)")
dbSendQuery(con, "CREATE INDEX idx_test_application_freq_unit ON tests(application_freq_unit)")
dbSendQuery(con, "CREATE INDEX idx_test_application_type ON tests(application_type)")
{% endhighlight %}
 
All tables are copied to the `'public'` SCHEMA.
However, I prefer to keep the data in schemas other than public and move all tables to a schema named `'ecotox'`
 

{% highlight r %}
# move to ecotox schema
dbSendQuery(con, "CREATE SCHEMA ecotox;")
for (i in names) {
  q <- paste0("ALTER TABLE ", i, " SET SCHEMA ecotox")
  dbSendQuery(con, q)
}
{% endhighlight %}
 
 
 
 
### Validation tables
Now the lookup tables in the `/validation` folder. 
The workflow is the same as above...
 

{% highlight r %}
# Copy validation tables to server
files2 <- list.files(file.path(datadir, "validation"), pattern = "*.txt", 
  full.names = T)
names2 <- gsub(".txt", "", basename(files2))
for (i in seq_along(files2)) {
  message("Read File: ", files2[i], "\n")
  df <- read.table(files2[i], header = TRUE, sep = '|', comment.char = '', 
    quote = '')
  dbWriteTable(con, names2[i], value = df, row.names = FALSE)
}
 
# Add primary keys (some table without PK -> these are just lookup tables)
dbSendQuery(con, "ALTER TABLE chemicals ADD PRIMARY KEY (cas_number)")
dbSendQuery(con, "ALTER TABLE \"references\" ADD PRIMARY KEY (reference_number)")
dbSendQuery(con, "ALTER TABLE species ADD PRIMARY KEY (species_number)")
dbSendQuery(con, "ALTER TABLE species_synonyms ADD PRIMARY KEY (species_number, latin_name)")
dbSendQuery(con, "ALTER TABLE trend_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE application_type_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE application_frequency_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE exposure_type_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE chemical_analysis_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE organism_source_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE gender_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE lifestage_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE response_site_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE measurement_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE effect_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE test_method_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE field_study_type_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE test_type_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE test_location_codes ADD PRIMARY KEY (code)")
dbSendQuery(con, "ALTER TABLE media_type_codes ADD PRIMARY KEY (code)")
 
# Add indexes
dbSendQuery(con, "CREATE INDEX idx_species_latin ON species(latin_name)")
dbSendQuery(con, "CREATE INDEX idx_species_group ON species(ecotox_group)")
dbSendQuery(con, "CREATE INDEX idx_media_type ON media_type_codes(code)")
 
# change name and schema of references
dbSendQuery(con, 'ALTER TABLE public.\"references\" RENAME TO refs')
dbSendQuery(con, 'ALTER TABLE public.refs SET SCHEMA ecotox')
 
# move to ecotox schema
for (i in names2[!names2 %in% 'references']) {
  q <- paste0("ALTER TABLE ", i, " SET SCHEMA ecotox")
  dbSendQuery(con, q)
}
{% endhighlight %}
 
 
### Additional tables
 
For data-cleaning I also created custom lookup tables to convert durations and concentrations, as well as to unify groups.
You can download these tables [here](https://github.com/EDiLD/localecotox) (in the folder `/data/conversions`).
Use them at your own risk!
I have some more of these tables available (contact me if interested) and you can also easily create the for your needs.
 
We follow the same procedure for these files:

{% highlight r %}
lookuppath <- 'data/conversions/'   # adjust to your needs
files3 <- list.files(lookuppath, pattern = "*.csv$", 
                     full.names = T)
names3 <- gsub(".csv", "", basename(files3))
for (i in seq_along(files3)) {
  message("Read File: ", files3[i], "\n")
  df <- read.table(files3[i], header = TRUE, sep = ';')
  dbWriteTable(con, names3[i], value = df, row.names = FALSE)
  dbSendQuery(con, paste0('ALTER TABLE ', names3[i], ' SET SCHEMA ecotox'))
}
 
 
# add pk
dbSendQuery(con, "ALTER TABLE ecotox.ecotox_group_convert ADD PRIMARY KEY (ecotox_group)")
dbSendQuery(con, "ALTER TABLE ecotox.unit_convert ADD PRIMARY KEY (unit)")
dbSendQuery(con, "ALTER TABLE aquire.duration_convert ADD PRIMARY KEY (duration, unit)")
 
# add indexes
dbSendQuery(con, "CREATE INDEX idx_duration_convert_unit ON ecotox.duration_convert(unit)")
dbSendQuery(con, "CREATE INDEX idx_duration_convert_duration ON ecotox.duration_convert(duration)")
{% endhighlight %}
 
 
### Custom PostgreSQL Functions
 
Some concentrations are stored as text in the database, to convert to numeric values I provide you a with function to convert these to numerics [(see here)](https://stackoverflow.com/questions/10306830/postgres-define-a-default-value-for-cast-failures):
 

{% highlight r %}
dbSendQuery(con, "
CREATE OR REPLACE FUNCTION cast_to_num(text) 
RETURNS numeric AS 
            $$
            begin
            -- Note the double casting to avoid infinite recursion.
            RETURN cast($1::varchar AS numeric);
            exception
            WHEN invalid_text_representation THEN
            RETURN NULL;
            end;
            $$ 
            language plpgsql immutable;
            ")
 
dbSendQuery(con, "
            CREATE CAST (text as numeric) WITH FUNCTION cast_to_num(text);"
)
{% endhighlight %}
This function tries to convert to a numeric and if this is not possible returns `NULL`.
 
 
### Maintenance
Now all data of the US EPA ECOTOX database is in our database.
But before we quit the connection to do some maintenance:
 

{% highlight r %}
dbSendQuery(con, 'VACUUM ANALYZE')
# Close connection to DB
dbDisconnect(con)
dbUnloadDriver(drv)
{% endhighlight %}
 
 
## Action!
 
Let's see what we can do now with this database.
 
### Query 1: All EC50/LC50 data for Chlorpyrifos towards *D. magna*, irrespective of effect, test duration, etc...
 
The first query is a rather simple one: 
I query all LC50/EC50 values of *D.manga* for Chlorpyrifos (CAS#: 2921-88-2):
 

 

{% highlight r %}
drv <- dbDriver("PostgreSQL")
con <- dbConnect(drv, user = DBuser, 
                 dbname = DBname, 
                 host = DBhost, 
                 port = DBport)
 
res <- dbGetQuery(con, "
SELECT 
  species.latin_name,
-- test duration
  results.obs_duration_mean,
  results.obs_duration_unit,
  results.endpoint,
  results.effect,
-- type of concentration (e.g active ingredient, formulation etc.)
  results.conc1_type,
-- value of endpoint
  results.conc1_mean,
  results.conc1_unit
FROM 
-- main table
  ecotox.tests
-- results table
  LEFT JOIN ecotox.results ON tests.test_id = results.test_id
  RIGHT JOIN ecotox.species ON tests.species_number = species.species_number
WHERE 
    species.latin_name = 'Daphnia magna' 
-- CAS# of Chlorpyrifos
AND tests.test_cas = 2921882 
-- endpoints
AND results.endpoint IN ('EC50', 'EC50/', 'EC50*', 'EC50*/', 'LC50', 'LC50/', 'LC50*', 'LC50*/');
")
head(res, 10)
{% endhighlight %}



{% highlight text %}
##       latin_name obs_duration_mean obs_duration_unit endpoint effect
## 1  Daphnia magna                24                 h     LC50    MOR
## 2  Daphnia magna                96                 h     EC50    ITX
## 3  Daphnia magna                96                 h     EC50    ITX
## 4  Daphnia magna                21                 d     LC50    MOR
## 5  Daphnia magna                96                 h     EC50    ITX
## 6  Daphnia magna                21                 d     LC50    MOR
## 7  Daphnia magna                96                 h     EC50    ITX
## 8  Daphnia magna                24                 h     LC50    MOR
## 9  Daphnia magna                48                 h     LC50    MOR
## 10 Daphnia magna                48                 h     LC50    MOR
##    conc1_type conc1_mean conc1_unit
## 1           F        0.4       ug/L
## 2           A       0.58       ug/L
## 3           A       0.11       ug/L
## 4           A       0.13       ug/L
## 5           A       0.21       ug/L
## 6           A         NR       ug/L
## 7           A       0.22       ug/L
## 8           A        3.7       ug/L
## 9           A        1.0       ug/L
## 10          A        0.6       ug/L
{% endhighlight %}
 
 
We  get 41 entries, from different exposure times and endpoints.
There are many variables stored in the database! - 
Here I show only a small subset of test duration, endpoint (e.g. $EC_x$), effect (e.g. mortality, growth, etc),
concentration type (e.g active ingredient or formulation)...
 
 
We can further filter to allow only acute tests (48h duration) and allow only Mortality (MOR) or Intoxication (ITX) effects.
The first condition is AFAIK not possible with the web-interface of the US EPA.
 

{% highlight r %}
res <- dbGetQuery(con, "
SELECT 
  species.latin_name,
  results.obs_duration_mean,
  results.obs_duration_unit,
  results.endpoint,
  results.effect,
  results.conc1_type,
  results.conc1_mean,
  results.conc1_unit
FROM 
  ecotox.tests
  LEFT JOIN ecotox.results ON tests.test_id = results.test_id
  RIGHT JOIN ecotox.species ON tests.species_number = species.species_number
WHERE 
    species.latin_name = 'Daphnia magna' 
AND tests.test_cas = 2921882 
AND results.endpoint IN ('EC50', 'EC50/', 'EC50*', 'EC50*/', 'LC50', 'LC50/', 'LC50*', 'LC50*/')
-- 48 duration
AND results.obs_duration_mean = '48'
AND results.obs_duration_unit = 'h'
-- only ITX or MOR effects
AND results.effect IN ('MOR', 'ITX')
")
head(res, 10)
{% endhighlight %}



{% highlight text %}
##       latin_name obs_duration_mean obs_duration_unit endpoint effect
## 1  Daphnia magna                48                 h     LC50    MOR
## 2  Daphnia magna                48                 h     LC50    MOR
## 3  Daphnia magna                48                 h     LC50    MOR
## 4  Daphnia magna                48                 h     EC50    ITX
## 5  Daphnia magna                48                 h     EC50    MOR
## 6  Daphnia magna                48                 h     EC50    MOR
## 7  Daphnia magna                48                 h     EC50    ITX
## 8  Daphnia magna                48                 h     EC50    ITX
## 9  Daphnia magna                48                 h     EC50    MOR
## 10 Daphnia magna                48                 h     LC50    MOR
##    conc1_type conc1_mean conc1_unit
## 1           A        1.0       ug/L
## 2           A        0.6       ug/L
## 3           A      0.344       mg/L
## 4           A       1074        ppt
## 5           F      0.325       ug/L
## 6           F      0.344       ug/L
## 7           A       0.19    AI ug/L
## 8           A    0.00074       mg/L
## 9           A       0.25       ug/L
## 10          A      27.43       ug/L
{% endhighlight %}
 
Which reduces the data to 19 entries. 
You might note, that there are different units used. 
It might be desirable to harmonize these to ug/L.
 
We can do that using the lookup table `unit_convert` I provide.
Moreover, I add another condition to take only active ingredients (`results.conc1_type = 'A'`, see table `concentration_type_codes` for description of types).
 

{% highlight r %}
res <- dbGetQuery(con, "
SELECT 
  species.latin_name,
  results.obs_duration_mean,
  results.obs_duration_unit,
  results.endpoint,
  results.effect,
  results.conc1_type,
  -- convert concentration if possible
  CASE  
    WHEN unit_convert.convert = 'yes'
      THEN CAST(results.conc1_mean AS numeric) * unit_convert.multiplier
  END AS conc1_conv,
  CASE  
    WHEN unit_convert.convert = 'yes'
      THEN unit_convert.unit_conv
  END AS conc1_unit_conv,
  results.conc1_mean,
  results.conc1_unit
FROM 
  ecotox.tests
  LEFT JOIN ecotox.results ON tests.test_id = results.test_id
  RIGHT JOIN ecotox.species ON tests.species_number = species.species_number
-- unit conversion table
  LEFT JOIN ecotox.unit_convert ON results.conc1_unit = unit_convert.unit
WHERE 
    species.latin_name = 'Daphnia magna' 
AND tests.test_cas = 2921882 
AND results.endpoint IN ('EC50', 'EC50/', 'EC50*', 'EC50*/', 'LC50', 'LC50/', 'LC50*', 'LC50*/')
AND results.obs_duration_mean = '48'
AND results.obs_duration_unit = 'h'
AND results.effect IN ('MOR', 'ITX')
-- only active ingredients
AND results.conc1_type = 'A'
")
head(res, 10)
{% endhighlight %}



{% highlight text %}
##       latin_name obs_duration_mean obs_duration_unit endpoint effect
## 1  Daphnia magna                48                 h     EC50    ITX
## 2  Daphnia magna                48                 h     LC50    MOR
## 3  Daphnia magna                48                 h     LC50    MOR
## 4  Daphnia magna                48                 h     EC50    ITX
## 5  Daphnia magna                48                 h     LC50    MOR
## 6  Daphnia magna                48                 h     EC50    MOR
## 7  Daphnia magna                48                 h     EC50    ITX
## 8  Daphnia magna                48                 h     LC50    MOR
## 9  Daphnia magna                48                 h     EC50    ITX
## 10 Daphnia magna                48                 h     EC50    ITX
##    conc1_type conc1_conv conc1_unit_conv conc1_mean conc1_unit
## 1           A     1.0740            ug/L       1074        ppt
## 2           A    27.4300            ug/L      27.43       ug/L
## 3           A     0.6000            ug/L        0.6       ug/L
## 4           A     0.4800            ug/L       0.48       ug/L
## 5           A     1.2200            ug/L       1.22    AI ug/L
## 6           A     0.2500            ug/L       0.25       ug/L
## 7           A     1.7200            ug/L       1.72        ppb
## 8           A   344.0000            ug/L      0.344       mg/L
## 9           A     0.1000            ug/L        0.1        ppb
## 10          A     0.0324            ug/L       32.4       ng/L
{% endhighlight %}
 
 
You might notice that the queries can get quickly complicated (and SQL knowledge indispensable).
I have a custom query with several hundred lines of code...
 
I encourage everybody to explore the entries in this database and to understand their relationship and meaning.
It is a huge resource for ecotoxicologists! 
Here's a plot of the (log) distribution of EC50 values:
 

{% highlight r %}
hist(log10(res$conc1_conv))
{% endhighlight %}

![plot of chunk unnamed-chunk-16](/figures/unnamed-chunk-16-1.png)
 
We see that the values spread over several orders of magnitude...
 
 
 
 
### Query 2: All acute EC50/LC50 for Chlorpyrifos to build an SSD.
 
In a [previous post](https://edild.github.io/ssd/) I showed how to calculate Species Sensitivity Distribution (SSDs) using R.
However, I did not show how to retrieve the used data.
 
The following query is similar to the one above. However, I do not restrict to a specific taxon:
 
* all taxa
* Chlorpyrifos
* only EC50 or LC50
* only 48h tests
* only ITX or MOR effects
* keep only data points that can be converted to $\mu g / L$.
 

{% highlight r %}
res <- dbGetQuery(con, "
SELECT 
  species.latin_name,
  ecotox_group_convert.ecotox_group_convert,
  results.obs_duration_mean,
  results.obs_duration_unit,
  results.endpoint,
  results.effect,
  results.conc1_type,
  -- convert concentration if possible
  CASE  
    WHEN unit_convert.convert = 'yes'
      THEN CAST(results.conc1_mean AS numeric) * unit_convert.multiplier
  END AS conc1_conv,
  CASE  
    WHEN unit_convert.convert = 'yes'
      THEN unit_convert.unit_conv
  END AS conc1_unit_conv
FROM 
  ecotox.tests
  LEFT JOIN ecotox.results ON tests.test_id = results.test_id
  RIGHT JOIN ecotox.species ON tests.species_number = species.species_number
  LEFT JOIN ecotox.unit_convert ON results.conc1_unit = unit_convert.unit
  LEFT JOIN ecotox.ecotox_group_convert ON species.ecotox_group = ecotox_group_convert.ecotox_group
WHERE 
  tests.test_cas =  2921882
  AND results.endpoint IN ('EC50', 'EC50/', 'EC50*', 'EC50*/', 'LC50', 'LC50/', 'LC50*', 'LC50*/')
  AND results.obs_duration_mean IN ('48')
  AND results.obs_duration_unit = 'h'
  AND results.effect IN ('MOR', 'ITX')
-- only units that can be converted to ug/L
  AND unit_convert.convert = 'yes'
  AND unit_convert.unit_conv = 'ug/L'
  AND results.conc1_mean != 'NR'
")
head(res)
{% endhighlight %}



{% highlight text %}
##            latin_name ecotox_group_convert obs_duration_mean
## 1 Oncorhynchus mykiss                 fish                48
## 2  Ceriodaphnia dubia          crustaceans                48
## 3    Daphnia carinata          crustaceans                48
## 4   Molanna angustata              insects                48
## 5  Gammarus lacustris          crustaceans                48
## 6   Lepomis cyanellus                 fish                48
##   obs_duration_unit endpoint effect conc1_type conc1_conv conc1_unit_conv
## 1                 h     LC50    MOR          A     11.400            ug/L
## 2                 h     LC50    MOR          A      0.050            ug/L
## 3                 h     LC50    MOR          A      0.512            ug/L
## 4                 h     EC50    ITX          A      1.860            ug/L
## 5                 h     LC50    MOR          F      0.400            ug/L
## 6                 h     LC50    MOR          F     50.000            ug/L
{% endhighlight %}
 
I also added the taxonomic group (from a custom lookup-table).
We end up with 286 entries from 115 taxa.
So we have multiple entries per taxon.
For graphical display, I will aggregate them using the geometric mean.
 
[Note, that further data-quality and plausibility checks could and should (!) be done:
e.g. based on the solubility, baseline-tox etc...].
 
 

{% highlight r %}
res_agg <- dbGetQuery(con, "
SELECT 
  species.latin_name,
  ecotox_group_convert.ecotox_group_convert,
-- geometric mean
  exp(avg(ln(
    CAST(results.conc1_mean AS numeric) * unit_convert.multiplier
  ))) AS conc1_conv
FROM 
  ecotox.tests
  LEFT JOIN ecotox.results ON tests.test_id = results.test_id
  RIGHT JOIN ecotox.species ON tests.species_number = species.species_number
  LEFT JOIN ecotox.unit_convert ON results.conc1_unit = unit_convert.unit
  LEFT JOIN ecotox.ecotox_group_convert ON species.ecotox_group = ecotox_group_convert.ecotox_group
WHERE 
  tests.test_cas =  2921882
  AND results.endpoint IN ('EC50', 'EC50/', 'EC50*', 'EC50*/', 'LC50', 'LC50/', 'LC50*', 'LC50*/')
  AND results.obs_duration_mean IN ('48')
  AND results.obs_duration_unit = 'h'
  AND results.effect IN ('MOR', 'ITX')
  AND unit_convert.convert = 'yes'
  AND unit_convert.unit_conv = 'ug/L'
  AND results.conc1_mean != 'NR'
-- Aggregate by latin_name
GROUP BY latin_name, ecotox_group_convert
ORDER BY latin_name
")
head(res_agg, 10)
{% endhighlight %}



{% highlight text %}
##                    latin_name ecotox_group_convert conc1_conv
## 1          Alabama argillacea              insects  9337.6263
## 2              Anax imperator              insects     3.2090
## 3           Anguilla anguilla                 fish   690.0000
## 4             Anisops sardeus              insects     0.9000
## 5   Anopheles quadrimaculatus              insects     1.0000
## 6             Aphanius iberus                 fish    38.6000
## 7           Asellus aquaticus          crustaceans     4.4824
## 8      Atalophlebia australis              insects     0.2800
## 9     Brachionus calyciflorus        invertebrates 12000.0000
## 10 Bufo bufo ssp. gargarizans           amphibians  1170.0000
{% endhighlight %}
 
The aggregation could be also carried out in R:

{% highlight r %}
require(plyr)
res2 <- ddply(res, .(latin_name, ecotox_group_convert ), summarize,
      conc1_conv = exp(mean(log(conc1_conv))))
{% endhighlight %}
 
 
We are left with data from 115 taxa.
Let's create plot show the distribution of LC50 values across taxa:
 

{% highlight r %}
df <- res2[order(res2$conc1_conv), ]
df$frac <- ppoints(df$conc1_conv, 0.5)
library(ggplot2)
library(scales)
ggplot(data = df) +
  geom_point(aes(x = conc1_conv, y = frac, col = ecotox_group_convert), size = 3) +
  geom_text(aes(x = conc1_conv, y = frac, label = latin_name, col = ecotox_group_convert), hjust = 1.1, size = 2) +
  theme_bw() +
  scale_x_log10(limits = c(0.0075, max(df$conc1_conv)),
                breaks = c(0.01, 0.1, 1, 10, 100, 1000, 100000),
                labels = comma) +
  scale_color_brewer(palette = 'Dark2') +
  labs(x = expression(paste('Concentration of Chlorpyrifos [ ', mu, 'g ', L^-1, ' ]')), 
       y = 'Fraction of taxa affected')
{% endhighlight %}

![plot of chunk unnamed-chunk-20](/figures/unnamed-chunk-20-1.png)
 
We may see an ordering of sensitivity: fish < insects < crustaceans...
 
 
## Conclusions
 
This was a rather technical post, but dealing with (big)data in ecotoxicology requires some technical expertise in data wrangling.
Nevertheless, it may be useful to others working with this data from the US EPA ECOTOX database.
It is an incredibly important resource for ecotoxicology.
Thanks @USEPA for making this available!
 
Hopefully, I showed that it is worth having a local mirror of this database:
it gives much more flexibility than the web interface and you can get directly the data you need in R (where can further process it).
I must admit: Because the database is so huge it need some time to get familiar with it and to use it efficiently (what are the tables? What are the variables therein? What are the meaning of the codes?). 
 
I only showed two example queries here, but don't take them seriously!
A robust use of this database requires also a lot of quality checks! (I'll leave this for a future post...)
 
I have created few more custom lookup tables - if you are interested in these or have a question feel free to contact me.
