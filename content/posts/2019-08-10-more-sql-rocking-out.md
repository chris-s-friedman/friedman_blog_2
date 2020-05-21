---
title: More SQL - Rocking Out
author: ''
template: "post"
date: '2019-08-10'
slug: more-sql-rocking-out
category: "sql"
tags:
  - sql
keywords:
  - tech
draft: true
description: "An Essay on Typography by Eric Gill takes the reader back to the year 1930. The year when a conflict between two worlds came to its term. The machines of the industrial world finally took over the handicrafts."
---

I've been playing around with the data science projects from codeacademy for a few weeks. First I played with stocks data [here](../../07/creating-a-stocks-database/) and [here](../../07/working-with-stocks-basic-sql).

Today, I'll be working through the second [codeacademy data science independant project: Watching the Stock Market]

This is a continuation of my work on the [codeacademy data science independant project #2: Explore a Sample Database](https://discuss.codecademy.com/t/data-science-independent-project-2-explore-a-sample-database/419945).

The database for this project is related to a fictional music store. As you can see in the diagram of the image below, it's a little more complicated than the single table stocks database!

![database diagram](/post/2019-08-10-more-sql-rocking-out_files/sqlite-sample-database-color.jpg)

Like the last project, this one is broken up into a sets of challenges. In an upcoming post, I'll examine the intermediate and advanced challenges

```{r setup}
library(DBI)
db = dbConnect(RSQLite::SQLite(), dbname = "../../data/Music Store - Chinook/chinook.db")
```

# Basic Challenges

## 1. Which tracks appeared in the most playlists? How many playlist did they appear in?

```{sql connection=db}
   SELECT track_list.TrackId,
          name,
          -- compute how many playlists each track is in
          COUNT(track_list.TrackId) AS playlist_count
     FROM playlist_track AS track_list
LEFT JOIN tracks
       ON track_list.TrackId = tracks.TrackId
       GROUP BY track_list.TrackId
   -- compare the count of each track to the max count
   HAVING COUNT(track_list.TrackId) =
          -- compute the max count here
          (SELECT MAX(plist_counts.playlist_count)
            FROM (SELECT TrackId, COUNT(*) AS playlist_count
                    FROM playlist_track
                GROUP BY TrackId) AS plist_counts)
```

Putting this together required using two tables - playlist_track, which connects TrackId to PlaylistId and the tracks table, which has information about each song. The playlist_track table has a row for each track - playlist combination. By counting how many times a track shows up in this table, we get a count of how many playlists that track is in. Then if you filter the playlist_track table so that it only shows the tracks that show up the max number of times, you see the tracks that are in the most playlists. The last bit that needs to be done after that is to bring in the track name, as I did with the `LEFT JOIN`.

## 2. Which track generated the most revenue? which album? which genre?

```{sql connection=db}
   SELECT invoice_items.TrackId, Name, SUM(invoice_items.UnitPrice * Quantity) AS track_revenue
     FROM invoice_items
LEFT JOIN tracks
       ON invoice_items.TrackId = tracks.TrackId
 GROUP BY invoice_items.TrackId
 ORDER BY track_revenue DESC;
```

So we can see that the top 8 tracks above garnered the most revenue. I didn't look these up, but I don't recognize any of them ¯\\_(ツ)_/¯.

What about Albums? Which ALbum garnered the most revenue?
```{sql connection=db}
   SELECT tracks.AlbumID, Title, SUM(invoice_items.UnitPrice * Quantity) AS album_revenue
     FROM invoice_items
LEFT JOIN tracks
       ON invoice_items.TrackId = tracks.TrackId
LEFT JOIN albums
       ON tracks.AlbumID = albums.AlbumID
 GROUP BY tracks.AlbumID
 ORDER BY album_revenue DESC;
```
The most revenue generating album? Looks like it isn't even an album and more the first season of Battlestar Galactica!

Now what about Genre?
```{sql connection=db}
   SELECT tracks.GenreId,
          genres.Name,
          SUM(invoice_items.UnitPrice * Quantity) AS genre_revenue
     FROM invoice_items
LEFT JOIN tracks
       ON invoice_items.TrackId = tracks.TrackId
LEFT JOIN genres
       ON tracks.GenreId = genres.GenreId
 GROUP BY tracks.GenreId
 ORDER BY genre_revenue DESC;
```

The most revenue generating? Rock!

## 3. Which countries have the highest sales revenue? What percent of total revenue does each country make up?

In the last challenge, we examined how we can figure out which items in the database produce the most revenue and further which groups of items produce the most revenue. The conclusion from all of that being is that investing in more of those classes of items can produce more revenue.

In this challenge we will use metadata about customers to examine where in the world (literally) the revenue comes from. As the challenge asks, what countries produce the most revenue? Importantly, we can get country daya from two different sources: customer country and billing country. I'm more curious about customer country, so we'll go there.

```{sql connection=db}
   SELECT country,
          SUM(invoice_items.UnitPrice * Quantity) AS country_revenue,
          ROUND(SUM(invoice_items.UnitPrice * Quantity) /
                (SELECT SUM(invoice_items.UnitPrice * Quantity)
                   FROM invoice_items) * 100,
                2) AS percent_total_revenue
     FROM invoice_items
LEFT JOIN invoices
       ON invoice_items.InvoiceId = invoices.InvoiceId
LEFT JOIN customers
       ON invoices.CustomerId = customers.CustomerId
 GROUP BY country
 ORDER BY country_revenue DESC;
```

Well, there you have it! The US generated the most revenue out of all the countries - $523.06, which is 22.46% of total revenue.

## 4. How many customers did each employee support, what is the average revenue for each sale, and what is their total sale?

With this question, we are trying to get at how personnel impact revenue.

How do employees interact with customers? How many customers is each employee helping?


```{sql connection=db}
   SELECT employees.FirstName,
          employees.LastName,
          COUNT(*) AS customers_helped
     FROM customers
LEFT JOIN employees
       ON customers.SupportRepId = employees.EmployeeId
 GROUP BY SupportRepId
```

Well, looks like they all help customers just about the same amount.

Okay, so now we know how customers are helped by the sales team, but how much do customers spend on average?

```{sql connection=db}
   SELECT AVG(Total) AS average_sale
     FROM invoices
```

So now that we know how many customers each sales rep helps and the average amount of money a customer spends in a single session, how much revenue is each employee responsible for?

```{sql connection=db}
   SELECT employees.FirstName,
          employees.LastName,
          SUM(invoices.Total) AS total_revenue
     FROM invoices
LEFT JOIN customers
       ON invoices.CustomerId = customers.CustomerId
LEFT JOIN employees
       ON customers.SupportRepId = employees.EmployeeId
 GROUP BY customers.SupportRepId
```

Well this just goes to show that interacting with more customers is at least related to total revenue generated by an employee. That said, with an N of 3, you can't really learn much.
