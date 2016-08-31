Utah Recidivism Model
================

About This Model
----------------

The following code is to analyze data and make ballpark predictions about how much a reduction in recidivism could potentially save the State of Utah. We are specifically interested in parolees with co-occuring disorders (COD).

### Assumptions

The greatest challenge in making predictions about the costs and benefits of the COD population is that the Justice Reinvestment Initiative (JRI) is old enough to affect outcomes in areas like recidivism, prison terms, crime frequencies, and parole sentences, but not old enough to collect reliable data on those effects. Therefore, I am forced to make several assumptions about the likely effects of the JRI on these outcomes. I will try to list some of the important ones below, but there are other assumptions throughout this notebook.

``` r
# A 10% reduction in recidivism 
recid_reduction_JRI <- .10

# One big change is that offenders can get what's called earned time credits while incarcerated, 
# which can also impact their length of stay (reduce it).
# A 20% reduction in mean time served seems reasonable
time_served_scale_JRI <- .8
```

About the data
--------------

The two main datasets that inform this model are, \* Prison Terms, which contain crime frequencies, their estimated average prison sentence, and an estimate of the direct taxpayer costs associated with that crime \* Survival Rates, which are the kaplan-meier curves from national data, extrapolated to fit Utah's trends

### Prison Terms

I pieced this data together using a an email from Julie Christensen, which contained the offense frequencies for our COD demographic, and various reports from the CCJJ. For the "frequency" column, I took the "frequency\_of\_crime" and added parole violations, which used to account for 70% of returns. In the model, "frequency" is used when the person is on parole, and "frequency\_of\_crime" is used when they are not. The [Justice Reinvestment Report](http://justice.utah.gov/Documents/CCJJ/Reports/Justice_Reinvestment_Report_2014.pdf) and [ACLU Report](http://www.acluutah.org/criminal-justice/item/download/15_0cfccf37c91e9fb16be4ac2e89ca12f2) contain the mean time served for various offenses, which I scale down 20% (as described above). Finally, the [Cost of Crime Report](http://www.justice.utah.gov/Documents/CCJJ/Cost%20of%20Crime/Utah%20Cost%20of%20Crime%202012%20-%20Methods%20Review%20Cost.pdf) contains estimates of the taxpayer costs for most categories (extrapolated for parole violations).

``` r
prison_terms <- read.csv("./clean_data/prison_terms.csv", stringsAsFactors = FALSE) %>% 
  mutate(mean_time_served = round(mean_time_served))

# I do these so that I can add a blank in the function below
prison_terms[8,1] <- ""

# replcace NAs with 0
prison_terms$frequency[is.na(prison_terms$frequency)] <- 0
prison_terms$frequency_of_crime[is.na(prison_terms$frequency_of_crime)] <- 0

# Scale down time served to reflect (guess)timated effects of the JRI
prison_terms$mean_time_served <- prison_terms$mean_time_served * time_served_scale_JRI

# Another big change from the JRI is that prison sentences for technical violations are capped
# So we change the 15-month mean to a 2-month one
# page 39: http://www.utah.gov/pmn/files/172049.pdf
prison_terms$mean_time_served[prison_terms$offense_type == "Parole_Violation"] <- 2


head(prison_terms, 10)
```

| offense\_type     |  mean\_time\_served|  frequency\_of\_crime|  frequency| assumptions                                |  court\_cost|  police\_cost|
|:------------------|-------------------:|---------------------:|----------:|:-------------------------------------------|------------:|-------------:|
| Murder            |               167.2|                0.0167|  0.0049223| NA                                         |        62037|          4509|
| Person            |                49.6|                0.2381|  0.0701802| NA                                         |         5443|          4509|
| Sex               |                58.4|                0.0667|  0.0196599| NA                                         |         5443|          4509|
| Property          |                18.4|                0.3143|  0.0926402| NA                                         |         2284|           880|
| Drug              |                14.4|                0.2214|  0.0652579| includes posession                         |         2284|           880|
| Other             |                13.6|                0.1428|  0.0420904| weapons and driving and other              |         2284|           880|
| Parole\_Violation |                 2.0|                0.0000|  0.7050000| court and police costs similar to property |         2284|           880|
|                   |                  NA|                0.0000|  0.0000000| NA                                         |           NA|            NA|

Including Plots
---------------

You can also embed plots, for example:

![](Utah_recidivism_model_files/figure-markdown_github/pressure-1.png)

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
