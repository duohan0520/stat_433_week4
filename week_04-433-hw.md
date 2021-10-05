week\_04
================
Duohan Zhang
10/4/2021

## Intro:

In this file, we use three different methods to reach different
conclusions on what time of day is most suitable for avoiding delayed
flights based on three different patterns. The first one is from data
“weather” in which weather info for every hour of every day is recorded.
The second one is from data “flights” from which we directly subtract
arriving delayed info. The third one is from both data “flights” and
data “planes”. We combine two datasets by using function left\_join so
every flights has the plane info.

``` r
library(dplyr)
```

    ## Warning: package 'dplyr' was built under R version 4.0.2

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(nycflights13)
```

    ## Warning: package 'nycflights13' was built under R version 4.0.2

``` r
library(ggplot2)
```

    ## Warning: package 'ggplot2' was built under R version 4.0.2

``` r
distinct(flights["hour"])
```

    ## # A tibble: 20 × 1
    ##     hour
    ##    <dbl>
    ##  1     5
    ##  2     6
    ##  3     7
    ##  4     8
    ##  5    18
    ##  6     9
    ##  7    10
    ##  8    11
    ##  9    12
    ## 10    13
    ## 11    14
    ## 12    15
    ## 13    16
    ## 14    17
    ## 15    19
    ## 16    20
    ## 17    21
    ## 18    22
    ## 19    23
    ## 20     1

From above: We see that there are no flights in 0,2,3,4 hour in a day.

1st pattern:weather I think visibility determines whether a flight will
be cancelled or not. If visibility of time A is longer than visibility
in time B, we think flights in time A are less likely to be delayed.

``` r
weather %>%
  group_by(hour) %>%
  summarise(visib_mean = mean(visib,na.rm = TRUE)) %>%
  arrange(desc(visib_mean))
```

    ## # A tibble: 24 × 2
    ##     hour visib_mean
    ##    <int>      <dbl>
    ##  1    18       9.40
    ##  2    17       9.38
    ##  3    16       9.38
    ##  4    21       9.38
    ##  5    19       9.36
    ##  6    15       9.36
    ##  7    13       9.35
    ##  8    14       9.33
    ##  9    20       9.33
    ## 10    22       9.33
    ## # … with 14 more rows

From the result above, we see that 18th hour in a day is the best to
have a flight since it has the best visibility averagely.

2nd pattern: directly calculate delaying proportion in flights. We think
delaying no more than 10 mins is regarded as a good flight.

``` r
flights %>%
  group_by(hour) %>%
  summarise(prop_of_good_flight = sum(arr_delay < 10,na.rm = TRUE)/n()) %>%
  arrange(desc(prop_of_good_flight))
```

    ## # A tibble: 20 × 2
    ##     hour prop_of_good_flight
    ##    <dbl>               <dbl>
    ##  1     5               0.845
    ##  2     7               0.829
    ##  3     6               0.823
    ##  4     8               0.777
    ##  5     9               0.767
    ##  6    11               0.754
    ##  7    10               0.752
    ##  8    12               0.730
    ##  9    13               0.697
    ## 10    14               0.666
    ## 11    23               0.629
    ## 12    15               0.624
    ## 13    16               0.621
    ## 14    22               0.603
    ## 15    17               0.601
    ## 16    18               0.598
    ## 17    19               0.581
    ## 18    20               0.579
    ## 19    21               0.559
    ## 20     1               0

From the result above, we see that in 2013 New York City, the best
flight time is 5th hour in a day.

3rd pattern: combine flights and planes so that each flight has a
manufacuring year for its plane. We tend to think a flight is less
likely to be delayed if the plane has the nearest manufacuring year. For
example, if time A has larger manufacturing year than that of time B, we
tend to choose time A.

``` r
flights %>%
  left_join(planes, by = "tailnum") %>%
  group_by(hour) %>%
  summarise(manu_year_ave = mean(year.y,na.rm = TRUE)) %>%
  arrange(desc(manu_year_ave))
```

    ## # A tibble: 20 × 2
    ##     hour manu_year_ave
    ##    <dbl>         <dbl>
    ##  1    22         2007.
    ##  2    23         2005.
    ##  3    20         2003.
    ##  4    21         2002.
    ##  5    16         2002.
    ##  6     6         2002.
    ##  7     5         2002.
    ##  8     9         2002.
    ##  9    14         2002.
    ## 10    18         2002.
    ## 11    13         2002.
    ## 12     8         2001.
    ## 13    17         2001.
    ## 14    10         2001.
    ## 15    12         2001.
    ## 16    19         2001.
    ## 17    11         2001.
    ## 18     7         2001.
    ## 19    15         2000.
    ## 20     1          NaN

From the result above, we see that averagely speaking, the planes in
22th hour have the largest manufacturing year. So we tend to think it is
the best.
