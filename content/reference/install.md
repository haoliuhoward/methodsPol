---
title: Installing R, RStudio, packages
menu:
  reference:
    parent: Guides
type: docs
weight: 1
output:
  blogdown::html_page:
      toc: TRUE
---



You will do all of your work in this class with the open source (and free!) programming language [R](https://cran.r-project.org/). You will use [RStudio](https://www.rstudio.com/) as the main program to access R. Think of R as an engine and RStudio as a car dashboard—R handles all the calculations and the actual statistics, while RStudio provides a nice interface for running R code.



# Install R

First you need to install R itself (the engine).

1. If you have a **WINDOWS**, click here: [Download R 4.2.1 for Windows](https://cran.r-project.org/bin/windows/base/R-4.2.1-win.exe)
2. If you have a **MAC**, click here: [Download R 4.2.1 for macOS](https://cran.r-project.org/bin/macosx/base/R-4.2.1.pkg)
3. Double click on the downloaded file (check your `Downloads` folder). Click yes through all the prompts to install like any other program.

4. If you use macOS, [download and install XQuartz](https://www.xquartz.org/). You do not need to do this on Windows.


# Install RStudio

Next, you need to install RStudio, the nicer graphical user interface (GUI) for R (the dashboard). Once R and RStudio are both installed, you can ignore R and only use RStudio. RStudio will use R automatically and you won't ever have to interact with R directly.

1. If you have a **WINDOWS**, click here: [Download RStudio for Windows](https://download1.rstudio.org/desktop/windows/RStudio-2022.07.1-554.exe)
2. If you have a **MAC**, click here: [Download RStudio for macOS](https://download1.rstudio.org/desktop/macos/RStudio-2022.07.1-554.dmg)
3. Double click on the downloaded file (check your `Downloads` folder). Click yes through all the prompts to install like any other program.

Double click on RStudio to run it (check your applications folder or start menu).



## Extra installation help (Mac): steps

Here's a different [helpful walkthrough for Mac](https://scribehow.com/shared/Google_Workflow__ivjLMJ3_SIaV9LDOwaxQHQ)



## Extra installation help (Windows): video


Here's a different, [helpful walkthrough for Windows](https://www.youtube.com/watch?v=TFGYlKvQEQ4)


<iframe width="560" height="315" src="https://www.youtube.com/embed/TFGYlKvQEQ4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


# Install packages

Most R packages are easy to install with RStudio. Select the packages panel, click on "Install," type the name of the package you want to install, and press enter. R will download the package from the web, so make sure you are connected to wifi when you do it. 


<img src="../../../../../img/install/install-r-package-panel.png" width="40%" />


A less tedious way to do this is via the console or in your script (just make sure to delete afterwards!), by running the following code:


```r
install.packages("name_of_package")
```


## Install `{juanr}` (and other packages from Github)


Some packages, like `{juanr}` cannot be installed using `install.packages()` because they are hosted on [Github](https://github.com/). To install `{juanr}`, you will first need to install `{remotes}` and then use `install_github()`, like so:



```r
install.packages("remotes")
remotes::install_github("hail2thief/juanr")
```


