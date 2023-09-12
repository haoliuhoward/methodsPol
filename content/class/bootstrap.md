---
title: "Bootstrap"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 14
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

# In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”.

[<i class="fas fa-file-archive"></i> `day2-uncertainty.R`](/slides/code/day2-uncertainty.R)

## Bootstrapping concepts

``` r
library(tidyverse)
library(moderndive)
library(infer)
library(socviz)
library(broom)
```

Remember, our estimate is based on a sample from some population, and each sample is going to give us a different estimate. This means our estimates will **vary** from sample to sample. How can we quantify this **variability**?

One approach is via **bootstrapping**, where we:

1.  Simulate many new datasets out of our original dataset
2.  Estimate the thing we want to estimate in each of those *bootstrapped* samples
3.  Look at the distribution of estimates across bootstrap samples

That distribution of bootstrapped estimates gives us a sense for how much an estimate might vary from sample to sample.

## Using the `infer` package

### Bootstrapping averages

Let’s do this with the `infer` package, to estimate the number of kids the average American has.

``` r
gss_sm %>%
  # specify the outcome variable
  specify(response = childs) %>%
  # generate the bootstrapped samples
  generate(reps = 1000, type = "bootstrap")
```

    ## Response: childs (numeric)
    ## # A tibble: 2,859,000 × 2
    ## # Groups:   replicate [1,000]
    ##    replicate childs
    ##        <int>  <dbl>
    ##  1         1      1
    ##  2         1      2
    ##  3         1      2
    ##  4         1      7
    ##  5         1      3
    ##  6         1      7
    ##  7         1      3
    ##  8         1      2
    ##  9         1      6
    ## 10         1      2
    ## # ℹ 2,858,990 more rows

Notice how each of the 1,000 bootstrapped samples has the same number of observations as the original dataset.

The above is equivalent to doing this 1,000 times:

``` r
gss_sm %>%
  # subset down to just kids
  select(childs) %>%
  # sample with replacement, same size as original dataset
  sample_n(nrow(gss_sm), replace = TRUE)
```

    ## # A tibble: 2,867 × 1
    ##    childs
    ##     <dbl>
    ##  1      0
    ##  2      0
    ##  3      3
    ##  4      4
    ##  5      3
    ##  6      1
    ##  7      3
    ##  8      2
    ##  9      3
    ## 10      1
    ## # ℹ 2,857 more rows

We can then calculate the average number of kids in each of those 1,000 bootstraps:

``` r
boot_kids = gss_sm %>%
  # specify the outcome variable
  specify(response = childs) %>%
  # generate the bootstrapped samples
  generate(reps = 1000, type = "bootstrap") %>%
  # find the average # of kids in each bootstrap sample
  calculate(stat = "mean")

boot_kids
```

    ## Response: childs (numeric)
    ## # A tibble: 1,000 × 2
    ##    replicate  stat
    ##        <int> <dbl>
    ##  1         1  1.80
    ##  2         2  1.83
    ##  3         3  1.89
    ##  4         4  1.84
    ##  5         5  1.86
    ##  6         6  1.80
    ##  7         7  1.79
    ##  8         8  1.79
    ##  9         9  1.89
    ## 10        10  1.85
    ## # ℹ 990 more rows

`stat` is the average number of kids in each bootstrap (`replicate`).

We can look at the distribution to get a sense for the variability in the estimates:

``` r
ggplot(boot_kids, aes(x = stat)) +
  geom_density(color = "white", fill = "coral", alpha = .7) +
  theme_bw() +
  labs(title = "Average number of kids across bootstraps",
       x = NULL, y = NULL) +
  scale_x_continuous(limits = c(1, 3))
```

<img src="/class/bootstrap_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Notice how almost all bootstrapped estimates of the average number of kids are between 1.75 and 2, and the vast majority are in a narrower range than that.

We can also **quantify** this variation by taking the standard deviation of `stat`:

``` r
boot_kids %>%
  summarise(se = sd(stat))
```

    ## # A tibble: 1 × 1
    ##       se
    ##    <dbl>
    ## 1 0.0311

This is also known as the **standard error**.

#### Variability gets bigger as sample size gets smaller

The process above gives us a sense of the variability in our estimate of the average number of kids in the US, based on our survey of `\(\approx\)` 2,800 people. What if we had a much smaller survey? Say 100 people?

We can mimic that below by taking 100 random people from `gss_sm` and pretending that’s our full survey:

``` r
boot_kids = gss_sm %>%
  # smaller survey of only 100 people
  sample_n(100) %>%
  # specify the outcome variable
  specify(response = childs) %>%
  # generate the bootstrapped samples
  generate(reps = 1000, type = "bootstrap") %>%
  # find the average # of kids in each bootstrap sample
  calculate(stat = "mean")
```

Notice how much wider this distribution is than the one above. The variability in our bootstrapped estimates is much higher!

``` r
ggplot(boot_kids, aes(x = stat)) +
  geom_density(color = "white", fill = "coral", alpha = .7) +
  theme_bw() +
  labs(title = "Average number of kids across bootstraps",
       x = NULL, y = NULL) +
  scale_x_continuous(limits = c(1, 3))
```

<img src="/class/bootstrap_files/figure-html/unnamed-chunk-8-1.png" width="672" />

You can quantify this too; notice how much bigger the standard error is of this much smaller survey:

``` r
boot_kids %>%
  summarise(se = sd(stat))
```

    ## # A tibble: 1 × 1
    ##      se
    ##   <dbl>
    ## 1 0.166

### Bootstrapping regression coefficients

We can use bootstrapping to approximate the variability in lots of things we might want to estimaet, including coefficients from regression.

Say we wanted to look at the effect of age on whether or not someone has children:

``` r
lm(childs ~ age, data = gss_sm) %>%
  tidy()
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)   0.153    0.0858       1.79 7.40e- 2
    ## 2 age           0.0345   0.00164     21.0  1.83e-91

Remember this estimate comes from a *sample*. Other samples will give us different estimates. We can get a sense for how much coefficients will vary across samples with bootstrapping:

``` r
boot_age = gss_sm %>%
  # specify is now Y ~ X
  specify(childs ~ age) %>%
  generate(reps = 1000, type = "bootstrap") %>%
  # "slope" is another word for coefficient
  calculate(stat = "slope")
```

Notice how the code looks a bit different: we put `childs ~ age` in `specify()` and we gotta tell R to calculate the “slope”, or coefficient, from `lm(childs ~ age)`.

We can plot:

``` r
ggplot(boot_age, aes(x = stat)) +
  geom_density(color = "white", fill = "coral", alpha = .7) +
  theme_bw() +
  labs(title = "Estimate of relationship between age and number of kids",
       subtitle = "Coefficient estimates across bootstrapped samples.",
       x = NULL, y = NULL)
```

<img src="/class/bootstrap_files/figure-html/unnamed-chunk-12-1.png" width="672" />

Notice how coefficient estimates vary a lot, from .03 to almost .04.

## Standard errors and confidence intervals

The distributions we get from bootstrapping give us a sense for the **variability** in our sample estimates. We can quantify that variability in two ways:

One is through the standard error (the standard deviation of the bootstrapped sample estimates):

``` r
boot_age %>%
  summarise(se = sd(stat))
```

    ## # A tibble: 1 × 1
    ##        se
    ##     <dbl>
    ## 1 0.00158

The other is the confidence interval, which provides our “best guess” of the thing we are trying to estimate:

``` r
boot_age %>% get_confidence_interval(level = .95)
```

    ## # A tibble: 1 × 2
    ##   lower_ci upper_ci
    ##      <dbl>    <dbl>
    ## 1   0.0315   0.0377

The standard is 95%: so the two numbers give us the upper and lower bound of the middle 95% of the bootstrapped distribution.

Notice that as the confidence increases, the bounds get larger:

``` r
boot_age %>% get_confidence_interval(level = .99)
```

    ## # A tibble: 1 × 2
    ##   lower_ci upper_ci
    ##      <dbl>    <dbl>
    ## 1   0.0307   0.0390

As the confidence decreases, the bounds get smaller:

``` r
boot_age %>% get_confidence_interval(level = .5)
```

    ## # A tibble: 1 × 2
    ##   lower_ci upper_ci
    ##      <dbl>    <dbl>
    ## 1   0.0335   0.0356

## Hypothesis testing

Remember, hypothesis testing is about how we decide between two competing hypotheses – or theories about the relationship between two variables – using data.

Take the example from class, on whether [bicep size predicts conservative ideology](https://www.theatlantic.com/health/archive/2013/05/study-mens-biceps-predict-their-political-ideologies/275942/). The two competing hypotheses are:

1.  Null hypothesis: there is no relationship between biceps and conservative ideology
2.  Alternative hypothesis: there is a positive relationship between biceps and conservative ideology

We make up fake data below:

``` r
set.seed(23424)
# fake bicep data
fake = tibble(person = 1:100,
              bicep = rnorm(100),
              conservative = runif(100)*100 + 2*bicep)
```

We estimate the relationship between bicep and ideology:

``` r
# what is the effect?
lm(conservative ~ bicep, data = fake) %>% summary()
```

    ## 
    ## Call:
    ## lm(formula = conservative ~ bicep, data = fake)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -45.688 -24.505  -5.035  28.053  54.741 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   46.839      2.892  16.197   <2e-16 ***
    ## bicep          3.996      2.754   1.451     0.15    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 28.92 on 98 degrees of freedom
    ## Multiple R-squared:  0.02103,	Adjusted R-squared:  0.01104 
    ## F-statistic: 2.105 on 1 and 98 DF,  p-value: 0.15

Remember, this estimate is based on one sample. How much might estimates vary? We can use bootstrapping to get a sense:

``` r
boot_bicep = fake %>%
  specify(conservative ~ bicep) %>%
  generate(reps = 1000, type = "bootstrap") %>%
  calculate(stat = "slope")


ggplot(boot_bicep, aes(x = stat)) +
  geom_density(color = "white", fill = "coral", alpha = .7) +
  theme_bw() +
  labs(title = "Estimate of relationship between bicep size and ideology score",
       subtitle = "Coefficient estimates across bootstrapped samples.",
       x = NULL, y = NULL)
```

<img src="/class/bootstrap_files/figure-html/unnamed-chunk-19-1.png" width="672" />

Notice how much the coefficient can vary: in some cases as large as 10 or more, in others 0, and in some cases even negative.

How do we decide, based on this distribution, whether there is in fact that a stable relationship between biceps and ideology?

### Hypothesis testing: permutation

One way to decide is to simulate what the world might look like under the null hypothesis – a world where biceps don’t affect ideology.

We can do this by randomly “shuffling” or *permuting* the values in our bicep variable. Why? This mimics the idea that if bicep size and ideology are unrelated, you could completely alter one variable without affecting the other. So we:

1.  “shuffle” or permute the bicep variable
2.  estimate `lm(conservative ~ bicep)`, store coefficient on bicep
3.  repeat N times

``` r
# permutation based hypothesis testing
null_biceps = fake %>%
  # specify is now Y ~ X
  specify(conservative ~ bicep) %>%
  # define null hypothesis: "independence"
  hypothesize(null = "independence") %>%
  # now we do "permute" instead of bootstrap
  generate(reps = 1000, type = "permute") %>%
  # "slope" is another word for coefficient
  calculate(stat = "slope")

null_biceps
```

    ## Response: conservative (numeric)
    ## Explanatory: bicep (numeric)
    ## Null Hypothesis: independence
    ## # A tibble: 1,000 × 2
    ##    replicate   stat
    ##        <int>  <dbl>
    ##  1         1  3.60 
    ##  2         2  3.28 
    ##  3         3  1.73 
    ##  4         4  0.390
    ##  5         5  2.15 
    ##  6         6  5.20 
    ##  7         7 -5.17 
    ##  8         8 -4.43 
    ##  9         9 -2.34 
    ## 10        10  1.15 
    ## # ℹ 990 more rows

Each row here is the estimated coefficient of bicep from a permuted sample – a sample where the values of bicep have been shuffled. Look at the distribution:

``` r
ggplot(null_biceps, aes(x = stat)) + geom_histogram(color = "white",
                                                     fill = "coral") +
  theme_bw() +
  labs(title = "The Simulated Null Distribution",
       subtitle = "What we would observe in a world where biceps and ideology are unrelated.",
       x = "Permuted coefficient estimates from lm(affairs ~ children)",
       y = NULL)
```

<img src="/class/bootstrap_files/figure-html/unnamed-chunk-21-1.png" width="672" />

This is our simulation of what we might observe under the *null hypothesis* – that there is no relationship between biceps and ideology. Two key takeaways here:

1.  As we would expect, most of our simulated coefficient estimates are close to zero. This makes sense! The null hypothesis is precisely that there is no relationship between the two (e.g., that the coefficient is not greater than zero!).

2.  Even in a world where the null is true, you can still get pretty large coefficient estimates by chance, even as large as -10 and +10.

Point (2) is really important: totally by chance, two variables that are unrelated can still have strong positive or negative correlations. It is *unlikely that this happens, but it’s still possible*.

With a simulation of what the world would like under the null hypothesis, we can turn back to our original estimate:

``` r
lm(conservative ~ bicep, data = fake) %>% moderndive::get_regression_table()
```

    ## # A tibble: 2 × 7
    ##   term      estimate std_error statistic p_value lower_ci upper_ci
    ##   <chr>        <dbl>     <dbl>     <dbl>   <dbl>    <dbl>    <dbl>
    ## 1 intercept    46.8       2.89     16.2     0       41.1     52.6 
    ## 2 bicep         4.00      2.75      1.45    0.15    -1.47     9.46

How likely is it that our estimate would have occurred by chance?

``` r
ggplot(null_biceps, aes(x = stat)) + geom_histogram(color = "white",
                                                     fill = "coral") +
  theme_bw() +
  labs(title = "The Simulated Null Distribution",
       subtitle = "What we would observe in a world where biceps and ideology are unrelated.",
       x = "Permuted coefficient estimates from lm(affairs ~ children)",
       y = NULL) +
  geom_vline(xintercept = 3.996, lty = 2, color = "black", size = 2)
```

<img src="/class/bootstrap_files/figure-html/unnamed-chunk-23-1.png" width="672" />

Our estimate is a fair bit larger than what you would expect to observe under the null hypothesis. We can calculate exactly how much larger using the **p-value**. We set `direction = right` because we want to know how much *larger* the estimate is than expected.

``` r
get_p_value(null_biceps, obs_stat = 3.996, direction = "both")
```

    ## # A tibble: 1 × 1
    ##   p_value
    ##     <dbl>
    ## 1   0.168

The **p-value** is .084, meaning that only 8.4% of the estimates we got by chance are as large or larger than our original result. In other words, our estimate is larger than 84% of the results we could observe by chance.

We can calculate this “by hand” too, like so:

``` r
null_biceps %>%
  mutate(bigger = ifelse(stat > 3.996, "yes", "no")) %>%
  group_by(bigger) %>%
  tally() %>%
  mutate(percent = n/sum(n))
```

    ## # A tibble: 2 × 3
    ##   bigger     n percent
    ##   <chr>  <int>   <dbl>
    ## 1 no       916   0.916
    ## 2 yes       84   0.084

84% larger than we might observe by chance – sounds like our result is unusually large, and unlikely to be the result of random chance. But is it unlikely enough for us to be confident?

### Alpha-levels (standards of evidence)

Remember the standard and totally arbitrary *alpha-level* accepted in the scientific community is .05. This means that p-values lower than .05 are considered too small to be the result of random chance, while p-values higher than that are considered too large to feel confident in. Arbitrary? Yes.

Since our p-value of .084 is larger than the .05 standard (the alpha-level), we would *fail to reject the null hypothesis*. In other words, the result is too plausible in a world where null hypothesis is true for us to reject it!

### Hypothesis testing: confidence intervals

A different, and much more straightforward way, to do hypothesis testing is to bootstrap confidence intervals and see if they exclude 0. Let’s do the bootstrap.

``` r
# boot_bicep = fake %>%
#   specify(conservative ~ bicep) %>%
#   generate(reps = 1000, type = "boot") %>%
#   calculate(stat = "slope")
```

Here is the distribution of bootstrapped estimates from our data:

``` r
ggplot(boot_bicep, aes(x = stat)) + geom_histogram(color = "white",
                                                     fill = "coral") +
  theme_bw() +
  labs(title = "Distribution of bootstrapped estimates",
       subtitle = "How the relationship between biceps and ideology might vary across samples.",
       x = "Bootstrapped coefficient estimates from lm(conservative ~ bicep)",
       y = NULL)
```

<img src="/class/bootstrap_files/figure-html/unnamed-chunk-27-1.png" width="672" />

Here’s the standard error:

``` r
boot_bicep %>%
  summarise(se = sd(stat))
```

    ## # A tibble: 1 × 1
    ##      se
    ##   <dbl>
    ## 1  2.85

Here’s the 95% confidence interval:

``` r
get_confidence_interval(boot_bicep, level = .95)
```

    ## # A tibble: 1 × 2
    ##   lower_ci upper_ci
    ##      <dbl>    <dbl>
    ## 1    -1.71     9.55

Notice this interval *includes* 0. This means that the range of values that are our “best guess” for the effect of biceps on ideology **include** the possibility that there is no effect (or that the effect is negative). Following the standard of using .05 as the threshold, we would *fail to reject the null hypothesis of no effect of biceps on ideology*.

Here’s the confidence interval again, with shading:

``` r
ggplot(boot_bicep, aes(x = stat)) + geom_histogram(color = "white",
                                                     fill = "coral") +
  theme_bw() +
  labs(title = "Distribution of bootstrapped estimates",
       subtitle = "How the relationship between biceps and ideology might vary across samples.",
       x = "Bootstrapped coefficient estimates from lm(conservative ~ bicep)",
       y = NULL) +
  annotate(geom = "rect", xmin = get_confidence_interval(boot_bicep)[["lower_ci"]],
           xmax = get_confidence_interval(boot_bicep)[["upper_ci"]], ymin = 0, ymax = Inf,
           fill = "#2ECC40", alpha = 0.25)
```

<img src="/class/bootstrap_files/figure-html/unnamed-chunk-30-1.png" width="672" />

## Alpha-levels: tradeoffs

Remember, the (abitrary) convention for deciding between competing hypotheses is a p-value less than .05; that .05 threshold is known as the `alpha-level`. An important thing to remember is that there is a trade-off in the size of the alpha-level:

- the bigger the alpha-level, the lower our standard of evidence for rejecting the null, which means a lower chance of Type 2 error (e.g., letting the guilty go free) and a higher chance of Type 1 error (e.g., convicting the innocent)

- the smaller the alpha-level, the higher our standard of evidence for rejecting the null, which means a higher chance of Type 2 error and a lower chance of Type 1 error

Let’s use simulation to see this. Let’s pretend once again that `gss_sm` is the whole of the American public. We want to estimate the effect of age on children in this population. H1 is that people who are older are more likely to have children than people who are younger. H0 (the null) is that people who are older don’t have children at higher rates than people who are younger.

We can know the exact effect of age on children in this made-up example:

``` r
lm(childs ~ age, data = gss_sm) %>%
  tidy()
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)   0.153    0.0858       1.79 7.40e- 2
    ## 2 age           0.0345   0.00164     21.0  1.83e-91

The true effect size is .0345. So we *should* reject the null, since the coefficient is not zero (or less than zero).

Now, say we have a random sample of 200 people from this population. We can take a sample, estimate 95% confidence interval below:

``` r
gss_sm %>%
  drop_na() %>%
  sample_n(200) %>%
  specify(childs ~ age) %>%
  generate(reps = 1000, type = "bootstrap") %>%
  calculate(stat = "slope") %>%
  get_confidence_interval()
```

    ## # A tibble: 1 × 2
    ##   lower_ci upper_ci
    ##      <dbl>    <dbl>
    ## 1   0.0143   0.0373

What if we do this many times? How many times will our confidence interval

``` r
get_ci = function(x)
{
  ci = x %>%
  specify(childs ~ age) %>%
  generate(reps = 1000, type = "bootstrap") %>%
  calculate(stat = "slope") %>%
  get_confidence_interval()

  return(ci)
}

gss_sm %>%
  select(childs, age) %>%
  rep_sample_n(size = 200, reps = 100, replace = FALSE) %>%
  nest() %>%
  mutate(cis = map(data, get_ci))
```

    ## # A tibble: 100 × 3
    ## # Groups:   replicate [100]
    ##    replicate data               cis             
    ##        <int> <list>             <list>          
    ##  1         1 <tibble [200 × 2]> <tibble [1 × 2]>
    ##  2         2 <tibble [200 × 2]> <tibble [1 × 2]>
    ##  3         3 <tibble [200 × 2]> <tibble [1 × 2]>
    ##  4         4 <tibble [200 × 2]> <tibble [1 × 2]>
    ##  5         5 <tibble [200 × 2]> <tibble [1 × 2]>
    ##  6         6 <tibble [200 × 2]> <tibble [1 × 2]>
    ##  7         7 <tibble [200 × 2]> <tibble [1 × 2]>
    ##  8         8 <tibble [200 × 2]> <tibble [1 × 2]>
    ##  9         9 <tibble [200 × 2]> <tibble [1 × 2]>
    ## 10        10 <tibble [200 × 2]> <tibble [1 × 2]>
    ## # ℹ 90 more rows
