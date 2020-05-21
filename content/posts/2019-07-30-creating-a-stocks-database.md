---
title: Creating a stocks database
author: ''
template: "post"
date: '2019-07-30'
slug: creating-a-stocks-database
category: "Data Collection"
tags:
  - stocks
  - sql
draft: true
description: "An Essay on Typography by Eric Gill takes the reader back to the year 1930. The year when a conflict between two worlds came to its term. The machines of the industrial world finally took over the handicrafts."
---


I just started working on teaching myself sql and after completing the [codeacadmey sql lessons](https://www.codecademy.com/learn/learn-sql), there are links to independant projects. My goal is to work on these independant projects in the coming weeks.

To look ahead (and have a place to find them later outside my pocket list) here's a list to all five I plan on working on:

1. [Watching the Stock Market](https://discuss.codecademy.com/t/data-science-independent-project-1-watching-the-stock-market/419943)
2. [Explore a Sample Database](https://discuss.codecademy.com/t/data-science-independent-project-2-explore-a-sample-database/419945)
3. [Education & Census Data](https://discuss.codecademy.com/t/data-science-independent-project-3-education-census-data/419947)
4. [Home Value Trends](https://discuss.codecademy.com/t/data-science-independent-project-4-home-value-trends/419948)
5. [Analyze Airfare Data](https://discuss.codecademy.com/t/data-science-independent-project-5-analyze-airfare-data/419949)

***

The first project has us working with stock market data. The project asks us to gather data for five different stocks

Before performing SQL work, I need to gather some stock data. [Yahoo Finance](https://finance.yahoo.com/) is a great resource for historical data and allows an export straight to csv.

I've downloaded data over the past three months (from April 30, 2019 through today, July 30. 2019). I suggest finding some stocks that interest you! I've decided to look at the stocks for

* Apple ([APPL](https://finance.yahoo.com/quote/AAPL))
* Boeing ([BA](https://finance.yahoo.com/quote/BA))
* Electronic Arts ([EA](https://finance.yahoo.com/quote/EA))
* Nvidia ([NVDA](https://finance.yahoo.com/quote/NVDA)
* Spotify ([SPOT](https://finance.yahoo.com/quote/SPOT))

I also saved the data I'm using [here](https://github.com/chris-s-friedman/friedman_blog/tree/master/data/stocks). Note that the price I'm using is the adjusted closing price.

For speed, I'm creating the database in the DB Browser GUI. Instructions for that are on the project instruction page. The database is saved along with individual CSVs for each stock.

Stay tuned for my next bit, using some simple sql to work with this data!
