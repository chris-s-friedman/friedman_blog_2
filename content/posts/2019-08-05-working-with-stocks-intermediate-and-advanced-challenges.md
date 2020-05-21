---
title: 'Working with stocks: Intermediate and Advanced Challenges'
author: ''
template: "post"
date: '2019-08-10'
draft: true
slug: working-with-stocks-intermediate-and-advanced-challenges
category: "sql"
tags:
  - sql
  - stocks
keywords:
  - tech
description: "An Essay on Typography by Eric Gill takes the reader back to the year 1930. The year when a conflict between two worlds came to its term. The machines of the industrial world finally took over the handicrafts."
---

This is a continuation of my work on the [codeacademy data science independent project #1: Watching the Stock Market](https://discuss.codecademy.com/t/data-science-independent-project-1-watching-the-stock-market/419943).

* My [first post](../../07/creating-a-stocks-database/) on the subject explores how I put gathered the stock data.
* In my [second post](../../07/working-with-stocks-basic-sql), I explored the data using basic sql queries (and put my sql code out into the world for the first time!)

***
After completing the basic challenges from my last post on the topic, there are intermediate and advanced challenges too while playing with these stocks data.

Let's jump into the challenges!
```{r setup, include=FALSE}
library(DBI)
db = dbConnect(RSQLite::SQLite(), dbname = "../../data/stocks/stocks.db")
```

# Intermediate Challenges

## 1. Use aggregate functions to look at key statistics about the data (e.g., min, max, average).

```{sql intermediate1, connection=db}
SELECT ROUND(AVG(price), 2) AS average_price,
       ROUND(MIN(price), 2) AS min_price,
       ROUND(MAX(price), 2) AS max_price
FROM stocks;
```

At the end of my last post, I mentioned that it may be possible that I picked... well expensive stocks. at an average cost of nearly $190 per share, I'd definitely say these aren't cheap!

## 2. Group the data by stock and repeat. How do the stocks compare to each other?

So, what's the variability in stock price within each stock?

```{sql intermediate2, connection=db}
SELECT name,
       ROUND(AVG(price), 2) AS average_price,
       ROUND(MIN(price), 2) AS min_price,
       ROUND(MAX(price), 2) AS max_price
FROM stocks
GROUP BY name
ORDER BY average_price;
```

So, EA's stock is lower than all the others and never even overlaps the stock price of any of the other stocks. On the other end of the spectrum is Boeing - more expensive than all the others by leaps over $100 dollars a share! Spotify and Nvidia are closer in price to each other than Nvidia and Apple but Spotify isn't as expensive as Nvidia.


## 3. Group the data by day or hour of day. Does day of week or time of day impact prices?

```{sql intermediate3, connection=db}
SELECT date,
       ROUND(AVG(price), 2) AS average_price
FROM stocks
GROUP BY date
ORDER BY average_price DESC;
```

Well, I don't know about day of the week but it does look like that things were looking particularly good at the beginning of may!


## 4. Which of the rows have a price greater than the average of all prices in the dataset?
```{sql test, connection=db}
SELECT *
FROM stocks
WHERE price > 189.5;
```
A lot of rows. A lot of rows have a price greater than the average price (110 to be precise). If you look at the table from challenge 2, you'll see that the only stocks above the average are from Apple and Boeing.

# Advanced Challenge

## 1. Calculate other key statistics, like median or variance.

```{sql variance, connection=db}
SELECT AVG((stocks.price - sub.a) * (stocks.price - sub.a)) AS variance
FROM stocks,
    (SELECT AVG(price) AS a
     FROM stocks) AS sub;
```

```{sql median, connection=db}
SELECT price AS median
FROM stocks
ORDER BY price
LIMIT 1
OFFSET (SELECT COUNT(price)
        FROM stocks) / 2;
```


## 2. Refactor the data into 2 tables

We'll refactor the stocks table into stock_info to store general info about the stock itself (ie. symbol, name) and stock_prices to store the collected data on price (ie. symbol, date, price).

```{sql create_stock_info, connection=db}
CREATE TABLE stock_info
AS SELECT DISTINCT symbol, name
   FROM stocks;
```
```{sql create_stock_price, connection=db}
CREATE TABLE stock_prices
AS SELECT symbol, date, price
   FROM stocks;
```

```{sql show_stock_info, connection = db}
SELECT *
FROM stock_info;
```
```{sql show_stock_prices, connection = db}
SELECT *
FROM stock_prices;
```

## 3. Now join the 2 tables in order to view more information on the stock with each row of price.
```{sql advanced3, connection = db}
SELECT stock_prices.symbol, date, price, name
FROM stock_prices
LEFT JOIN stock_info ON stock_info.symbol = stock_prices.symbol;
```

Awesome! Now on to the next set of SQL challenges in another post!

```{sql clean_up, connection = db}
DROP TABLE stock_prices;
```
```{sql clean_up2, connection = db}
DROP TABLE stock_info;
```
