---
title: "DAGs"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 8
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

# In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”.

- [<i class="fas fa-file-archive"></i> `day1-dags.R`](/slides/code/day1-dags.R)
- [<i class="fas fa-file-archive"></i> `day2-dags.R`](/slides/code/day2-dags.R)

## Making DAGs with ggdag()

To make DAGs we use `dagify()` from the `ggdag` package. Note the basic format:

- Y ~ A + B + C + … = think of this as “Y is caused by A, B, C, …”.
- you need to specify the treatment variable with `exposure` (another word for “treatment”) and the outcome variable with `outcome`

``` r
library(ggdag)
library(tidyverse)

# make a dag
dag = dagify(Y ~ X + A + B + C, 
             X ~ A, 
             A ~ B + C, exposure = "X", outcome = "Y")
```

We can then plot with `ggdag`:

``` r
# plot it
ggdag(dag)
```

<img src="/class/dags_files/figure-html/unnamed-chunk-2-1.png" width="672" />

We can make prettier in the following ways (but no need):

``` r
ggdag(dag) + theme_dag()
```

<img src="/class/dags_files/figure-html/unnamed-chunk-3-1.png" width="672" />

We can use `ggdag_paths` to identify front and backdoor paths from treatment to outcome:

``` r
ggdag_paths(dag)
```

<img src="/class/dags_files/figure-html/unnamed-chunk-4-1.png" width="672" />

We can use `ggdag_adjustment_set` to see what variables we need to control for:

``` r
ggdag_adjustment_set(dag)
```

<img src="/class/dags_files/figure-html/unnamed-chunk-5-1.png" width="672" />

We can also use words instead of letters:

``` r
## made up exqmple: shuttle service --> turnout
dag = dagify(turnout ~ shuttle + income + distance_poll + schedule_flex, 
             income ~ job + location, 
             distance_poll ~ location + car, 
             shuttle ~ cost + partisan + location, 
             exposure = "shuttle", outcome = "turnout")
```

If we want to use labels instead of text to make it easier to read:

``` r
ggdag(dag, use_labels = "name", text = FALSE) + theme_dag()
```

<img src="/class/dags_files/figure-html/unnamed-chunk-7-1.png" width="672" />
