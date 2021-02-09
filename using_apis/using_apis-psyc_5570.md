---
title: 'Introduction to APIs'
author: "A. Paxton (*University of Connecticut*)"
output:
  html_document:
    keep_md: yes
    number_sections: yes
---

An API---or *application programming interface*---is a formal avenue created 
by an organization or website to programmatically access their data. We could
most directly contrast an API with *web scraping*, which we will cover in a 
future session.

To use an API, you must submit a *request* to the system. That request
includes a set of information that has been specified by the API developers.
Minimally, the request will describe the kind of data that you want to access
(in the manner outlined by the API). For some systems, this must also include an
*API key*, which is an authenticated permission identifier that allows you to
access data. 

After submitting your request, you will receive a *response* that contains the
data you've requested. Your next job will be to parse the response into usable
data.

In this exercise, we'll walk through the process of using an API directly from R.

***

# Preliminaries

First, we'll need to prepare for our exercises. We'll do this by loading the
packages we need, including---if needed---installing those packages. For the
sake of the exercise, we'll load the packages silently, meaning that we won't
clutter our R markdown output with a bunch of warnings and output messages
from the loading and installation process.


```r
# clear the workspace (useful if we're not knitting)
rm(list=ls())
```




```r
# specify which packages we'll need
required_packages = c("httr",
                      "jsonlite")

# install them (if necessary) and load them
load_or_install_packages(required_packages)
```

***

# Finding APIs

Not every website, repository, or organization has an API available for users to
access data, but those that do have them tend to make them easily available. You
might start by checking the website for an API information link. This might be
at the bottom of the webpage or available in a "Developers" menu of the website.
If you can't find one, try using the "find" function in your browser for `API`,
or try searching `API` in the website's search function (if they have one).

If you still can't find it, try doing a general search on your favorite search
engine with the name of the website and API. (As you may already know, putting
both terms in separate double-quotes will ensure you only get webpage results
that include both terms.)

If you still can't find anything, it may be time to turn to web scraping---but,
again, we won't be covering that here. (Again, keep in mind matters of
ethicality and legality, including the *hiQ Labs v. LinkedIn* ruling in the US.)

***

# Submitting a request

For this exercise, we'll be accessing the [NYC Open Data
repository](https://opendata.cityofnewyork.us/). This rich dataset includes open
data from a variety of government sources from New York, NY (USA). If you take
time to poke around the website, you'll see a large number of datasets
available. Let's choose the [2015 Street Tree Census
data](https://data.cityofnewyork.us/Environment/2015-Street-Tree-Census-Tree-Data/pi5s-9p35).

For this, we'll be using the `httr` library's `GET` function.

## Identify API URL

To get started, we need to figure out more about this API. A good place to start
is to take a look at the website. Once you get to the platform, you should see
(as of February 2021) an interactive map of the tree census with a few boxes of
words above the upper-right corner of the map. If you click "Export," a new
sidebar will appear, and you'll get access to a number of options (yes,
including the option to just download a CSV of the data). One option will be the
"SODA API" (or *Socrata Open Data API (SODA)*), which we'll use for this exercise. 

Once you click on the SODA API link, you'll see new information pop up,
including lots of helpful links to additional documentation about the SODA API
and a list of fields in this dataset. After looking at the API information, we 
know the URL of our dataset. Let's create a variable to hold onto that.


```r
# specify the API location
tree_data_location = "https://data.cityofnewyork.us/resource/5rq2-4hqu.json"
```

## Identify filtering parameters for request

Next, we decide what subset of the data we'd like to receive.
Unfortunately, the metadata isn't terribly detailed, so let's click on a few
trees on the map to find out what some examples might be. 

This might feel clunky or inelegant, but keep in mind that a lot of working with
naturally occurring data requires persistence and creativity---along with all of
the usual admonitions to know your data!

After poking around, let's say that we want to find all trees in the Bronx. In
looking at the metadata list and some of the options in the visualized dataset,
we've found the variable name (`zip_city`) and value (`Bronx`) for it. We need
to prepare the variable in a `list` item, which `httr::GET()` expects.

When we input the values, we need to respect the data type in R and in the API.
Given that `Bronx` is a string, we'll need to remember to encapsulate it in
double-quotes so that R doesn't get angry with us.


```r
# specify what we want to filter by
return_variables = list(zip_city = "Bronx")
```

## Create and submit the request

Finally, let's put together the pieces we've already created and submit the
request to the API.


```r
# create and submit the request
request = GET(url = tree_data_location,
              query = return_variables)

# show us what we received
request
```

```
## Response [https://data.cityofnewyork.us/resource/5rq2-4hqu.json?zip_city=Bronx]
##   Date: 2021-02-02 16:59
##   Status: 200
##   Content-Type: application/json;charset=utf-8
##   Size: 900 kB
## [{"created_at":"08/29/2015","tree_id":"184375","block_id":"505172","the_geom"...
## ,{"created_at":"08/29/2015","tree_id":"184317","block_id":"505563","the_geom"...
## ,{"created_at":"08/29/2015","tree_id":"184031","block_id":"504204","the_geom"...
## ,{"created_at":"08/29/2015","tree_id":"186888","block_id":"506316","the_geom"...
## ,{"created_at":"09/08/2015","tree_id":"209441","block_id":"503960","the_geom"...
## ,{"created_at":"09/08/2015","tree_id":"209909","block_id":"504038","the_geom"...
## ,{"created_at":"08/29/2015","tree_id":"186060","block_id":"505600","the_geom"...
## ,{"created_at":"09/04/2015","tree_id":"201682","block_id":"515571","the_geom"...
## ,{"created_at":"08/29/2015","tree_id":"183942","block_id":"504242","the_geom"...
## ,{"created_at":"09/01/2015","tree_id":"196020","block_id":"506702","the_geom"...
## ...
```
Great! It looks like our submission worked. It's useful to check for errors by
ensuring that the `Status` value of the response is `200`. You can do this by
manually inspecting the output (as seen above) or by calling it from the 
returned value, as shown below:


```r
# programmatically grab the response value
request$status_code
```

```
## [1] 200
```

## A quick note about API keys or tokens

As mentioned in the beginning of this exercise, some APIs may gate access to
their data by issuing API keys or tokens that must be included in a request. In
some cases, the keys may be required to access the API at all; in other cases,
they may only be required if a user is requesting data above a specific rate or
above a specific volume. The idea behind the token is that it identifies the
requester to the organization. Any API that requires a key should have detailed
information about how to request one and any restrictions for use; be sure to
consult the documentation for specific information from each organization, since
these policies vary widely.

***

# Debugging

Before we move onto parsing the data, let's take a moment to consider some possible
issues with our requests.

## Problems with variable names

Let's say that we had a typo in our request, using `zipcity` instead of
`zip_city` for a variable name.


```r
# let's say that we forgot the underscore in the variable
return_variables_with_name_typo = list(zipcity = "Bronx")

# we'll create and submit our faulty request
request_with_name_typo = GET(url = tree_data_location,
                             query = return_variables_with_name_typo)

# show us what we received
request_with_name_typo
```

```
## Response [https://data.cityofnewyork.us/resource/5rq2-4hqu.json?zipcity=Bronx]
##   Date: 2021-02-02 16:59
##   Status: 400
##   Content-Type: application/json; charset=utf-8
##   Size: 71 B
## {
##   "error" : true,
##   "message" : "Unrecognized arguments [zipcity]"
## }
```

As you can see, it is possible for us to correctly submit something from the R
side that the API won't accept. In this case, `httr::GET()` received everything
in the correct format and therefore submitted our request to the API. However,
the API sent back an error to let us know that there was an unrecognized
variable name. If you receive this kind of error, be sure to go back to the list
of variables and check for typos.

## Problems with variable values

It's important to contrast the typo in a variable name with a typo in the
variable *value*. In this example, let's say that we correctly specify the
variable name (`zip_city`) but that we have a typo in our desired value (`Brox`
instead of `Bronx`).


```r
# let's say that we forgot the underscore in the variable
return_variables_with_value_typo = list(zip_city = "Brox")

# we'll create and submit our faulty request
request_with_value_typo = GET(url = tree_data_location,
                             query = return_variables_with_value_typo)

# show us what we received
request_with_value_typo
```

```
## Response [https://data.cityofnewyork.us/resource/5rq2-4hqu.json?zip_city=Brox]
##   Date: 2021-02-02 16:59
##   Status: 200
##   Content-Type: application/json;charset=utf-8
##   Size: 3 B
## []
```

Again, `httr::GET()` had no problems submitting this request. However, we don't
get the nice error message from the API this time. Instead, because we submitted
something with existing variable names, it simply winnowed down the list and
returned every record with a value of `Brox` in the `zip_city` variable---that
is to say, zero records. This is a more obvious problem if the typo resulted in
a nonexistent value in the variable (because it will return zero records), but
it could pose a more serious problem if the typo results in a valid value
*other* than the one you intended. This highlights the importance of checking
our output for expected values *and* checking your code for accuracy.

*** 

# Parsing output

Now that we have our intended response (saved in the `request` variable), we'll
need to convert it into something usable. (To avoid cluttering the output, we'll
skip displaying the intermediary steps here, but if you'd like to see the mess
that is the text-only response, feel free to inspect the `response` variable.)


```r
# convert the response's values to text and then to a dataframe
response = content(request, as = "text")
response_df = data.frame(fromJSON(response, flatten=TRUE))

# let's take a peek at what we got
head(response_df)
```

```
##   created_at tree_id block_id tree_dbh stump_diam curb_loc status health
## 1 08/29/2015  184375   505172        3          0   OnCurb  Alive   Good
## 2 08/29/2015  184317   505563        5          0   OnCurb  Alive   Good
## 3 08/29/2015  184031   504204        3          0   OnCurb  Alive   Good
## 4 08/29/2015  186888   506316       15          0   OnCurb  Alive   Fair
## 5 09/08/2015  209441   503960        5          0   OnCurb  Alive   Poor
## 6 09/08/2015  209909   504038        2          0   OnCurb  Alive   Good
##                            spc_latin       spc_common steward guards sidewalk
## 1 Gleditsia triacanthos var. inermis      honeylocust    None   None NoDamage
## 2                  Quercus palustris          pin oak    None   None NoDamage
## 3                        Acer rubrum        red maple    1or2   None NoDamage
## 4              Platanus x acerifolia London planetree    None   None NoDamage
## 5                    Ulmus americana     American elm    None   None NoDamage
## 6                    Ulmus americana     American elm    None   None NoDamage
##          user_type problems root_stone root_grate root_other trnk_wire
## 1        Volunteer   Stones        Yes         No         No        No
## 2        Volunteer   Stones        Yes         No         No        No
## 3 TreesCount Staff     None         No         No         No        No
## 4        Volunteer     None         No         No         No        No
## 5 TreesCount Staff   Stones        Yes         No         No        No
## 6        Volunteer     None         No         No         No        No
##   trnk_light trnk_other brnch_ligh brnch_shoe brnch_othe
## 1         No         No         No         No         No
## 2         No         No         No         No         No
## 3         No         No         No         No         No
## 4         No         No         No         No         No
## 5         No         No         No         No         No
## 6         No         No         No         No         No
##                          address zipcode zip_city cb_num borocode boroname
## 1        2022 LA FONTAINE AVENUE   10457    Bronx    206        2    Bronx
## 2                  4634 3 AVENUE   10458    Bronx    206        2    Bronx
## 3            223 EAST 179 STREET   10457    Bronx    205        2    Bronx
## 4 249 EAST MOSHOLU PARKWAY NORTH   10467    Bronx    207        2    Bronx
## 5             75 FEATHERBED LANE   10453    Bronx    205        2    Bronx
## 6           1840 GRAND CONCOURSE   10457    Bronx    205        2    Bronx
##   cncldist st_assem st_senate  nta                          nta_name boro_ct
## 1       17       86        33 BX17                      East Tremont 2037504
## 2       15       78        33 BX06                           Belmont 2038700
## 3       15       86        33 BX41                        Mount Hope 2023502
## 4       11       80        36 BX43                           Norwood 2041900
## 5       14       77        29 BX36 University Heights-Morris Heights 2021502
## 6       15       86        33 BX41                        Mount Hope 2023302
##      state    latitude    longitude          x_sp          y_sp the_geom.type
## 1 New York 40.84794708 -73.89338226 1013747.44263 248225.382208         Point
## 2 New York 40.85677447 -73.89097861 1014408.44656  251442.35412         Point
## 3 New York 40.85091786 -73.90356058  1010930.2629 249304.486021         Point
## 4 New York 40.87343706 -73.88088627 1017192.02072   257516.8131         Point
## 5 New York 40.84637583 -73.91763219 1007038.86419 247645.677987         Point
## 6 New York 40.84810623 -73.90688618 1010011.31362 248279.107166         Point
##   the_geom.coordinates
## 1  -73.89338, 40.84795
## 2  -73.89098, 40.85677
## 3  -73.90356, 40.85092
## 4  -73.88089, 40.87344
## 5  -73.91763, 40.84638
## 6  -73.90689, 40.84811
```
Fantastic! Our dataframe looks well-formed and ready to go.

***

# Next steps

Of course, just because we have a dataframe, doesn't mean we can jump right into 
analyses. You'll want to be sure to go through all of the steps of cleaning and
preparing your dataset (e.g., converting variables from `chr`, looking for missing
values, transforming your data), which we'll go through in later sessions.

***
