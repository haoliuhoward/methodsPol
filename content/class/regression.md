---
title: "Correlation and regression"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 5
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

## In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”

- [<i class="fas fa-file-archive"></i> `day1-models.R`](/slides/code/day1-models.R)
- [<i class="fas fa-file-archive"></i> `day2-models.R`](/slides/code/day2-models.R)

## Drawing lines (geom_smooth)

Why draw trend lines? Trend lines give us a good, educated guess as to what the value of a Y variable is given some value of X. We can draw a trend line (or line of best fit) using `geom_smooth`, as below. Notice `method = "lm"`.

``` r
# libraries
library(tidyverse)

# mtcars
ggplot(mtcars, aes(x = wt, y = mpg)) + geom_point() +
  geom_smooth(method = "lm")
```

<img src="/class/regression_files/figure-html/unnamed-chunk-1-1.png" width="672" />

## ggcorrplot

``` r
# libraries
library(tidyverse)
library(socviz)
library(fivethirtyeight)
library(gapminder)
library(nycflights13)
library(ggcorrplot)
library(juanr)
library(palmerpenguins)
```

Look at the correlations:

``` r
# switch out gapminder with a dataset you want below
therm %>%
  # correlation only works with numeric columns; keep only those
  select(where(is.numeric)) %>%
  # the cor() function doesn't take NA; drop them all
  drop_na() %>%
  # get the correlation
  cor() %>%
  # plot the correlation
  ggcorrplot(lab = TRUE)
```

<img src="/class/regression_files/figure-html/unnamed-chunk-3-1.png" width="672" />

## Draw the line

``` r
# draw a line: alter the intercept and slope in geom_abline()
# to draw the line
  ggplot() + geom_abline(intercept = 1, slope = 2, size = 1) +
  # change the limits on the x and y-axis
  scale_x_continuous(limits = c(-10, 10)) +
  scale_y_continuous(limits = c(-10, 10)) +
  # add a vertical and horizontal line at 0
  geom_hline(yintercept = 0, lty = 2) +
  geom_vline(xintercept = 0, lty = 2) +
  theme_bw()
```

<img src="/class/regression_files/figure-html/unnamed-chunk-4-1.png" width="672" />

## Convince yourself

Make the scatterplot:

``` r
# convince yourself about the line of best fit: run this code below
# set the seed
set.seed(1990)

# make the fake data
df = tibble(x = rnorm(50, mean = 10),
            y = 3 + 2*x + rnorm(50))

# line of best fit?
model = lm(y ~ x, df)
true = tibble(true_intercept = coef(model)[1],
              true_slope = coef(model)[2])

ggplot(data = df, aes(x = x, y = y)) +
  geom_point() +
  geom_smooth(method = "lm")
```

<img src="/class/regression_files/figure-html/unnamed-chunk-5-1.png" width="672" />

## IR Econ

The plot

``` r
ir_1959 = trade %>%
  filter(year == 2008)


ggplot(ir_1959, aes(x = imports, y = exports,
                    label = country)) + geom_point() + geom_smooth(method = "lm") +
  geom_text()
```

<img src="/class/regression_files/figure-html/unnamed-chunk-6-1.png" width="672" />

Estimate a model, and interpret:

``` r
library(broom)

igo_pop = lm(exports ~ pop, data = trade)

tidy(igo_pop)
```

    ## # A tibble: 2 × 5
    ##   term            estimate     std.error statistic  p.value
    ##   <chr>              <dbl>         <dbl>     <dbl>    <dbl>
    ## 1 (Intercept) 13322.       1023.              13.0 1.81e-38
    ## 2 pop             0.000393    0.00000920      42.7 0

Penguins regression:

``` r
penguins_model = lm(body_mass_g ~ species,
                    data = penguins)

tidy(penguins_model)
```

    ## # A tibble: 3 × 5
    ##   term             estimate std.error statistic   p.value
    ##   <chr>               <dbl>     <dbl>     <dbl>     <dbl>
    ## 1 (Intercept)        3701.       37.6    98.4   2.49e-251
    ## 2 speciesChinstrap     32.4      67.5     0.480 6.31e-  1
    ## 3 speciesGentoo      1375.       56.1    24.5   5.42e- 77

Interpretation:

- Chinstrap penguins weigh 32 more grams, on average, than Adelie penguins.
- Gentoo penguins weigh 1,375 more grams, on average, than Adelie penguins.
- Adelie penguins weigh, on average, 3,700 grams.

another one:

``` r
lm(tvhours ~ race, data = gss_cat) %>%
  tidy()
```

    ## # A tibble: 3 × 5
    ##   term        estimate std.error statistic   p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>     <dbl>
    ## 1 (Intercept)  2.76       0.0792    34.9   3.90e-253
    ## 2 raceBlack    1.42       0.100     14.1   5.90e- 45
    ## 3 raceWhite    0.00894    0.0838     0.107 9.15e-  1

Interpretation:

- Black respondents watch 1.42 more hours of tv, on average, than respondents who identify as “Other”.
- White respondents watch .009 more hours of tv, on average, than respondents who identify as “Other”.
- Respondents who identify as “Other” watch, on average, 2.76 hours of TV.
