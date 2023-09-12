---
title: "Summarizing Data"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 4
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

## In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”

- [<i class="fas fa-file-archive"></i> `day1-summarise.R`](/slides/code/day1-summarise.R)
- [<i class="fas fa-file-archive"></i> `day2-summarise.R`](/slides/code/day2-summarise.R)

## Summarize

Let’s load the libraries.

``` r
# libraries
library(tidyverse)
library(nycflights13)
library(fivethirtyeight)
```

Say we want to take the average of a variable in our dataset. `summarize()` can help us do that.

Say we wanted to know how late in departure is the *average* flight in our dataset and what’s the latest a flight has ever been?

``` r
## on average, how late are flights in departing?
flights %>%
  summarise(avg_late = mean(dep_delay, na.rm = TRUE),
            most_late = max(dep_delay, na.rm = TRUE))
```

    ## # A tibble: 1 × 2
    ##   avg_late most_late
    ##      <dbl>     <dbl>
    ## 1     12.6      1301

Not the `na.rm = TRUE` above and what happens if you remove it. The problem is there are missing values (`NA`) in our data, and R can’t take the average of a bunch of numbers where some are missing. `na.rm = TRUE` tells R to ignore those missing numbers and use only the complete observations.

## Summarize + group_by()

Say we wanted to know how average departure delays vary across airlines. Conceptually, this means taking the average of departure delays for each airline in the dataset separately. We can do this by combining `group_by()` and `summarise()`.

``` r
# what if we wanted to know these statistics
## for each month in our dataset?
carrier_late = flights %>%
  group_by(carrier) %>%
  summarise(avg_late = mean(dep_delay, na.rm = TRUE),
            most_late = max(dep_delay, na.rm = TRUE))


# make a plot
ggplot(carrier_late, aes(x = carrier, y = avg_late)) +
  geom_col() +
  coord_flip()
```

<img src="/class/data-summary_files/figure-html/unnamed-chunk-3-1.png" width="672" />

## The Bob Ross example

Happy tree?

``` r
bob_ross %>%
  summarise(prop_tree = mean(tree, na.rm = TRUE))
```

    ## # A tibble: 1 × 1
    ##   prop_tree
    ##       <dbl>
    ## 1     0.896

Clouds over time?

``` r
bob_clouds = bob_ross %>%
  group_by(season) %>%
  summarise(prop_clouds = mean(clouds, na.rm = TRUE))

ggplot(bob_clouds, aes(x = season, y = prop_clouds)) + geom_line()
```

<img src="/class/data-summary_files/figure-html/unnamed-chunk-5-1.png" width="672" />

snowy mountain?

``` r
bob_ross %>%
  filter(mountain == 1) %>%
  summarise(snowiness = mean(snowy_mountain, na.rm = TRUE))
```

    ## # A tibble: 1 × 1
    ##   snowiness
    ##       <dbl>
    ## 1     0.681

``` r
bob_ross %>%
  group_by(mountain) %>%
  summarise(snowiness = mean(snowy_mountain, na.rm = TRUE))
```

    ## # A tibble: 2 × 2
    ##   mountain snowiness
    ##      <int>     <dbl>
    ## 1        0     0    
    ## 2        1     0.681

Steve ross?

``` r
bob_ross %>%
  group_by(steve_ross) %>%
  summarise(lake_chance = mean(lake, na.rm = TRUE))
```

    ## # A tibble: 2 × 2
    ##   steve_ross lake_chance
    ##        <int>       <dbl>
    ## 1          0       0.339
    ## 2          1       0.909

## The flying etiquette example

Middle arm rest?

``` r
middle_arm_rests = flying %>%
  group_by(two_arm_rests) %>%
  tally() %>%
  mutate(percent = n/sum(n))

ggplot(middle_arm_rests, aes(x = percent, y = two_arm_rests)) +
  geom_col()
```

<img src="/class/data-summary_files/figure-html/unnamed-chunk-8-1.png" width="672" />

Unruly children?

``` r
nasty_kids = flying %>%
  group_by(children_under_18, unruly_child) %>%
  tally() %>%
  mutate(p_unruly = n/sum(n))

ggplot(nasty_kids, aes(x = unruly_child, y = p_unruly, fill = children_under_18)) + geom_col(position = "dodge")
```

<img src="/class/data-summary_files/figure-html/unnamed-chunk-9-1.png" width="672" />
