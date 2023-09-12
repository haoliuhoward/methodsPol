---
title: "Prediction"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 6
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

## In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”

- [<i class="fas fa-file-archive"></i> `day1-prediction.R`](/slides/code/day1-prediction.R)

## Predictions (simple model, one explanatory variable)

``` r
# libraries
library(tidyverse)
library(broom)
library(moderndive)
```

Let’s say we want to use mtcars to make predictions about a car’s fuel efficency (mpg) using a car’s weight (wt).

``` r
# data
head(mtcars)
```

    ##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## Mazda RX4         21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
    ## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
    ## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
    ## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1

First we fit a model and look at the output.

``` r
# fitting a model of mpg ~ weight
mod = lm(mpg ~ wt, data = mtcars)
tidy(mod)
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)    37.3      1.88      19.9  8.24e-19
    ## 2 wt             -5.34     0.559     -9.56 1.29e-10

Next, we define a scenario we want a prediction for. For example, what MPG should we expect with a car that weighs 1.05 tons?

``` r
scenario = crossing(wt = 1.05)
```

Finally, we use the `augment` function to get our prediction. We give it our model and our scenario.

``` r
# get my prediction with augment()
augment(mod, newdata = scenario)
```

    ## # A tibble: 1 × 2
    ##      wt .fitted
    ##   <dbl>   <dbl>
    ## 1  1.05    31.7

We can also do this by hand (plus or minus some rounding!). We just take our regression equation and plug in 1.05 for weight:

``` r
37.3 + -5.34*1.05
```

    ## [1] 31.693

We can make more complicated predictions. For example, predicted MPG for cars with weights 1 through 5:

``` r
# what is the predicted mpg, for a car with: 
# wt = 1, 2, 3, 4, 5 tons?
scenario = crossing(wt = c(1, 2, 3, 4, 5))

# get prediction (i saved as data object to plot below)
preds_wt = augment(mod, newdata = scenario)

# look at predictions
preds_wt
```

    ## # A tibble: 5 × 2
    ##      wt .fitted
    ##   <dbl>   <dbl>
    ## 1     1    31.9
    ## 2     2    26.6
    ## 3     3    21.3
    ## 4     4    15.9
    ## 5     5    10.6

We could use this to plot our predictions:

``` r
ggplot(preds_wt, aes(x = wt, y = .fitted)) + 
  # the stuff in geom_col is just for aesthetic purposes!
  geom_col(color = "white", fill = "red", alpha = .8) + 
  theme_light() + 
  labs(x = "Weight (in tons)", y = "Predicted fuel efficiency")
```

<img src="/class/prediction_files/figure-html/unnamed-chunk-8-1.png" width="672" />

## More complicated models (multiple explanatory variables)

The real power of modeling is in using more than one explanatory variable. Below, we can model MPG using car weight (wt), number of cylinders (cyl), horsepower (hp), and the shape of the engine (vs; vs = 0 is v-shaped, vs = 1 is straight).

``` r
mod = lm(mpg ~ wt + cyl + hp + vs, data = mtcars)
tidy(mod)
```

    ## # A tibble: 5 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)  38.5       3.34     11.5    6.14e-12
    ## 2 wt           -3.18      0.774    -4.11   3.26e- 4
    ## 3 cyl          -0.905     0.679    -1.33   1.94e- 1
    ## 4 hp           -0.0179    0.0122   -1.46   1.56e- 1
    ## 5 vs            0.155     1.62      0.0957 9.24e- 1

We can then make use of these explanatory variables to make (hopefully) more precise predictions about the outcome we care about. Here’s an example predicting fuel efficiency for a 1 ton car, with a 4 cylinder engine, with 120 horsepower, and a straight engine:

``` r
# set scenario
scen = crossing(wt = 3, 
              cyl = 4, 
              hp = 160, 
              vs = 0)

# get prediction
augment(mod, newdata = scen)
```

    ## # A tibble: 1 × 5
    ##      wt   cyl    hp    vs .fitted
    ##   <dbl> <dbl> <dbl> <dbl>   <dbl>
    ## 1     3     4   160     0    22.5

I can also get predictions where I vary the values one variable takes on and leave the others at a constant value:

``` r
# i have a car that weighs 1 ton, hp = 100, vs = 0, 
# what does fuel efficiency look like, if I have 4, 6, 8?
scenario = crossing(wt = 1, 
                  hp = 100, 
                  vs = 0, 
                  cyl = c(4, 6, 8))

augment(mod, newdata = scenario)
```

    ## # A tibble: 3 × 5
    ##      wt    hp    vs   cyl .fitted
    ##   <dbl> <dbl> <dbl> <dbl>   <dbl>
    ## 1     1   100     0     4    29.9
    ## 2     1   100     0     6    28.1
    ## 3     1   100     0     8    26.3

## Interpreting regression output

Remember, the basic template for interpreting regression coefficients for **continuous variables** is the following:

> For every unit increase in EXPLANATORY VARIABLE there is an associated COEFFICIENT ESTIMATE change in OUTCOME VARIABLE

The basic template for interpreting regression coefficients for **categorical variables** is the following:

> observations with CATEGORY have, on average, COEFFICIENT ESTIMATE more/less OUTCOME VARIABLE than observations with BASELINE CATEGORY

And the template for interpreting the *intercept* is the following:

> The average value of the OUTCOME VARIABLE when all EXPLANATORY VARIABLES are set to 0

So with our model above:

``` r
tidy(mod)
```

    ## # A tibble: 5 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)  38.5       3.34     11.5    6.14e-12
    ## 2 wt           -3.18      0.774    -4.11   3.26e- 4
    ## 3 cyl          -0.905     0.679    -1.33   1.94e- 1
    ## 4 hp           -0.0179    0.0122   -1.46   1.56e- 1
    ## 5 vs            0.155     1.62      0.0957 9.24e- 1

- `wt` = for every ADDITIONAL TON OF WEIGHT a car has, there is an associated **3.18** decrease in expected MILES PER GALLON (continuous)
- `cyl` = for every ADDITIONAL ENGINE CYLYNDER a car has, there is an associated **.905** decrease in expected MILES PER GALLON (continuous)
- `hp` = for every ADDITIONAL UNIT OF HORSE POWER a car has, there is an associated **.018** decrease in expected MILES PER GALLON (continuous)
- `vs` = cars with STRAIGHT ENGINES have, on average, .155 more MILES PER GALLON than cars with V-SHAPED ENGINES. (categorical)
- `intercept` = The average MILES PER GALLON of a car with *0 weight (wt = 0)*, *0 cylinders (cyl = 0)*, *0 horsepower (hp = 0)*, and *a v-shaped engine (vs = 0)* is **38.5**.

Here’s a different example, using the `evals` dataset:

``` r
# fit model
mod_evals = lm(score ~ age + bty_avg + gender + rank, data = evals)
tidy(mod_evals)
```

    ## # A tibble: 6 × 5
    ##   term             estimate std.error statistic  p.value
    ##   <chr>               <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)       4.32      0.195       22.1  3.42e-74
    ## 2 age              -0.00840   0.00314     -2.67 7.78e- 3
    ## 3 bty_avg           0.0624    0.0168       3.72 2.22e- 4
    ## 4 gendermale        0.209     0.0522       4.01 7.07e- 5
    ## 5 ranktenure track -0.226     0.0804      -2.81 5.13e- 3
    ## 6 ranktenured      -0.153     0.0622      -2.46 1.42e- 2

- `age` = for every ADDITIONAL YEAR OF AGE of a professor, there is an associated **.008** decrease in their STUDENT EVALUATION SCORE (continuous)
- `bty_score` = for every ADDITIONAL POINT OF of a professor’s BEAUTY SCORE, there is an associated **.008** decrease in their STUDENT EVALUATION SCORE (continuous)
- `gendermale` = MALE professors have, on average, .2 more points on their STUDENT EVALUATION SCORE than FEMALE professors (continuous)
- `ranktenure track` = TENURE TRACK professors have, on average, .226 fewer points on their STUDENT EVALUATION SCORE than TEACHING (the baseline!) professors (categorical)
- `ranktenured` = TENURED professors have, on average, .153 fewer points on their STUDENT EVALUATION SCORE than TEACHING (the baseline!) professors (categorical)
- `intercept` = the average STUDENT EVALUATION SCORE for a professor who is *0 years old (age = 0)*, has a *beauty score of 0 (bty_avg = 0)*, is *female (gendermale = 0)*, is *not tenure track (ranktenure track = 0)* and *not tenured (ranktenured = 0)* (i.e., they are “teaching track”, the baseline category)
