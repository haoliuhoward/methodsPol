---
title: How to scrape websites
menu:
  reference:
    parent: Guides
type: docs
weight: 4
output:
  blogdown::html_page:
      toc: TRUE
---






## Intro



```r
library(tidyverse)
library(rvest)
library(janitor)
```


There's a lot of information on the internet. Sometimes this information is nicely formatted, which means we can scrape it from the internet fairly easily. 

Take a look at the table on this page of Simpsons guest star appearances: https://en.wikipedia.org/wiki/List_of_The_Simpsons_guest_stars


We're gonna pull this table from the internet into R. You'll need the `rvest` and `janitor` packages. Install if you don't have them.



## Pulling the table

First step is to pull down the whole Wikipedia page. To do so, use the `read_html` function, putting the URL of the site we want to scrape inside of it (in quotation marks!). Assign this to an object named `content`. 


```r
df = read_html("https://en.wikipedia.org/wiki/List_of_The_Simpsons_guest_stars")
```


Now we have the whole page. We just want that table. Run the `html_table` function on `content` and store that in an object called `table`. Add `fill = TRUE` within the function otherwise you'll get an error. 



```r
table = html_table(df, fill = TRUE)
```

Notice up top in your environment that you have an object called `table` that is a list with 13 elements. That means we have 13 tables from that page. But we only want the one with guest stars! Which one is it? 

We need to look at the elements in that list to figure out which of the 13 tables is ours. To look at a specific element in a list, we can use the `pluck()` function, like so:



```r
table %>% pluck(1)
```

```
## # A tibble: 1 × 1
##   X1                                                                            
##   <chr>                                                                         
## 1 Seasons: 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 • Movie • 19 20 21 22 2…
```

The first element in our list is not what we want; what about the second? 



```r
table %>% pluck(2)
```

```
## # A tibble: 746 × 6
##    Season `Guest star`             `Role(s)`  No.   `Prod. code` `Episode title`
##     <int> <chr>                    <chr>      <chr> <chr>        <chr>          
##  1     21 Matt Groening            Himself    442–… LABF13       "\"Homer the W…
##  2     21 Kevin Michael Richardson Security … 442–… LABF13       "\"Homer the W…
##  3     21 Seth Rogen               Lyle McCa… 442–… LABF13       "\"Homer the W…
##  4     21 Marcia Wallace           Edna Krab… 443–… LABF15       "\"Bart Gets a…
##  5     21 Chuck Liddell            Himself    444–… LABF16       "\"The Great W…
##  6     21 Marcia Wallace           Edna Krab… 444–… LABF16       "\"The Great W…
##  7     21 Marcia Wallace           Edna Krab… 445–… LABF14       "\"Treehouse o…
##  8     21 Marcia Wallace           Edna Krab… 446–… LABF17       "\"The Devil W…
##  9     21 Jonah Hill               Andy Hami… 447–… LABF18       "\"Pranks and …
## 10     21 Marcia Wallace           Edna Krab… 447–… LABF18       "\"Pranks and …
## # ℹ 736 more rows
```

That's what we want. Let's assign that as an object:



```r
simpsons = table %>% 
  pluck(2)
```


## Cleaning up the data and making the plot

The column names of this table are hard to work with. Let's use the `clean_names` function on our table and assign that to another object called `clean_simpsons`. 



```r
clean_simpsons = 
  simpsons %>% clean_names()
```

Finally, we can use our tidyverse know-how to calculate how many times each Guest Star has appeared on the Simpson's, and filter the data down to just those who have appeared more than twice. We can then make a plot showing how many times each of these guest stars has appeared on the show. 


```r
plot_simpsons = clean_simpsons %>% 
  group_by(guest_star) %>% 
  tally() %>% 
  filter(n > 2)

ggplot(plot_simpsons, aes(x = reorder(guest_star, n), y = n)) + 
  geom_col() + 
  coord_flip()
```

<img src="/reference/scraping-websites_files/figure-html/unnamed-chunk-8-1.png" width="672" />



Done! Here are some other good ones to try: 

- https://en.wikipedia.org/wiki/List_of_nicknames_used_by_Donald_Trump
- https://en.wikipedia.org/wiki/List_of_helicopter_prison_escapes
- https://en.wikipedia.org/wiki/Passengers_of_the_Titanic
- https://en.wikipedia.org/wiki/Last_meal
- https://en.wikipedia.org/wiki/List_of_consorts_of_the_Ottoman_sultans
- https://en.wikipedia.org/wiki/List_of_people_who_died_climbing_Mount_Everest
