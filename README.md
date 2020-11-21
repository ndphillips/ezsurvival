
<!-- README.md is generated from README.Rmd. Please edit that file -->

# iwillsurvive 0.1.0 <img src="https://cdn.iconscout.com/icon/free/png-512/disco-1-62616.png" align="right" height="139"/>

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![Codename:
shuffled](https://img.shields.io/badge/version-0.1_'Gloria'-yellow.svg)](https://en.wikipedia.org/wiki/Gloria_Gaynor)

The goal of iwillsurvive is to make it easy to fit and visualize
survival models. Essentially, it provides wrapper functions around
commonly used functions from survival packages such as
`survival::survfit()` and `survminer::ggsurvplot()`, while providing
user-friendly in-line messages, notes, and warnings.

## Installation

`iwillsurvive` is hosted at
<https://git.the.flatiron.com/qs_r_packages/iwillsurvive>. Here is how
to install it:

``` r
devtools::install_gitlab(repo = "qs_r_packages/iwillsurvive", 
                         host = "git.the.flatiron.com",
                         build_vignettes = TRUE)
```

## Example

``` r
library(iwillsurvive)
#> -----------------------------------------------------------------------
#> iwillsurvive 0.1.0 'Gloria'
#> Intro   : vignette('introduction', 'iwillsurvive')
#> Repo    : https://git.the.flatiron.com/qs_r_packages/iwillsurvive
#> Contact : #TBD
#> .......................................................................
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
contains columns corresponding to patientid, IndexDate (like
`lotstartdate`), censordate, and an EventDate (like `dateofdeath`)

For this example, we’ll `ez_cohort`, a dataframe of simulated patients
contained in `iwillsurvive`:

``` r
ez_cohort
#> # A tibble: 100 x 5
#>    patientid group lotstartdate censordate dateofdeath
#>    <chr>     <chr> <date>       <date>     <date>     
#>  1 F00001    a     2020-07-20   2020-09-15 NA         
#>  2 F00002    b     2020-04-21   2020-07-27 NA         
#>  3 F00003    b     2020-07-24   NA         2020-08-09 
#>  4 F00004    a     2020-01-04   2020-08-14 NA         
#>  5 F00005    a     2020-04-07   2020-08-22 NA         
#>  6 F00006    b     2020-01-07   2020-04-23 NA         
#>  7 F00007    b     2020-07-01   NA         2020-07-10 
#>  8 F00008    a     2020-10-25   NA         2020-11-30 
#>  9 F00009    a     2020-05-25   2020-06-20 NA         
#> 10 F00010    a     2020-10-07   2020-11-07 NA         
#> # … with 90 more rows
```

Use the derive\_ functions to calculate key derived columns:

-   `follow_up_date` - dateofdeath, if known, and censordate, otherwise
-   `follow_up_days` - Days from lotstartdate to follow\_up\_date
-   `Death` - A logical column indicating whether or not the event
    (death) is known

``` r
cohort <- ez_cohort %>%
  
  derive_follow_up_date(event_date = "dateofdeath",
                      censor_date = "censordate") %>%
  
  derive_follow_up_time(index_date = "lotstartdate") %>%
  
  derive_event_status(event_date = "dateofdeath")
```

Here is our updated cohort object:

``` r
cohort %>%
  select(patientid, follow_up_date, follow_up_days, event_status, everything())
#> # A tibble: 100 x 8
#>    patientid follow_up_date follow_up_days event_status group lotstartdate
#>    <chr>     <date>                  <dbl> <lgl>        <chr> <date>      
#>  1 F00001    2020-09-15              57.1  FALSE        a     2020-07-20  
#>  2 F00002    2020-07-27              97.1  FALSE        b     2020-04-21  
#>  3 F00003    2020-08-09              16.5  TRUE         b     2020-07-24  
#>  4 F00004    2020-08-14             224.   FALSE        a     2020-01-04  
#>  5 F00005    2020-08-22             138.   FALSE        a     2020-04-07  
#>  6 F00006    2020-04-23             107.   FALSE        b     2020-01-07  
#>  7 F00007    2020-07-10               9.87 TRUE         b     2020-07-01  
#>  8 F00008    2020-11-30              36.6  TRUE         a     2020-10-25  
#>  9 F00009    2020-06-20              26.4  FALSE        a     2020-05-25  
#> 10 F00010    2020-11-07              31.7  FALSE        a     2020-10-07  
#> # … with 90 more rows, and 2 more variables: censordate <date>,
#> #   dateofdeath <date>
```

Use `plot_follow_up_time` to visualize the time at risk data

``` r
plot_follow_up_time(cohort, 
                    follow_up_time = "follow_up_days", 
                    event_name = "Death", 
                    index_name = "LOT1 Start")
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="85%" />

Use `fit_survival()` to fit the survival model - we’ll set the follow up
time to be follow\_up\_days, event status to be Death, and include group
as a categorical covariate.

<!-- If we were using `survival::survfit()` we'd need to specify this nasty looking formula `survival::survfit(survival::Surv(follow_up_days, event_status, type = 'right') ~ group, data = cohort)` directly.  -->
<!-- With `fit_survival()`, we can simply specify the column names of interest and let the function take care of the formula: -->

``` r
cohort_fit <- fit_survival(cohort, 
                           follow_up_time = "follow_up_days", 
                           terms = "group")
#> ── fit_survival ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
#> - survival::survfit(survival::Surv(follow_up_days, event_status, type = 'right') ~ group, data = cohort)
#> - 39 of 100 (39%) patient(s) experienced the event.
```

The result is a `survfit` object (from the `survival` package)

``` r
class(cohort_fit)
#> [1] "survfit"

cohort_fit
#> Call: survfit(formula = survival::Surv(follow_up_days, event_status, 
#>     type = "right") ~ group, data = cohort)
#> 
#>          n events median 0.95LCL 0.95UCL
#> group=a 53     19   91.0    46.5      NA
#> group=b 47     20   83.8    55.9      NA
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
