---
title: 'Working with stocks: Basic SQL'
author: ''
template: "post"
date: '2019-07-31'
slug: working-with-stocks-basic-sql
category: "SQL"
tags:
  - sql
  - stocks
keywords:
  - tech
draft: true
description: "An Essay on Typography by Eric Gill takes the reader back to the year 1930. The year when a conflict between two worlds came to its term. The machines of the industrial world finally took over the handicrafts."
---

This is a continuation of my work on the [codeacademy data science independant project #1: Watching the Stock Market](https://discuss.codecademy.com/t/data-science-independent-project-1-watching-the-stock-market/419943). My first post is [here](../creating-a-stocks-database/).

***
In this project, the basic tasks are:

1. What are the distinct stocks in the table?
2. Query all data for a single stock. Do you notice any overall trends?
3. Which rows have a price above 100? between 40 to 50, etc?
4. Sort the table by price. What are the minimum and maximum prices?

# Setup

I write this blog using rmarkdown and the entire project is powered by bookdown. SO there's some setup that needs to take place to get it all to work.

```{r setup}
library(DBI)
db = dbConnect(RSQLite::SQLite(), dbname = "../../data/stocks/stocks.db")
```


# What are the distinct stocks in the table?
```{sql connection=db}
SELECT DISTINCT name
FROM stocks;
```

In my last post about putting together the database, I said that I was collecting data on stocks for Apple, Boeing, Spotify, EA, and Nvidia. Here's the proof!

# Query all data for a single stock. Do you notice any overall trends?

Alright! To analyze trends for the Apple stocks, we'll plot them thorugh time. And, we'll do it in R! First, I'll run some SQL code to extract the APPL stocks. Silently, the output of the SQL query below will be returned as variable we can access in an R code chunk, `appl_stocks`.
```{sql connection=db, output.var="appl_stocks"}
SELECT *
FROM stocks
WHERE name = 'Apple'
```

```{r appl_overall_trends}
library(ggplot2)

appl_stocks$date <- as.Date(appl_stocks$date, "%m/%d/%y")

ggplot(appl_stocks, aes(x = date, y = price)) +
  geom_line()

```

Huh! Something happened in the beginning of June that was bad for Apple's stock. Looks like it is taking about two months for the price to recover. Wonder what that is... My first guess was the anouncement of Johnny Ive, but I looked and that wasn't until the [end of June](https://www.wired.com/story/jony-ive-leaves-apple/). My best guess is decline in iPhone sales coupled with s[ome negative predictions of tech stocks](https://www.usnews.com/news/business/articles/2019-07-30/apples-quarterly-profit-falls-as-iphone-sales-sputter).

# Which rows have a price above 100? between 40 to 50, etc?

This just requires some simple filtering!

```{sql connection=db}
SELECT COUNT(*)
FROM stocks
WHERE price > 100;
```

Wow! That's a few stocks!

```{sql connection=db}
SELECT COUNT(*)
FROM stocks
WHERE price >= 40
  AND price <= 50;
```

Well, I picked pricy stocks...

# Sort the table by price. What are the minimum and maximum prices?

These are the least expensive stocks
```{sql connection=db}
SELECT *
FROM stocks
ORDER BY price;
```

And these are the most expensive stocks.
```{sql connection=db}
SELECT *
FROM stocks
ORDER BY price DESC;
```
