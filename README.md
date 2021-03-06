# bayesLifeHIV

[![Travis-CI Build Status](https://travis-ci.org/PPgp/bayesLifeHIV.svg?branch=master)](https://travis-ci.org/PPgp/bayesLifeHIV)

Extension of the **bayesLife** R package that takes into account HIV/AIDS, as described in Godwin and Raftery (2017). It requires **bayesLife** version 4.0-2 or higher.

### Installation

You can either install it the traditional way of cloning GitHub repositories and installing the packages from local directories. Or you can use the **devtools** package as shown below.

* Install **bayesLifeHIV** from GitHub:

	```
	library(devtools)
	install_github("PPgP/bayesLifeHIV")
	```
	

### Usage

```
library(bayesLifeHIV)
```
Since **bayesLifeHIV** shares (and overwrites) some of the global objects of **bayesLife**, it is recommended not to mix simulations from **bayesLifeHIV** and **bayesLife**  in the same session. Or, if you work on objects created with **bayesLifeHIV**, start with 

```
using.bayesLifeHIV()
```

When you switch to a **bayesLife** simulation, do 

```
using.bayesLife()
```

The two functions above reset the global settings to the defaults of the respective package.
 
#### MCMC estimation

Here is an example of a toy simulation:

```
sim.dir <- "e0simHIV"
m <- run.e0hiv.mcmc(nr.chains = 2, iter = 50, thin = 1, 
			output.dir = sim.dir, replace.output = TRUE,
			verbose = TRUE)

```
The function above estimates MCMCs for ALL countries, non-epidemic as  well as epidemic. 
Countries considered as epidemic in the estimation can be viewed via

```
hiv.countries.est(m$meta)
```

Summaries, DL curves etc. can be viewed as with ususal **bayesLife** objects, see ``?bayesLife``. E.g. 

```
e0.partraces.plot(m)
```

#### Projections

To generate predictions for all countries for the toy simulation above, run the following command:
 
```
pred <- e0hiv.predict(sim.dir = sim.dir, burnin = 10, 
			replace.output=TRUE)
```
Again, this generates predictions for non-epidemic as well as epidemic countries. 
Countries considered as epidemic in the prediction can be viewed via

```
hiv.countries.pred(m$meta)
```

Summaries, trajectory plots, maps etc. can be viewed as with ususal **bayesLife** objects, see ``?bayesLife``. E.g.


```
e0.map.gvis(pred)
```

#### MCMC settings

In this version of **bayesLife** (version 4.0 and higher) and thus **bayesLifeHIV**, the ``run.e0.mcmc()`` and ``run.e0hiv.mcmc()`` functions were made cleaner in a way that many of the arguments were moved into global options. They can be viewed using 

```
e0mcmc.options()
```

These options are reset by ``using.bayesLifeHIV()`` (HIV setup) and ``using.bayesLife()`` (normal setup). 

To change some of these values, one can use the  ``e0mcmc.options()`` command prior to ``run.e0.mcmc()``. E.g. to change the upper limit of the ``Triangle``'s prior, do 

```
e0mcmc.options(Triangle = list(
					prior.up = c(110, 110, 110, 110))
				)
```

The run function stores the current options into the mcmc object (``meta$mcmcm.options``). Thus, if we estimate with the above settings, we can see that the resulting object contains the changes:

```
sim.dir <- "e0simHIVopttest1"
m1 <- run.e0hiv.mcmc(nr.chains = 1, iter = 10, thin = 1, 
              output.dir = sim.dir, replace.output = TRUE,
              verbose = TRUE)
m1$meta$mcmc.options$Triangle$prior.up
# compare to the original mcmc object 
m$meta$mcmc.options$Triangle$prior.up
```

Another way of changing the setting is to pass it directly into the run function, namely in the ``mcmc.options`` argument. Let's now change the lower bound of ``Triangle``:

```
sim.dir <- "e0simHIVopttest2"
m2 <- run.e0hiv.mcmc(nr.chains = 1, iter = 10, thin = 1, 
              output.dir = sim.dir, replace.output = TRUE,
              mcmc.options = list(Triangle = list(
                         prior.low = c(5, 5, 0, 5))),
              verbose = TRUE)
m2$meta$mcmc.options$Triangle$prior.low
``` 

The difference is that this time the global options were not changed, see ``e0mcmc.options("Triangle")``.

### Required Datasets

The package requires three additional datasets that are beyond of what is needed in **bayesLife**. Currently, they are included in the package in  the ``data`` directory. 
All files are tab delimited, with the suffix ".txt"
 
#### Estimation

  * **HIVprevalence**: contains historical and projected HIV prevalence for countries with past or/and future epidemics. Currently it has 58 rows and 29 columns, corresponding to 58 countries and time periods from 1970 to 2100 (note that columns for future years are only used for scaling, see below). It has also a column "include\_code". Only countries are considered as epidemic in the estimation, if their "include\_code" is 1. View the dataset via
  
    ```
    data(HIVprevalence)
    head(HIVprevalence)
    ```
    The dataset can be overwritten by passing the name of the new file as the argument ``my.hiv.file`` in the ``run.e0hiv.mcmc()`` function.
    
  * **ARTcoverage**: contains historical and projected values of ART coverage for countries with past or/and future epidemics. It has the same structure as HIVprevalence, including the "include\_code" column. Countries considered as epidemic must have their "include\_code" set to 1 in both datasets, HIVprevalence and ARTcoverage. By default, all 58 countries are considered as epidemic. Note that in the estimation, only columns for historical time periods are used, while future time periods are used in the projections. View the dataset via
  
    ```
    data(ARTcoverage)
    head(ARTcoverage)
    ```
    The dataset can be overwritten by passing the name of the new file as the argument ``my.art.file`` in the ``run.e0hiv.mcmc()`` function.

During the estimation, these two datasets are merged and converted into a delta(nonART) dataset, which is delta(hiv * (100 - art)/100), containing all countries. It can be accessed from an mcmc object, e.g. for an object ``m``:

```
m$meta$dlt.nart[, 1:10]
```

#### Projections

  * **ARTcoverage**: as mentioned above, columns that correspond to future years are used in the projections. The dataset is again filtered using the "include\_code" column. The default ``ARTcoverage`` can be overwritten by passing the name of the new file as the argument ``my.art.file`` in the ``e0hiv.predict()`` function.

  * **HIVprevTrajectories**: these are probabilistic trajectories of HIV prevalence for countries considered as epidemic in the future. It should have columns "country\_code", "Trajectory", and time periods "2010-2015", ..., "2095-2100". The default dataset contains 1000 trajectories for 58 countries (scaled to the HIVprevalence dataset, see below), thus resulting in a dimension of 58,000 x 20. View the dataset by 
  
    ```
    data(HIVprevTrajectories)
    head(HIVprevTrajectories)
    ``` 
    The dataset can be overwritten by passing the name of the new file as the argument ``my.hivtraj.file`` in the ``e0hiv.predict()`` function. It has to have the same format as the default dataset. If scaling to the HIVprevalence dataset is desired on the user-defined data, set the argument ``scale.hivtraj`` to ``TRUE``.
    
    
Similarly to the estimation, these two datasets are merged and converted into a dataset of delta(nonART) trajectories, which are then used in the prediction step.

By default, countries considered as epidemic in the projection are those that have "include\_code" set to 3 in the ``include_{wpp}`` dataset in the **bayesLife** package. This can be overwritten by the argument ``hiv.countries`` in ``e0hiv.predict()``, which should be a vector of country codes. However, in both cases, such countries are required to have records in both, the ARTcoverage and HIVprevTrajectories datasets.

##### Trajectories Scaling 

We have scaled our original HIV trajectories so that the median for each country aligns with the values in the **HIVprevalence** dataset (columns corresponding to future time periods). The **HIVprevTrajectories** is a result of such scaling. 

The scaling (which uses adjusted logit) is implemented in the function ``scale.hiv.trajectories()``. One can pass a dataset of trajectories (in the same format as HIVprevTrajectories) and a dataset of one time series per country (in the same format as HIVprevalence). Alternatively, it can be performed on the fly within the prediction function ``e0hiv.predict`` by setting the argument ``scale.hivtraj`` to ``TRUE``. Argument ``scale.hivtraj.tofile`` in ``e0hiv.predict`` can be used to pass the file name containing the dataset to be scaled to. The HIVprevalence dataset is used as the default.

### Note

This package is still under construction, especially the documentation needs some more work. We apologize for the inconvenience. 

### References

[Godwin, J. and Raftery, A.E. (2017): Bayesian projection of life expectancy accounting for the HIV/AIDS epidemic. Demographic Research 37(48):1549–1610.](http://www.demographic-research.org/Volumes/Vol37/48)
