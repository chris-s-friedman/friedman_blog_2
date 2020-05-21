---
title: Rocking Out on SQL - More Exploring
author: ''
draft: true
date: '2019-08-10'
slug: rocking-out-on-sql-more-exploring
categories:
  - sql
tags:
  - sql
keywords:
  - tech
---




# Intermediate Challenge

## 1. Do longer or shorter length albums tend to generate more revenue?
        Hint: We can use the WITH clause to create a temporary table that determines the number of tracks in each album, then group by the length of the album to compare the average revenue generated for each.
## 2. Is the number of times a track appear in any playlist a good indicator of sales?
        Hint: We can use the WITH clause to create a temporary table that determines the number of times each track appears in a playlist, then group by the number of times to compare the average revenue generated for each.

# Advanced Challenge

## 1. How much revenue is generated each year, and what is its percent change 4 from the previous year?
        Hint: The InvoiceDate field is formatted as ‘yyyy-mm-dd hh:mm:ss’. Try taking a look at using the strftime() function to help extract just the year. Then, we can use a subquery in the SELECT statement to query the total revenue from the previous year. Remember that strftime() returns the date as a string, so we would need to CAST it to an integer type for this part. Finally, since we cannot refer to a column alias in the SELECT statement, it may be useful to use the WITH clause to query the previous year total in a temporary table, and then calculate the percent change in the final SELECT statement.

