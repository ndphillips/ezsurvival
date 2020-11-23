
<!-- README.md is generated from README.Rmd. Please edit that file -->

# iwillsurvive 0.1.0.9000 <img src="https://cdn.iconscout.com/icon/free/png-512/disco-1-62616.png" align="right" height="139"/>

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![Codename:
shuffled](https://img.shields.io/badge/version-0.1_'Gloria'-yellow.svg)](https://en.wikipedia.org/wiki/Gloria_Gaynor)

The goal of `iwillsurvive` is to make it easy to fit and visualize
simple survival models. It provides wrapper functions around commonly
used functions from survival packages such as `survival::survfit()` and
`survminer::ggsurvplot()`, while providing user-friendly in-line
messages, notes, and warnings.

## Installation

`iwillsurvive` is hosted at
<https://github.com/ndphillips/iwillsurvive>. Here is how to install it:

``` r
devtools::install_github(repo = "https://github.com/ndphillips/iwillsurvive",
                         build_vignettes = TRUE)
```

## Example

``` r
library(iwillsurvive)
#> -----------------------------------------------------
#> iwillsurvive 0.1.0.9000 'Gloria'
#> Intro  : vignette('introduction', 'iwillsurvive')
#> Repo   : https://github.com/ndphillips/iwillsurvive
#> .....................................................
library(dplyr)
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union
```

It’s best to start with one-row-per-patient (ORPP) cohort object that
contains columns corresponding to

-   `patientid`, a unique patient identifier
-   `index_date`, a date corresponding to an index date.
-   `censor_date`, date corresponding to when patients were censored
-   `event_date`, date corresponding to the event of interest. NA values
    indicate that the event was not observed.

`iwillsurvive` provides one such example in `ez_cohort`, a dataframe of
250 simulated patients:

``` r
ez_cohort
#> # A tibble: 250 x 5
#>    patientid condition lotstartdate lastvisitdate dateofdeath
#>    <chr>     <chr>     <date>       <date>        <date>     
#>  1 F00001    placebo   2016-05-17   2020-12-01    NA         
#>  2 F00002    placebo   2020-07-27   2020-08-25    2020-10-05 
#>  3 F00003    drug      2016-04-14   2017-02-16    2017-03-13 
#>  4 F00004    drug      2020-06-12   2020-11-25    NA         
#>  5 F00005    placebo   2019-03-20   2020-01-13    2020-02-21 
#>  6 F00006    placebo   2017-04-02   2017-10-18    2017-11-19 
#>  7 F00007    placebo   2018-01-26   2019-01-12    2019-02-17 
#>  8 F00008    placebo   2015-07-02   2015-11-20    2015-12-23 
#>  9 F00009    drug      2019-03-08   2020-07-18    2020-08-17 
#> 10 F00010    placebo   2018-08-23   2019-02-14    2019-03-08 
#> # … with 240 more rows
```

Use the `derive_*()` functions to calculate key derived columns:

-   `followup_date` - `dateofdeath`, if known, and `censordate`,
    otherwise
-   `followup_days` - Days from `index_date` (in our case,
    `lotstartdate`) to `followup_date`
-   `event_status` - A logical column indicating whether or not the
    event (`dateofdeath`) is known.

``` r
cohort <- ez_cohort %>%
  
  derive_followup_date(event_date = "dateofdeath",
                        censor_date = "lastvisitdate") %>%
  
  derive_followup_time(index_date = "lotstartdate") %>%
  
  derive_event_status(event_date = "dateofdeath")
```

Here is our updated cohort object (moving our derived columns to the
front) for visibility

``` r
cohort %>%
  select(patientid, followup_date, followup_days, event_status, 
         everything())
#> # A tibble: 250 x 8
#>    patientid followup_date followup_days event_status condition lotstartdate
#>    <chr>     <date>                <dbl> <lgl>        <chr>     <date>      
#>  1 F00001    2020-12-01           1660.  FALSE        placebo   2016-05-17  
#>  2 F00002    2020-10-05             70.1 TRUE         placebo   2020-07-27  
#>  3 F00003    2017-03-13            333.  TRUE         drug      2016-04-14  
#>  4 F00004    2020-11-25            167.  FALSE        drug      2020-06-12  
#>  5 F00005    2020-02-21            338.  TRUE         placebo   2019-03-20  
#>  6 F00006    2017-11-19            232.  TRUE         placebo   2017-04-02  
#>  7 F00007    2019-02-17            388.  TRUE         placebo   2018-01-26  
#>  8 F00008    2015-12-23            175.  TRUE         placebo   2015-07-02  
#>  9 F00009    2020-08-17            528.  TRUE         drug      2019-03-08  
#> 10 F00010    2019-03-08            197.  TRUE         placebo   2018-08-23  
#> # … with 240 more rows, and 2 more variables: lastvisitdate <date>,
#> #   dateofdeath <date>
```

Use `plot_followup_time()` to visualize the time at risk data

``` r
plot_followup_time(cohort, 
                    followup_time = "followup_days", 
                    event_name = "Death", 
                    index_name = "LOT1 Start")
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="85%" />

Use `fit_survival()` to fit the survival model. We’ll set the follow up
time to be `followup_days` and specify “condition” as a term (i.e.;
covariate) to be used in the model

<!-- If we were using `survival::survfit()` we'd need to specify this nasty 
looking formula `survival::survfit(survival::Surv(followup_days, event_status, 
type = 'right') ~ group, data = cohort)` directly.  -->
<!-- With `fit_survival()`, we can simply specify the column names of interest 
and let the function take care of the formula: -->

``` r
cohort_fit <- fit_survival(cohort, 
                           followup_time = "followup_days", 
                           terms = "condition")
#> ── fit_survival ────────────────────────────────────────────────────────────────
#> - survival::survfit(survival::Surv(followup_days, event_status, type = 'right') ~ condition, data = cohort)
#> - 202 of 250 (81%) patient(s) experienced the event.
```

The result is a `survfit` object (from the `survival` package)

``` r
class(cohort_fit)
#> [1] "survfit"

cohort_fit
#> Call: survfit(formula = survival::Surv(followup_days, event_status, 
#>     type = "right") ~ condition, data = cohort)
#> 
#>                     n events median 0.95LCL 0.95UCL
#> condition=drug    132    105    410     329     590
#> condition=placebo 118     97    232     184     313
```

Use `plot_survival()` to plot the result. Use the `index_name` and
`event_name` to give descriptive names to the key events:

``` r
plot_survival(cohort_fit, 
              cohort = cohort, 
              index_name = "LOT1 Start", 
              event_name = "Death")
#> Warning: Vectorized input to `element_text()` is not officially supported.
#> Results may be unexpected or may change in future versions of ggplot2.
```

<img src="man/figures/README-unnamed-chunk-9-1.png" width="85%" />
