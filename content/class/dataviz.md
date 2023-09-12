---
title: "Data visualization"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 2
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

# In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”.

- Day one: [<i class="fas fa-file-archive"></i> `Five graphs`](/files/five-graphs.R)
- Day two: [<i class="fas fa-file-archive"></i> `Day 2`](/files/viz-day2.R)

# Making graphs with ggplot

Here we will walk through how to make some of the basic graphs in R.

The code chunk below loads our libraries and prepares the data.

``` r
# load libraries -------------------------------------------------------------------------
library(tidyverse)
library(gapminder) # install this if you don't have it!
library(ggbeeswarm) # install this if you don't have it!
library(moderndive)

# clean data ------------------------------------------------------------------------


# subset data to focus on 2007
gap_07 =
  gapminder %>%
  filter(year == 2007)


# calculate average life span by year
life_yr =
  gapminder %>%
  select(year, lifeExp) %>%
  group_by(year) %>%
  summarise(avg_yrs = mean(lifeExp))

# calculate average life expectancy by continent
life_region =
  gap_07 %>%
  group_by(continent) %>%
  summarise(avg_yrs = mean(lifeExp))

# calculate average life expectancy by continent-year
life_region_yr =
  gapminder %>%
  group_by(continent, year) %>%
  summarise(avg_yrs = mean(lifeExp))
```

## The scatterplot

The scatterplot puts two variables on the same graph so we can see how they move together:

``` r
# the fancy, final product
ggplot(gap_07, aes(x = gdpPercap, y = lifeExp,
                   color = continent, size = pop)) +
  geom_point()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-2-1.png" width="672" />

We can add labels using `labs()` and a `theme` using `theme_bw()`:

``` r
ggplot(gap_07, aes(x = gdpPercap, y = lifeExp,
                   color = continent, size = pop)) +
  geom_point() +
  labs(x = "GDP per capita ($USD, inflation-adjusted)",
       y = "Life expectancy (in years)",
       title = "Wealth and Health Around the World",
       subtitle = "Data from 2007. Source: gapminder package.") +
  theme_bw()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Note that there are many [more themes out there](https://ggplot2.tidyverse.org/reference/ggtheme.html).

## The time series

The time series shows you how a variable moves over time, using a line. For this one we will use the `life_yr` data object we constructed above, which looks like this:

``` r
## look at the data
life_yr
```

    ## # A tibble: 12 × 2
    ##     year avg_yrs
    ##    <int>   <dbl>
    ##  1  1952    49.1
    ##  2  1957    51.5
    ##  3  1962    53.6
    ##  4  1967    55.7
    ##  5  1972    57.6
    ##  6  1977    59.6
    ##  7  1982    61.5
    ##  8  1987    63.2
    ##  9  1992    64.2
    ## 10  1997    65.0
    ## 11  2002    65.7
    ## 12  2007    67.0

We make the plot below:

``` r
# make the plot
ggplot(life_yr, aes(x = year, y = avg_yrs)) +
  geom_line() +
  theme_bw()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-5-1.png" width="672" />

## Multiple time series

Sometimes we have data where we observe multiple *units* moving over time, like so:

``` r
life_region_yr
```

    ## # A tibble: 60 × 3
    ## # Groups:   continent [5]
    ##    continent  year avg_yrs
    ##    <fct>     <int>   <dbl>
    ##  1 Africa     1952    39.1
    ##  2 Africa     1957    41.3
    ##  3 Africa     1962    43.3
    ##  4 Africa     1967    45.3
    ##  5 Africa     1972    47.5
    ##  6 Africa     1977    49.6
    ##  7 Africa     1982    51.6
    ##  8 Africa     1987    53.3
    ##  9 Africa     1992    53.6
    ## 10 Africa     1997    53.6
    ## # ℹ 50 more rows

We can draw separate time series for each *unit* by either using the `color` aesthetic to separate the lines:

``` r
ggplot(life_region_yr, aes(x = year, y = avg_yrs, color = continent)) +
  geom_line() +
  theme_bw()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Or the `group` aesthetic:

``` r
ggplot(life_region_yr, aes(x = year, y = avg_yrs, group = continent)) +
  geom_line() +
  theme_bw()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-8-1.png" width="672" />

## Histogram with `geom_histogram()`

``` r
ggplot(gap_07, aes(x = lifeExp)) +
  geom_histogram() +
  theme_bw()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-9-1.png" width="672" />

## Grouped histogram

``` r
ggplot(gap_07, aes(x = lifeExp, fill = continent)) +
  geom_histogram() +
  theme_bw()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-10-1.png" width="672" />

## Barplot with `geom_col()`

``` r
ggplot(life_region_yr, aes(x = continent, y = avg_yrs)) +
  geom_col()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-11-1.png" width="672" />

## The boxplot

``` r
ggplot(data = gapminder, aes(x = continent, y = lifeExp)) + 
  geom_boxplot()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-12-1.png" width="672" />

# Other layers and features

## Multi-panel plots with `facet_wrap()`

``` r
ggplot(gapminder, aes(x = lifeExp, fill = continent)) +
  geom_histogram() +
  facet_wrap(vars(year)) +
  theme_minimal()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-13-1.png" width="672" />

## Make aesthetics static within the geometries

``` r
ggplot(gap_07, aes(x = gdpPercap, y = lifeExp)) + theme_light() +
  geom_point(size = 3, color = "orange", shape = 2, alpha = .5) #<<
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-14-1.png" width="60%" />

Take your aesthetics out of `aes()` and into `geom()` to make them *static*

## Show a trend-line using `geom_smooth()`

``` r
ggplot(evals, aes(x = age, y = score)) +
  geom_point() + theme_bw() + labs(x = "Professor age", y = "Student evals") +
  geom_smooth() #<<
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-15-1.png" width="60%" />

Trend lines can reveal patterns in “clumpy” data

## Show *separate* trend-lines

``` r
ggplot(evals, aes(x = age, y = score, color = gender)) + #<<
  geom_point() + theme_bw() + labs(x = "Professor age", y = "Student evals") +
  geom_smooth() #<<
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-16-1.png" width="60%" />

Relationships can look different *within* groups

## Use different color and fill scales

``` r
ggplot(gapminder, aes(x = lifeExp, fill = continent)) +
  geom_histogram() +
  scale_fill_brewer(palette = "Blues") #<<
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-17-1.png" width="60%" />

`fill_brewer()` for `fill`, `color_brewer` for `color`

## Many other themes

``` r
library(tvthemes)
ggplot(gapminder, aes(x = lifeExp, fill = continent)) +
  geom_histogram() + theme_spongeBob() + labs(title = "Horrible") +
  scale_fill_spongeBob()
```

<img src="/class/dataviz_files/figure-html/unnamed-chunk-18-1.png" width="80%" />

`theme_spongeBob()` from `tvthemes` package, many more online
