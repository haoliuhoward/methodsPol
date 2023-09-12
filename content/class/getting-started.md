---
title: "R, Rstudio, basics"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  class:
    parent: Class
    weight: 1
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

## In-class example

Here’s the code we’ll be using in class. Download it and store it with the rest of your materials for this course. If simply clicking doesn’t trigger download, you should right-click and select “save link as…”.

[<i class="fas fa-file-archive"></i> `UN Voting Data`](/files/eat-cake.R)

## Example: UN voting patterns

Your first class script `eat-cake.R` does two things:

- takes a dataset that records every vote at the UN going back decades and cleans it
- visualizes voting patterns over time for the US and Turkey

At this point, none of this should “make sense” to you. You do not need to understand what each line of code here does. I just want you to run the code and try to make sense of what the different parts are doing.

Let’s break the script down:

The first thing the script does is load a set of packages, or libraries, which will give us access to the UN data and functions to use with that data.

``` r
library(tidyverse)
library(unvotes) # this is where the UN data lives
library(lubridate)
library(scales)
```

We can see some of the UN data below:

``` r
un_votes %>%
  slice(1:5) %>%
  knitr::kable()
```

| rcid | country            | country_code | vote |
|-----:|:-------------------|:-------------|:-----|
|    3 | United States      | US           | yes  |
|    3 | Canada             | CA           | no   |
|    3 | Cuba               | CU           | yes  |
|    3 | Haiti              | HT           | yes  |
|    3 | Dominican Republic | DO           | yes  |

The variable `rcid` is the way the data identifies the issue being voted on. So on RCID 3, the US voted “yes” while Canada voted “no”.

Here are some of the issues:

``` r
un_roll_calls %>%
  select(rcid, date, short) %>%
  slice(1:5) %>%
  knitr::kable()
```

| rcid | date       | short                          |
|-----:|:-----------|:-------------------------------|
|    3 | 1946-01-01 | AMENDMENTS, RULES OF PROCEDURE |
|    4 | 1946-01-02 | SECURITY COUNCIL ELECTIONS     |
|    5 | 1946-01-04 | VOTING PROCEDURE               |
|    6 | 1946-01-04 | DECLARATION OF HUMAN RIGHTS    |
|    7 | 1946-01-02 | GENERAL ASSEMBLY ELECTIONS     |

The next chunk of code takes the UN voting data and calculates the percentage of times the US and Turkey voted “yes” on an issue in each year for which there is data.

``` r
un_yes = un_votes %>%
  filter(country %in% c("United States", "Turkey")) %>%
  inner_join(un_roll_calls, by = "rcid") %>%
  inner_join(un_roll_call_issues, by = "rcid") %>%
  group_by(country, year = year(date), issue) %>%
  summarize(
    votes = n(),
    percent_yes = mean(vote == "yes")
  ) %>%
  filter(votes > 5)  # only use records where there are more than 5 votes
```

You can see the results of this below:

``` r
un_yes %>%
  head() %>%
  knitr::kable()
```

| country | year | issue                        | votes | percent_yes |
|:--------|-----:|:-----------------------------|------:|------------:|
| Turkey  | 1946 | Colonialism                  |    15 |   0.8000000 |
| Turkey  | 1946 | Economic development         |    10 |   0.6000000 |
| Turkey  | 1947 | Colonialism                  |     9 |   0.2222222 |
| Turkey  | 1947 | Palestinian conflict         |     7 |   0.1428571 |
| Turkey  | 1948 | Colonialism                  |    12 |   0.4166667 |
| Turkey  | 1948 | Arms control and disarmament |     9 |   0.0000000 |

So in 1946 Turkey voted 15 times on issues related to “Colonialism”, and of those votes, 80% were a “yes”.

This last bit of code produces the visualization:

``` r
ggplot(un_yes, aes(x = year, y = percent_yes, color = country)) +
  geom_point() +
  geom_smooth(method = "loess", se = FALSE) +
  facet_wrap(~ issue) +
  labs(
    title = "Percentage of 'Yes' votes in the UN General Assembly",
    subtitle = "1946 to 2015",
    y = "% Yes",
    x = "Year",
    color = "Country"
  ) +
  scale_y_continuous(labels = percent)
```

<img src="/class/getting-started_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Try playing around with the code! What happens when you replace the United States and/or Turkey with another country?
