---
title: "Introduction to Data Cleaning with the `tidyverse`"
author: "A. Paxton (*University of Connecticut*)"
output:
  html_document:
    keep_md: yes
---

Data cleaning is a critical first step for working with many naturally occurring
datasets. Data cleaning (also known in as *data munging*) is an opportunity for
you to explore your data, identify systematic issues with specific variables,
address missing data, and generally ensure that your data are ready for analysis.

You may already be in the habit of inspecting your data with experimentally
derived datasets, as it's *always* a good habit to check out your data (no
matter their source) before working with them. However, it's a bit more
important (and, potentially, a bit trickier) when dealing with NODS, as you may
not entirely know what to expect in the NODS as you first begin using them
(e.g., reliability of the data logging or data collection, nature of the data
collection equipment).

In this tutorial, we will be using the `tidyverse` library in R to walk through
some data cleaning steps. Remember that every dataset is different, and the
steps that we will be presenting here are by no means comprehensive for every
project. It's also useful to note that there are many ways to clean your data,
including many functions in base R, but the Tidyverse is a suite of libraries
that is rapidly gaining popularity in the behavioral and cognitive sciences.

This tutorial is an expansion and refocusing of an earlier tutorial of mine
about [clustering in R](https://github.com/a-paxton/clustering-tutorial).

***

# Preliminaries

First, let's get ready for our analyses. We do this by clearing our workspace
and loading in the libraries we'll need. It's good to get in the habit of 
clearing your workspace from the start so that you don't accidentally have
clashes with unneeded variables or dataframes that could affect your results.

As with our other tutorials, we'll use a function here to check for all required
packages and---if necessary---install them before loading them. Implementing
this (or a similar) function is a helpful first step, especially if you plan on
sharing your code with other people.


```r
# clear the workspace (useful if we're not knitting)
rm(list=ls())
```




```r
# specify which packages we'll need
required_packages = c("tidyverse")

# install them (if necessary) and load them
load_or_install_packages(required_packages)
```

***

# Data preparation

***

## Load the data

Next, we'll load our data. For this first tutorial, we'll use a toy dataset
that's already included in `dplyr`: `starwars`. It includes information about
various Star Wars characters. This is a relatively simple naturally occurring
dataset culled from the [SWAPI](https://swapi.dev/), and it's fairly easy to
work with because it's already in R.


```r
# read in the dataset
starwars_df = as.data.frame(starwars)
```

We gave our variable an informative name---here, `starwars_df`. A common
convention is to use `df` as an abbreviation for "dataframe," or a collection of
variables. It's good to get in the habit of giving your variables and dataframes
more informative names than a single letter (e.g., `x`, `a`) or generic names
(e.g., `dataframe`) so that you can remember what you're handling at any moment.

(And don't forget: Future You will thank Past You when you've made easily
understandable variable names, too, rather than cursing Past You for making
Future You go through all of the code to figure out what `this_variable` is
doing and why.)

In our example, let's say that we want to examine whether pilots appear in more
*Star Wars* movies that non-pilots.

## Visual inspection

A good first step is to check out your dataset after you load it. You can use
the `head()` function to just look at the first 6 records (or however many you'd
like to specify manually).


```r
# inspect the first 6 records
head(starwars_df)
```

```
##             name height mass  hair_color  skin_color eye_color birth_year
## 1 Luke Skywalker    172   77       blond        fair      blue       19.0
## 2          C-3PO    167   75        <NA>        gold    yellow      112.0
## 3          R2-D2     96   32        <NA> white, blue       red       33.0
## 4    Darth Vader    202  136        none       white    yellow       41.9
## 5    Leia Organa    150   49       brown       light     brown       19.0
## 6      Owen Lars    178  120 brown, grey       light      blue       52.0
##      sex    gender homeworld species
## 1   male masculine  Tatooine   Human
## 2   none masculine  Tatooine   Droid
## 3   none masculine     Naboo   Droid
## 4   male masculine  Tatooine   Human
## 5 female  feminine  Alderaan   Human
## 6   male masculine  Tatooine   Human
##                                                                                                                                       films
## 1                                           The Empire Strikes Back, Revenge of the Sith, Return of the Jedi, A New Hope, The Force Awakens
## 2                    The Empire Strikes Back, Attack of the Clones, The Phantom Menace, Revenge of the Sith, Return of the Jedi, A New Hope
## 3 The Empire Strikes Back, Attack of the Clones, The Phantom Menace, Revenge of the Sith, Return of the Jedi, A New Hope, The Force Awakens
## 4                                                              The Empire Strikes Back, Revenge of the Sith, Return of the Jedi, A New Hope
## 5                                           The Empire Strikes Back, Revenge of the Sith, Return of the Jedi, A New Hope, The Force Awakens
## 6                                                                                     Attack of the Clones, Revenge of the Sith, A New Hope
##                             vehicles                starships
## 1 Snowspeeder, Imperial Speeder Bike X-wing, Imperial shuttle
## 2                                                            
## 3                                                            
## 4                                             TIE Advanced x1
## 5              Imperial Speeder Bike                         
## 6
```

You can also choose to inspect the data manually using RStudio's data viewer.
You can use the `View()` function to call it programmatically, although you'll
note that nothing will pop up when we knit such a command.


```r
# programmatically call the data viewer
View(starwars_df)
```

## Summary statistics

After our visual inspection, we might want to get a better feel for the
distributions of our data. We can use the `summary()` basic R commands to
understand all of our variables at a glance.


```r
# print summary statistics for all of our variables at once
summary(starwars_df)
```

```
##      name               height           mass          hair_color       
##  Length:87          Min.   : 66.0   Min.   :  15.00   Length:87         
##  Class :character   1st Qu.:167.0   1st Qu.:  55.60   Class :character  
##  Mode  :character   Median :180.0   Median :  79.00   Mode  :character  
##                     Mean   :174.4   Mean   :  97.31                     
##                     3rd Qu.:191.0   3rd Qu.:  84.50                     
##                     Max.   :264.0   Max.   :1358.00                     
##                     NA's   :6       NA's   :28                          
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##                                                                         
##   skin_color         eye_color           birth_year         sex           
##  Length:87          Length:87          Min.   :  8.00   Length:87         
##  Class :character   Class :character   1st Qu.: 35.00   Class :character  
##  Mode  :character   Mode  :character   Median : 52.00   Mode  :character  
##                                        Mean   : 87.57                     
##                                        3rd Qu.: 72.00                     
##                                        Max.   :896.00                     
##                                        NA's   :44                         
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##                                                                           
##     gender           homeworld           species         
##  Length:87          Length:87          Length:87         
##  Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character  
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##                                                          
##  films.Length  films.Class  films.Mode
##  5          -none-     character      
##  6          -none-     character      
##  7          -none-     character      
##  4          -none-     character      
##  5          -none-     character      
##  3          -none-     character      
##  3          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  6          -none-     character      
##  3          -none-     character      
##  2          -none-     character      
##  5          -none-     character      
##  4          -none-     character      
##  1          -none-     character      
##  3          -none-     character      
##  3          -none-     character      
##  1          -none-     character      
##  5          -none-     character      
##  5          -none-     character      
##  3          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  3          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  3          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  3          -none-     character      
##  3          -none-     character      
##  3          -none-     character      
##  2          -none-     character      
##  2          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  3          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  2          -none-     character      
##  2          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  1          -none-     character      
##  3          -none-     character      
##  vehicles.Length  vehicles.Class  vehicles.Mode
##  2          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  2          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  1          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  0          -none-     character               
##  starships.Length  starships.Class  starships.Mode
##  2          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  5          -none-     character                  
##  3          -none-     character                  
##  0          -none-     character                  
##  2          -none-     character                  
##  2          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  1          -none-     character                  
##  0          -none-     character                  
##  0          -none-     character                  
##  3          -none-     character
```

While `summary()` is a great quick function for this, let's take a look at some
`tidy` alternatives and begin our introduction to the Tidyverse. We can use the
`tidyverse::summarize()` function to compute a single value from a given
function---like, for example, a mean or standard deviation.


```r
# get summary stats for a single variable, removing NAs as needed
starwars_df %>%
  summarize(height_min = min(height, na.rm=TRUE),
            height_mean = mean(height, na.rm=TRUE),
            height_max = max(height, na.rm=TRUE),
            height_sd = sd(height, na.rm=TRUE))
```

```
##   height_min height_mean height_max height_sd
## 1         66     174.358        264  34.77043
```

Note that we're here using this symbol: `%>%`. This is called a "pipe" in the
Tidyverse. The Tidyverse is controversial: To some, it's a way of streamlining
sometimes complex code to make it more human-readable; to others, it's a
headache-inducing abomination. I admit that I fall into the former camp, but I
recognize that many folks---especially those who are more fond of Python than
R---hold the opposing opinion. However, even if you don't personally prefer this
style, it's useful for you to know how to read and interact with the Tidyverse,
since it's relatively common within the cognitive science and psychology
community.

The pipe allows you to pass the results of one function to another function
in the same chunk of code. It's like an assembly line, passing the dataframe
that you call in the first line (above, `starwars_df %>%`) to the next line
(above, `summarize(...)`). You can continue passing the results to additional
functions by continuing to pipe.

You might wonder why we're putting each function on a new line. You don't need
to do that: You could simply pipe to another command on the same line. However,
a good habit for programmers---especially for programmers who use git as part of
their code-sharing or version control---is to keep your lines to 80 characters
or less. If you take a look at the `.Rmd` of this tutorial, you'll see that I'm
breaking the lines of text to be about 80 characters, too. This not only helps
your code be a bit more readable (rather than really long lines of code) but
also helps you see more easily the changes that have happened from version to
version in git.

### Comparing Tidyverse and base R options

I personally find that base R's `summary()` is more efficient for very quick
introductions to my data. For the `tidyverse::summarize()` function, we need to
specify each variable and the specific summarization function we want to run.
This provides flexibility (e.g., our addition of the standard deviation), but
you may not always need it.

However, you may also note the massive headache that we got when we subjected a
character variable to `summary()`, which we *didn't* get when we could be more
targeted in our variable selection.

Together, this example shows the utility of combining the Tidyverse with other
functions: As with any toolbox, you want to find the perfect tool for a specific
job---not just treat everything as a `tidy` hammer or a base screwdriver.

## Handling missing values

Next, we'll want to check for missing values. There isn't a clean way of grabbing
only rows with missing data with the Tidyverse, so instead, we'll take a two-step
approach by creating a new dataframe without any missing variables (`drop_na()`)
and then doing an anti-join (`anti_join()`) between the two dataframes.


```r
# drop any missing values
full_starwars_df = starwars_df %>%
  drop_na()

# let's take a look at what we have missing
missing_starwars_df = starwars_df %>%
  anti_join(., full_starwars_df)
```

```
## Joining, by = c("name", "height", "mass", "hair_color", "skin_color", "eye_color", "birth_year", "sex", "gender", "homeworld", "species", "films", "vehicles", "starships")
```

```r
head(missing_starwars_df)
```

```
##                 name height mass  hair_color  skin_color eye_color birth_year
## 1              C-3PO    167   75        <NA>        gold    yellow      112.0
## 2              R2-D2     96   32        <NA> white, blue       red       33.0
## 3        Darth Vader    202  136        none       white    yellow       41.9
## 4        Leia Organa    150   49       brown       light     brown       19.0
## 5          Owen Lars    178  120 brown, grey       light      blue       52.0
## 6 Beru Whitesun lars    165   75       brown       light      blue       47.0
##      sex    gender homeworld species
## 1   none masculine  Tatooine   Droid
## 2   none masculine     Naboo   Droid
## 3   male masculine  Tatooine   Human
## 4 female  feminine  Alderaan   Human
## 5   male masculine  Tatooine   Human
## 6 female  feminine  Tatooine   Human
##                                                                                                                                       films
## 1                    The Empire Strikes Back, Attack of the Clones, The Phantom Menace, Revenge of the Sith, Return of the Jedi, A New Hope
## 2 The Empire Strikes Back, Attack of the Clones, The Phantom Menace, Revenge of the Sith, Return of the Jedi, A New Hope, The Force Awakens
## 3                                                              The Empire Strikes Back, Revenge of the Sith, Return of the Jedi, A New Hope
## 4                                           The Empire Strikes Back, Revenge of the Sith, Return of the Jedi, A New Hope, The Force Awakens
## 5                                                                                     Attack of the Clones, Revenge of the Sith, A New Hope
## 6                                                                                     Attack of the Clones, Revenge of the Sith, A New Hope
##                vehicles       starships
## 1                                      
## 2                                      
## 3                       TIE Advanced x1
## 4 Imperial Speeder Bike                
## 5                                      
## 6
```

(As a quick side note before we get started: You'll see a long list of specific
variables output above, which is how `tidyverse` tells you what variables it
identified as repeating across the two datasets. You can manually specify these
variables if you'd like---which is the safer route---or can allow it to do it
automagically.)

Well, it looks like we've got a whole lot of data considered missing here. If
someone hasn't piloted a vehicle or a starship, for example, they come up as an
`NA`. In our case, we don't want that to show up as missing: We need to have
this coded so that we can differentiate between pilots and non-pilots. We can
do this by using the `mutate()` function, which alters variables.


```r
# let's create new pilot dummy variables
starwars_df = starwars_df %>%
  
  # rowwise functions can be tricky sometimes;
  # just try it with the next call silenced
  rowwise() %>%
  
  # let's make one for vehicles
  mutate(dummy_vehicle_pilot = ifelse(is_empty(vehicles),
                                      0,
                                      1)) %>%
  
  # and then one for starships
  mutate(dummy_starship_pilot = ifelse(is_empty(starships),
                                      0,
                                      1)) %>%
  
  # and then one for any pilot
  mutate(pilot = ifelse((dummy_vehicle_pilot + dummy_starship_pilot)>=1,
                        1,
                        0))

# let's see what we get
head(starwars_df)
```

```
## # A tibble: 6 x 17
## # Rowwise: 
##   name  height  mass hair_color skin_color eye_color birth_year sex   gender
##   <chr>  <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
## 1 Luke…    172    77 blond      fair       blue            19   male  mascu…
## 2 C-3PO    167    75 <NA>       gold       yellow         112   none  mascu…
## 3 R2-D2     96    32 <NA>       white, bl… red             33   none  mascu…
## 4 Dart…    202   136 none       white      yellow          41.9 male  mascu…
## 5 Leia…    150    49 brown      light      brown           19   fema… femin…
## 6 Owen…    178   120 brown, gr… light      blue            52   male  mascu…
## # … with 8 more variables: homeworld <chr>, species <chr>, films <list>,
## #   vehicles <list>, starships <list>, dummy_vehicle_pilot <dbl>,
## #   dummy_starship_pilot <dbl>, pilot <dbl>
```

Great! Our data are shaping up nicely for us. Let's keep moving. Next, we'll need
to remove anyone who wasn't in a *Star Wars* film, given our interests.


```r
# grab anyone with full data
full_starwars_df = starwars_df %>%
  drop_na(films)

# identify anyone with missing data
missing_starwars_df = starwars_df %>%
  anti_join(., full_starwars_df)
```

```
## Joining, by = c("name", "height", "mass", "hair_color", "skin_color", "eye_color", "birth_year", "sex", "gender", "homeworld", "species", "films", "vehicles", "starships", "dummy_vehicle_pilot", "dummy_starship_pilot", "pilot")
```

```r
# now, let's see how many we have in each
print(paste0("Full records: ",dim(full_starwars_df)[1]))
```

```
## [1] "Full records: 87"
```

```r
print(paste0("Missing records: ",dim(missing_starwars_df)[1]))
```

```
## [1] "Missing records: 0"
```

## Derive movie counts

Here, we'll need to figure out how to identify the number of movies from the
embedded list in the `films` variable. Find a way of extracting this as an
integer to be included as a new variable, `film_count`.


```r
full_starwars_df = full_starwars_df %>%
  mutate(film_count = ...)
```

***

# Data manipulation

Now that we're pretty confident about the variables we want, we can create a
clean dataframe with those to minimize the amount of hassle we have when trying
to quickly inspect them.


```r
# use select() to identify only the variables we want to include
clean_starwars_df = full_starwars_df %>%
  select(name, height, mass, birth_year, gender, films, pilot)
```

With this cleaner dataframe, let's go ahead and transform our data to the
correct variable type.


```r
# let's properly hold integers and factors
clean_starwars_df = clean_starwars_df%>%
  mutate_at(vars(gender, pilot), as.factor) %>%
  mutate_at(vars(height, mass), as.integer)
```

The `mutate_at` syntax is just one example of a way that we can complete
more complex functions over dataframes using the Tidyverse. In this case,
we are asking to convert each variable that is a numeric-type variable.

***

# Data visualization

Always, always, always plot your data. Here, come up with 1-2 visualizations of
the data, focusing on the most relevant variables to our analyses


```r
plot(...)
```

***

# Exercise

Now, try your hand at using the Star Wars API to download the planets or
vehicles dataframe and join them together with the characters dataset that we've
been using. Try joining them together and structuring a dataset that could
answer a new question.



