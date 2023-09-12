---
title: "Natural Experiments"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 10
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

# In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”.

[<i class="fas fa-file-archive"></i> `socviz-maps.R`](/slides/code/socviz-maps.R)

## Closing backdoors with controls

``` r
library(ggdag)
library(tidyverse)
library(broom)

# set seed
set.seed(1990)
```

Remember that one of the lessons from our week on DAGs is that in order to estimate the effect of X on Y, sometimes we need to control for some variables and *avoid* controlling for others.

### Dealing with forks

Here’s an example dealing with forks. The classic fork looks like this:

``` r
dag = dagify(Y ~ Z, 
       X ~ Z, 
       exposure = "X", 
       outcome = "Y")
ggdag(dag) + theme_dag_blank()
```

<img src="/class/controls_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Notice that we need to control for Z:

``` r
ggdag_adjustment_set(dag)
```

<img src="/class/controls_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Let’s simulate some fake data from this DAG. Notice that X does not cause Y:

``` r
# fake data
fake = tibble(person = 1:200, 
       z = rnorm(200), 
       x = rnorm(200) + 2*z, 
       y = rnorm(200) - 3*z)
```

And yet when we look at X and Y there’s a relationship!

``` r
ggplot(fake, aes(x = x, y = y)) + 
  geom_point() + geom_smooth(method = "lm")
```

<img src="/class/controls_files/figure-html/unnamed-chunk-5-1.png" width="672" />

We can see this relationship in regression too:

``` r
# naive regression
lm(y ~ x, data = fake) %>% tidy()
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)   0.0655    0.125      0.523 6.01e- 1
    ## 2 x            -1.20      0.0524   -23.0   1.07e-57

But notice that when we correcty control for Z, the estimate on X goes to 0:

``` r
# with controls
lm(y ~ x + z, data = fake) %>% tidy()
```

    ## # A tibble: 3 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)   0.0515    0.0741     0.694 4.88e- 1
    ## 2 x            -0.0245    0.0689    -0.356 7.22e- 1
    ## 3 z            -2.99      0.156    -19.1   7.53e-47

### Dealing with pipes

Here’s an example dealing with pipes The classic pipe looks like this:

``` r
dag = dagify(Y ~ Z, 
             Z ~ X, 
             exposure = "X", 
             outcome = "Y")
ggdag(dag) + theme_dag_blank()
```

<img src="/class/controls_files/figure-html/unnamed-chunk-8-1.png" width="672" />

Notice that we **shouldn’t** control for Z:

``` r
ggdag_adjustment_set(dag)
```

<img src="/class/controls_files/figure-html/unnamed-chunk-9-1.png" width="672" />

Let’s simulate some fake data from this DAG. Notice that X -\> Y -\> Z:

``` r
# fake data
fake = tibble(person = 1:200, 
              x = rnorm(200), 
              z = rnorm(200) + 2*x, 
              y = rnorm(200) - 3*z)
```

When we look at X and Y, there’s a relationship:

``` r
ggplot(fake, aes(x = x, y = y)) + 
  geom_point() + geom_smooth(method = "lm")
```

<img src="/class/controls_files/figure-html/unnamed-chunk-11-1.png" width="672" />

We can see this relationship in regression too:

``` r
lm(y ~ x, data = fake) %>% tidy()
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)    0.302     0.217      1.39 1.65e- 1
    ## 2 x             -6.17      0.235    -26.3  1.81e-66

But notice that when we **incorrectly** control for Z, the estimate on X goes to 0:

``` r
# with controls
lm(y ~ x + z, data = fake) %>% tidy()
```

    ## # A tibble: 3 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)  0.00961    0.0698     0.138 8.91e- 1
    ## 2 x           -0.0667     0.164     -0.406 6.85e- 1
    ## 3 z           -2.99       0.0718   -41.7   1.03e-99

### Dealing with colliders

Here’s an example dealing with colliders The classic collider looks like this:

``` r
dag = dagify(Z ~ Y + X, 
             exposure = "X", 
             outcome = "Y")
ggdag(dag) + theme_dag_blank()
```

<img src="/class/controls_files/figure-html/unnamed-chunk-14-1.png" width="672" />

Notice that we **shouldn’t** control for Z:

``` r
ggdag_adjustment_set(dag)
```

<img src="/class/controls_files/figure-html/unnamed-chunk-15-1.png" width="672" />

Let’s simulate some fake data from this DAG. Notice that X does not cause Y:

``` r
# fake data
fake = tibble(person = 1:200, 
              x = rnorm(200), 
              y = rnorm(200),
              z = rnorm(200) + 2*x + -3*y)
```

When we look at X and Y alone, there’s no relationship:

``` r
ggplot(fake, aes(x = x, y = y)) + 
  geom_point() + geom_smooth(method = "lm")
```

<img src="/class/controls_files/figure-html/unnamed-chunk-17-1.png" width="672" />

We can see this relationship in regression too:

``` r
# naive regression
lm(y ~ x, data = fake) %>% tidy()
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>   <dbl>
    ## 1 (Intercept)  -0.0602    0.0765    -0.788   0.432
    ## 2 x            -0.0991    0.0738    -1.34    0.181

But notice that when we **incorrectly** control for Z, the estimate on X goes away from 0:

``` r
# with controls
lm(y ~ x + z, data = fake) %>% tidy()
```

    ## # A tibble: 3 × 5
    ##   term        estimate std.error statistic   p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>     <dbl>
    ## 1 (Intercept)  0.00726   0.0235      0.309 7.58e-  1
    ## 2 x            0.577     0.0274     21.1   2.62e- 52
    ## 3 z           -0.305     0.00698   -43.7   2.68e-103
