---
title: Getting RateBeer Data…Programmatically
author: ''
template: "post"
date: '2017-12-07'
slug: getting-ratebeer-data-programmatically
category: "Data Collection"
tags:
  - API
  - beer
  - data_collection
  - httr
  - jsonlite
  - r
  - ratebeer
  - scraping
  - rvest
description: "I show how we can get more beers at once. After that, I'm going to show how we can use the API, `rvest`, and `purrr` to get beers from all the brewers around me."
---

```{r, include=FALSE}
library(httr)
library(jsonlite)
library(purrr)
library(rvest)
knitr::opts_chunk$set(eval = FALSE)
API_key <- Sys.getenv("rateBeer_API_key")
```



In my [last post](../collecting-beer-data-using-ratebeer) I showed how using the package `httr`, You can access the RateBeer API to get information about beers made by a brewery. When I left off, I showed a problem - the API only shows 10 beers at a time.

Today, I'm going to show how we can get more beers at once. After that, I'm going to show how we can use the API, `rvest`, and `purrr` to get beers from all the brewers around me.

## Updating the call to the API

### Setting the `first:` argument

In the last post, I didn't mention one of the arguments that can be used when making `beersByBrewer` query. Besides the argument for the brewerID, we can also use the `first` argument to specify how many beers we want to see from a brewer. As you will see below, one of the changes I make to the call to the API includes seting the value for the argument `first` to `999`.

Ninety-nine beers on the wall? Let's make it Nine hundred and ninety-nine.

### Turning the call to the API into a function.

As you will see later on, it will be handy to have the call to the API as a function. Below, I declare that function:

```{r}
get_beers_from_brewer <- function(brewer_id, api_key) {
  URL <- "https://api.ratebeer.com/v1/api/graphql"
  POST(URL,
       body = list(
         query = paste0(
"query{
  beersByBrewer(brewerId: ", brewer_id, ", first: 999) {
    totalCount
      items{
        name
        abv
        averageRating
        ratingCount
        isRetired
        style{
          name
        }
        brewer {
          id
          name
          streetAddress
          city
          state {
            name
          }
          zip
        }
      }
     }
}"),
       variables = "{}",
       operationName = NULL),
       encode = "json", # tells httr to encode the body of the request as json
       add_headers("content-type" = "application/json",
                   "Accept" = "application/json",
                   "x-api-key" = api_key))
}
```

## Finding Breweries

Okay, so now, I have an easy way to get information about the beers that breweries make. All I need to do now is point the API to the brewer ID of each brewery I want information on.

Remember, the brewer id can be found by looking at the url for that brewery and is in the form:

`https://www.ratebeer.com/brewers/<BREWERY_NAME_HERE>/<BREWER_ID_HERE>`

The thing is, I like data and want A LOT of it. There's no way I'm going to hand sort through urls to try to find breweries. Can't this be automated?

YES!

RateBeer maintains lists of breweries by state. For example, the breweries in Pennsylvania can be found [here](https://www.ratebeer.com/breweries/pennsylvania/38/213/). Using the package [`rvest`](https://github.com/hadley/rvest), we can pull information about all of the breweries in the state, and then use [`purrr`](https://github.com/tidyverse/purrr) to iterate over that list, using the function, `get_beers_from_brewer()`.   

`rvest` is a package to make harvesting information from the web easy. Below, you see how, in three steps, I have a list of urls that point to all of the breweries in the state.

```{r}
library(rvest)

brewery_list_url <- "https://www.ratebeer.com/breweries/pennsylvania/38/213/"

brewery_ids <-  read_html(brewery_list_url) %>%
  html_nodes("#brewerTable a:nth-child(1)") %>%
  html_attr('href')
```

To explain what happened in the previous code chunk:

1. `read_html()` loads the url for the list of breweris in PA. This is the same thing that happens if you click this [link](https://www.ratebeer.com/breweries/pennsylvania/38/213/).
2. `html_nodes()` searches for every place in the html file we navigated to that has a link to a brewery. The text in the argument for the function points to the css selector for where breweries can be found on the page. I found this selector using [selectorgadget](http://selectorgadget.com/). This function gives me a list of the each time that selector shows up.
3. `html_attr()` searches that list for an attribute of the specified type. In this case I specified a hyperlink.

```{r, echo=FALSE}
yards_url <- grep("yards", brewery_ids, value = TRUE)
```

As I said, this outputs a list of urls. As an example, the url for Yards Brewing looks like this:

> `/brewers/yards-brewing-company/166/`

Now, I can take this list of urls, and use the function `purrr::map_chr()` to get a list of brewer IDs. As a note, I use `map_chr` because it flattens the list of IDs into a single character vector. I highly suggest that you check out the rest of the `map_*` functions.

Below, I take each item in the list of urls, split each url at any `"/"` and then use `map_chr()` to select the fourth element, the brewer ID.

```{r}
brewery_ids <- brewery_ids %>%
  strsplit("/") %>%
  map_chr(4)
```

Now I have a list of brewery IDs that I can feed into `get_beers_from_brewer()`

## Getting ALL THE BEERS!

To iterate through the list, we'll use `map()` and some functions from `jsonlite`, a package that can parse JSON. Then we'll use `map()` to work through the levels of the response, from it's highest level ("data") down to the actual data frame (held in the named object, "items").


```{r, message=FALSE, warning=FALSE}
brewery_beer_df <- brewery_ids %>%
  map(function(brewer_id){
    Sys.sleep(1) # the API restricts to 1 request per second.
    get_beers_from_brewer(brewer_id, API_key)
  }) %>%
  # Here we use jsonlite functions to turn the response of the request into
  # json
  map(content, type = "text") %>%
  # and then turn that into an r object which has dfs in it from each
  # brewery.
  map(fromJSON, flatten = TRUE) %>%
  # working down through the response levels.
  map("data") %>% map("beersByBrewer") %>% map_dfr("items")
```

The object `brewery_beer_df` is the data frame with all the beers from breweries we requested.
