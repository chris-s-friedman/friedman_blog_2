---
title: Accessing the RateBeer API with R Using httr
author: ''
template: "post"
date: '2017-12-05'
slug: collecting-beer-data-using-ratebeer
category: "Data Collection"
tags:
  - beer
  - data_collection
  - scraping
  - httr
  - r
  - jsonlite
  - ratebeer
  - API
draft: false
description: "A few months ago, I was talking with a friend of mine about the idea for this blog and how I wanted to use data science to explore beer. He suggested that I use the blog as well as beer to learn something new about where I live. So I ask, what can beer teach me about Philadelphia?"
---
```{r init, warning=FALSE, message=FALSE, echo=FALSE}
library(knitr)
library(kableExtra)
```

A few months ago, I was talking with a friend of mine about the idea for this blog and how I wanted to use data science to explore beer. He suggested that I use the blog as well as beer to learn something new about where I live. So I ask, what can beer teach me about Philadelphia?

## The first thing I need? Data!

Oddly enough, it's actually pretty challenging to get access to high quality, current beer data.

I chose to use [RateBeer's](https://www.ratebeer.com/) data, mostly because they have an easily accessible [API](https://www.ratebeer.com/api.asp), and meet my needs better than anyone else. They also [disclose](https://www.ratebeer.com/ratingsqa.asp) how they come to their average beer rating, allowing me to see what's under the hood. In the footnotes, I briefly explain some alternatives[^1]

## Collecting Data

I want to look at breweries in the area. Sadly, the RateBeer API doesn't have a feature to search for breweries in the area. There is however, a way to query what beers a brewery makes.
To get a list of beers a brewery makes though, I need to know what that brewery's unique ID is. Easy enough to find. the URL for a brewery on RateBeer is of the form:

`https://www.ratebeer.com/brewers/<BREWERY_NAME_HERE>/<BREWER_ID_HERE>`

So, as an example, `166` is the ID for [Yards Brewering Company](https://www.ratebeer.com/brewers/yards-brewing-company/166/). The url is:

`https://www.ratebeer.com/brewers/yards-brewing-company/166/`

RateBeer's API uses the language of [GraphQL](http://graphql.org/learn/). It's beyond the scope of this post to dive into GraphQL, so instead, I'll explain how it's implemented in regards to the queries that I make.

GraphQL requests are written in JSON format. Basically, I specify that I make a query. Nested in that call is the type of query I want to make (as well as arguments to that query), which then has the responses I wanted nested within the call.

So, a query for the name of the beer with the ID number `4934` looks like this:

```
query {
  beer(id: 4934) {
    name
  }
}
```

Similarly, my query for the beers made by Yards will look like this:

```
query{
  beersByBrewer(brewerId: 166) { # 166 is Yards
    totalCount # This gives the total number of beers in the beer list
    items{ # For each beer, I want these items...
      name # the name of the beer
      abv # the beer's ABV
      averageRating # the average rating of the beer
      ratingCount # the number of ratings the beer has
      isRetired # is the beer retired?
      style{
        # Style needs to be jumped into one level because when I query style, I
        # can also ask for a description of the style, and can even jump into
        # recommended glassware. That said, all I want is the style name.
        name
      }
    }
  }
}
```

Now that I know what format to make my request in, it's time to actually get my first bit of data off the API!

### Hello, httr

[`httr`](https://github.com/r-lib/httr) is a package developed by Hadley Wickham of RStudio to make it easy to make HTTP requests. the package and the nuances of HTTP wont be gone into here, but some good resources for httr include the [quickstart vignette](https://cran.r-project.org/web/packages/httr/vignettes/quickstart.html), and Bradley Boehmke's [post](http://bradleyboehmke.github.io/2016/01/scraping-via-apis.html#httr_api) on using httr.
The httr syntax is quite simple. The main functions are curl verbs (httr is a wrapper for [curl](https://github.com/jeroen/curl) functions), and the function arguments all start with the URL and are then followed by things to modify the URL and to send with the URL.

RateBeer's API is at `https://api.ratebeer.com/v1/api/graphql`. In the header of the request, content type and response type are required along with an API key. The query itself modifies the URL to call. So, my call to the RateBeer API to get the beers made by yards looks like this:

```{r get_data}
library(httr)

API_key <- Sys.getenv("rateBeer_API_key")

URL <- "https://api.ratebeer.com/v1/api/graphql"

beers_by_yards <- POST(URL,
         body = list(
           query =
"query{
  beersByBrewer(brewerId: 166) {
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
    }
  }
}",
           variables = "{}",
           operationName = NULL),
         encode = "json", # tells httr to encode the body of the request as json
         add_headers("content-type" = "application/json",
                     "Accept" = "application/json",
                     "x-api-key" = API_key))

```

### Parsing the Response

`content()` is httr's function for extracting content from a request. Using the `type = ` argument, we can have the function give us the data from the request as valid JSON. then we can use the `jsonlite` package to make the data easier to work with.

```{r parse_json, message=FALSE, warning=FALSE}
library(jsonlite)

json <- content(beers_by_yards, type = "text")

parsed_json <- fromJSON(json, flatten = TRUE)
```

Recall that the response to the request was supposed to be JSON? This means that all of our items are nested in the same way as we requested them. So, to get the number of beers that Yards makes, as well as a data frame with those beers, we work through those levels.

```{r look_at_data}
beer_count <- parsed_json$data$beersByBrewer$totalCount

beer_df <- parsed_json$data$beersByBrewer$items

beer_count
kable(beer_df, "html") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```

So, where's all 111 beers? The API gives us 10 beers at a time. When We make the request to the API, we can tell it where to start that list of 10 beers, alongside our request to look at beers from a certain brewery.

In my next post, I'll show how we can get all 111 of those beers and beers from other breweries, programmatically.

[^1]:
* **[Beer Advocate](https://www.beeradvocate.com/community/threads/terms-of-service.101118/)** expressly forbids scraping and does not have an official API.
* **[Untappd](https://untappd.com/terms/api)** has an API but they don't give out API keys to people that are just interested in data. If I build an app, maybe my decision will change, but in the meantime, no using their API. It looks like they may not expressly [forbid](https://untappd.com/terms/)scraping or crawling on the site, but scraping has its own challenges. I may cover it in the future, but in the meantime, I want to just use an API.
* **[BeerDB](http://www.brewerydb.com/developers/docs)** looks like an awesome idea - beer data for developers! Yet, the API doesn't show ratings and you can only get ABV if you are a premium user.I can get all the information that I am looking for from other APIs, so no need to use *and* pay for this one.
* **[Open Beer Database](https://openbeerdb.com/)** hasn't updated the database since 2011. That's a solid no.
* **[The Beer Spot](http://www.thebeerspot.com/)** looks like it could be a fun community, but considering that A), [they may not have very many users](http://www.thebeerspot.com/forum/index.php/topic,17296.msg728792.html#msg728792) and B) no one has [reviewd Yuengling](http://www.thebeerspot.com/beer/info/yuengling-brewery/yuengling-traditional-lager) (beer geek or not, a Philadelphia area staple) I'm not going to use their API.
