---
title: Importing data into R
menu:
  reference:
    parent: Guides
type: docs
weight: 2
output:
  blogdown::html_page:
      toc: TRUE
---


Most of the data in this course will come from packages. I do this so we have more time to focus on the stuff that matters. But most of the data in the world is not in a package. How can I read this kind of data into R?


There's ways to do this in Rstudio through [drop-down menus](https://support.rstudio.com/hc/en-us/articles/218611977-Importing-Data-with-the-RStudio-IDE) but I call that the "no learning" approach. 


To read data into R you need to know two things:


1) What kind of file is it? What is its *extension*?
2) Where is it? What is its *file path*?



## What kind of file is it?


There are many different types of files out there. You can tell what type of file a dataset is by looking at its **extension** - the bit in the file name that comes after the period. Some common file types/extensions are `.csv`, `.dta`, `.xlsx`, etc.


R has different functions you can use to import different types of files. For a `.csv` file you would use `read_csv()` or `read.csv()`. For a `.dta` you would use `read_dta()` from the `{haven}` package. For an `.xlsx` file you would use `read_excel()` from `{readxl}`. If you're not sure, google what function to use for a particular data file. 


## Where is it?


This part is tricky for people who don't have a lot of experience working with computers. But, in short: every file that is on your computer has a "path" or "address" that uniquely identifies it on your computer. This address changes with your file's location; if you moved the file for whatever reason, the file path would change. You need the file path in order to tell R where to look at your data. 


How to find it? Operating systems are always changing so in general the best thign is to google how to find a file's filepath and then remember it. But the following seems to have worked on Mac for a long time now:

- find the file in your Finder
- right-click and select "get info"
- at the top left, under "General", you'll see the file path next to "Where:"
- right-click that path and select "copy as Pathname"


And on Windows you can [do this](https://www.howtogeek.com/670447/how-to-copy-the-full-path-of-a-file-on-windows-10/). 


Code to import data into R might look like this: 



```r
df = read_csv("/Users/juan/Dropbox/teaching/co2-data/owid-co2-data.csv")
```


## How to do this better


For working on longer term projects, it is more convenient to use *relative* file paths in conjunction with something like R projects (`.Rproj`) + the `{here}` library. 


