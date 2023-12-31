---
title: "Uncertainty"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 11
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

# In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”.

[<i class="fas fa-file-archive"></i> `day1-uncertainty.R`](/slides/code/day1-uncertainty.R)

# Other examples

``` r
library(tidyverse)
library(broom)
library(socviz)
library(moderndive)

set.seed(1990)
```

## Why are we uncertain?

Remember that with data we are always trying to estimate something: the effect of X on Y, the average value of Y among some population, etc, – but we can be more or less *certain* about our estimate.

A key reason we are uncertain is that our data is usually a *sample* of something bigger that we want to learn about (that bigger thing = the *population*). The problem is that each sample is going to give us a (slightly) different answer.

For example, below, imagine that in the US there were only `\(\approx\)` 2,800 people, and they were all included in the `gss_sm` dataset from `socviz`.

What proportion of men and women voted for Obama in 2012? In this made up example, we can know the *exact answer*:

``` r
true_prop = gss_sm %>% 
  group_by(sex) %>% 
  summarise(obama = mean(obama, na.rm = TRUE))
true_prop
```

    ## # A tibble: 2 × 2
    ##   sex    obama
    ##   <fct>  <dbl>
    ## 1 Male   0.576
    ## 2 Female 0.663

Now, imagine that we instead only had a *sample* of `gss_sm.` Say, we picked 10 random people from `gss_sm` and calculated the proportion of support among men and women for Obama.

``` r
gss_sm %>% 
  # randomly draw 10 observations
  sample_n(10) %>% 
  group_by(sex) %>% 
  summarise(obama = mean(obama, na.rm = TRUE))
```

    ## # A tibble: 2 × 2
    ##   sex    obama
    ##   <fct>  <dbl>
    ## 1 Male    0.25
    ## 2 Female  1

Run the code above over and over. Notice: you get a different answer each time! Some very far from the true proportion of men and women who voted from Obama. This is the big problem: we only have a sample and each sample gives us a different answer.[^1] How do we know that our sample is *close* to the true answer and not weirdly off?

## Become less uncertain

It turns out that if a sample is *random, representative, and large*, our estimate from the **sample** will be pretty close to the estimate from the population.

Let’s see how this works: the code below uses the function `rep_sample_n` from `moderndive` to:

1.  Pick `N` random people from `gss_sm` (our sample)
2.  Estimate the proportion of men and women who voted for Obama in that sample (our sample estimate)
3.  Repeat this `P` times

Below, `N` = 10 and `P` = 10. So 10 different samples, each with 10 random people.

``` r
gss_sm %>% 
  # 10 samples, each of size 10
  rep_sample_n(size = 10, reps = 10) %>% 
  group_by(sex, replicate) %>% 
  summarise(obama = mean(obama, na.rm = TRUE)) %>% 
  arrange(replicate)
```

    ## # A tibble: 19 × 3
    ## # Groups:   sex [2]
    ##    sex    replicate   obama
    ##    <fct>      <int>   <dbl>
    ##  1 Female         1   0.571
    ##  2 Male           2 NaN    
    ##  3 Female         2   0.333
    ##  4 Male           3   0    
    ##  5 Female         3   0.5  
    ##  6 Male           4   0.5  
    ##  7 Female         4   0.5  
    ##  8 Male           5   0    
    ##  9 Female         5   1    
    ## 10 Male           6   0.5  
    ## 11 Female         6   0.6  
    ## 12 Male           7   0    
    ## 13 Female         7   1    
    ## 14 Male           8   0.667
    ## 15 Female         8   0.5  
    ## 16 Male           9   1    
    ## 17 Female         9   1    
    ## 18 Male          10   0.5  
    ## 19 Female        10   0.5

Notice how the sample estimate differ varies across samples (the `replicate` variable tells you which sample you are looking at).

Let’s do this many more times, and plot the distribution of our sample estimates. Below, each sample only has 10 people in it, and we take 1,000 samples. The black bars are the *proportion of men and women who voted for Obama across all of gss_sm* (the population parameter):

``` r
gss_sm %>% 
  rep_sample_n(size = 10, reps = 1000) %>% 
  group_by(replicate, sex) %>% 
  summarise(obama = mean(obama, na.rm = TRUE)) %>% 
  ggplot(aes(x = obama, fill = sex)) + 
  geom_histogram(color = "white", alpha = .8) + 
  facet_wrap(vars(sex), scales = "free") + 
  theme_bw() + 
  geom_vline(data = true_prop, aes(xintercept = obama), 
             size = 2, color = "black")
```

<img src="/class/uncertainty_files/figure-html/unnamed-chunk-4-1.png" width="672" />

This is a sample that is *random and representative*, but small (only 10 people in each sample!). Notice how some of our sample estimates are far off from the population estimate: we get lots of cases where every man, for instance, voted for Obama, which is really wrong.

``` r
gss_sm %>% 
  rep_sample_n(size = 10, reps = 1000) %>% 
  group_by(replicate, sex) %>% 
  summarise(obama = mean(obama, na.rm = TRUE)) %>% 
  ggplot(aes(x = obama, fill = sex)) + 
  geom_histogram(color = "white", alpha = .8) + 
  facet_wrap(vars(sex), scales = "free") + 
  theme_bw() + 
  geom_vline(data = true_prop, aes(xintercept = obama), 
             size = 2, color = "black") + 
  scale_x_continuous(limits = c(-.1, 1.1))
```

<img src="/class/uncertainty_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Now look what happens when we make our samples bigger, for instance, where each sample is composed of 50 people:

``` r
gss_sm %>% 
  rep_sample_n(size = 50, reps = 1000) %>% 
  group_by(replicate, sex) %>% 
  summarise(obama = mean(obama, na.rm = TRUE)) %>% 
  ggplot(aes(x = obama, fill = sex)) + 
  geom_histogram(color = "white", alpha = .8) + 
  facet_wrap(vars(sex), scales = "free") + 
  theme_bw() + 
  geom_vline(data = true_prop, aes(xintercept = obama), 
             size = 2, color = "black")  + 
  scale_x_continuous(limits = c(0, 1)) + 
  scale_x_continuous(limits = c(-.1, 1.1))
```

<img src="/class/uncertainty_files/figure-html/unnamed-chunk-6-1.png" width="672" />

Now our estimates look very different: many of them are pretty close to the true population proportion. Let’s make our samples even bigger, with 100 people each:

``` r
gss_sm %>% 
  rep_sample_n(size = 100, reps = 1000) %>% 
  group_by(replicate, sex) %>% 
  summarise(obama = mean(obama, na.rm = TRUE)) %>% 
  ggplot(aes(x = obama, fill = sex)) + 
  geom_histogram(color = "white", alpha = .8) + 
  facet_wrap(vars(sex), scales = "free") + 
  theme_bw() + 
  geom_vline(data = true_prop, aes(xintercept = obama), 
             size = 2, color = "black")  + 
  scale_x_continuous(limits = c(0, 1)) + 
  scale_x_continuous(limits = c(-.1, 1.1))
```

<img src="/class/uncertainty_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Notice how much “narrower” the distribution is getting. We’re getting fewer and fewer samples that are far away from the black line. Let’s look at a sample of 500:

``` r
gss_sm %>% 
  rep_sample_n(size = 500, reps = 1000) %>% 
  group_by(replicate, sex) %>% 
  summarise(obama = mean(obama, na.rm = TRUE)) %>% 
  ggplot(aes(x = obama, fill = sex)) + 
  geom_histogram(color = "white", alpha = .8) + 
  facet_wrap(vars(sex), scales = "free") + 
  theme_bw() + 
  geom_vline(data = true_prop, aes(xintercept = obama), 
             size = 2, color = "black")  + 
  scale_x_continuous(limits = c(0, 1)) + 
  scale_x_continuous(limits = c(-.1, 1.1))
```

<img src="/class/uncertainty_files/figure-html/unnamed-chunk-8-1.png" width="672" />

Even closer! The result is that if you take a large, random, representative sample the vast majority of estimates are going to be pretty damn close to the population parameter.

## More data, more certain

What’s happening above is pretty intuitive; if you are estimating something, you should feel more confident about that estimate the more data you have.

Compare the two trend-lines below: we should be much more certain about the one on the left, even though both have the sample slope (2)!

``` r
# uncertainty example
low_certain = tibble(x = rnorm(10), 
                     y = 2*x + rnorm(10), 
                     certain = "low")

hi_certain = tibble(x = rnorm(1000), 
                     y = 2*x + rnorm(1000), 
                     certain = "high")


rbind(low_certain, hi_certain) %>% 
  ggplot(aes(x = x, y = y, color = certain)) + 
  geom_jitter(size = 2, width = 1) + 
  facet_wrap(vars(certain)) + 
  geom_smooth(method = "lm", color = "blue") + 
  theme_bw() + 
  theme(legend.position = "none")
```

<img src="/class/uncertainty_files/figure-html/unnamed-chunk-9-1.png" width="672" />

## Good and bad samples

Bigger samples decrease uncertainty in our estimate, but this all hinges on the sample being *good*. By good I mean the sample is random and representative of the population.

What makes a sample representative of the population? It’s easier to think about what makes a sample unrepresentative. Imagine, for whatever reason, that in the example above, *no one under 50 showed up in our sample*.

We can simulate this easily by just adding a call to `filter` below to exclude people below 50:

``` r
gss_sm %>% 
  filter(age >= 50) %>% 
  rep_sample_n(size = 500, reps = 1000) %>% 
  group_by(replicate, sex) %>% 
  summarise(obama = mean(obama, na.rm = TRUE)) %>% 
  ggplot(aes(x = obama, fill = sex)) + 
  geom_histogram(color = "white", alpha = .8) + 
  facet_wrap(vars(sex), scales = "free") + 
  theme_bw() + 
  geom_vline(data = true_prop, aes(xintercept = obama), 
             size = 2, color = "black")  + 
  scale_x_continuous(limits = c(0, 1)) + 
  scale_x_continuous(limits = c(-.1, 1.1))
```

<img src="/class/uncertainty_files/figure-html/unnamed-chunk-10-1.png" width="672" />

Notice how the distribution of sample estimates are no longer centered around the true population parameter (the black lines). They are centered elsewhere. Increasing the sample size will continue to make the distribution taller and skinnier, but it will not be centered around the black lines.

This is because our sample is *biased*. In the population, there are people of all sorts of ages. But in our sample, it’s only older people. A bigger sample will not solve this problem.

Why would our sample not have young people in it, even though the population does? These *sampling biases* happens in the real world all the time. Maybe younger people are less willing to answer polls, or to take surveys, or to have landlines, etc., that means they are “missing” at higher rates from samples.

[^1]: There are other sources of uncertainty (e.g., measurement error) which we will not discuss here.
