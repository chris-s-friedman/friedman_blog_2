---
title: Importing Social Network Data into R
author: ''
template: "post"
date: '2017-12-19'
slug: importing-social-network-data-into-r
category: "Social Network Analysis"
tags:
  - r
  - Social Network Analysis
  - igraph
  - egocentric networks
description: "In my professional life, I manage and analyze data on a team that studies the social networks surrounding children with autism. The purpose of this post is not to discuss that work in depth, but rather to show how to quickly and easily import one type of data I work with into R. "
---

```{r, echo=FALSE}
library(knitr)
library(printr)
```


In my professional life, I manage and analyze data on a team that studies the social networks surrounding children with autism. The purpose of this post is not to discuss that work in depth, but rather to show how to quickly and easily import one type of data I work with into R. For social network analysis, I use the package [igraph](http://igraph.org/r/).

The type of data I'm going to talk about importing today is egocentric network data. For those that don't know, egocentric network data involves asking a single person about the makeup of an entire network.

## Collecting Egocentric Network Data
To demonstrate, imagine that I'm interested in social networks at the gym and that you go to the gym with a three friends every week. I approach you and after asking you to join my study, ask you who you go the gym with.

You say "James, Jen, and Rene."

Then I ask some questions about the group in relation to James:

> How many times a week do you see James?

> How many times a week does James see Jen?

> How many times a week does James see Rene?

We then do the same for Jen and Rene where I ask you often you see Jen and how often she sees Rene, and then I ask how often you see Rene.[^1]

My survey here came in two parts:

1. A **name generatorr** where I ask you who you interact with
2. A **name modifier** where I ask you how you interact with each person

After the data are collected, I log it all in my spreadsheet where each row corresponds to a data collection instance.

So, I have data that look like this:

```{r, message=FALSE}
library(dplyr)

gym_networks <- data_frame(
  participant = c("You", "Bart", "Lisa"),
  p1 = c("James", "Milhouse", "Sherry"),
  p2 = c("Jen", "Nelson", "Terry"),
  p3 = c("Rene", "Martin", "Ralph"),
  participant_x_p1 = sample(1:10, 3),
  p1_x_p2 = sample(1:10, 3),
  p1_x_p3 = sample(1:10, 3),  
  participant_x_p2 = sample(1:10, 3),
  p2_x_p3 = sample(1:10, 3),
  participant_x_p3 = sample(1:10, 3)
)
print(gym_networks)
```

## Getting the data ready for analysis

The data here are represented in an adjacency list with weights attached. Although igraph has a function for importing adjacency lists, it isn't not configured to handle weights, so we will take our adjacency list and convert it into an edge list, which igraph can handle with weights.

To accomplish this, we'll use the package [tidyr](http://tidyr.tidyverse.org/).

```{r, message=FALSE}
library(tidyr)

el <- gym_networks %>%
  # Step 1: Make each row a single edge
  gather(key, value = "weight", -(participant:p3)) %>%
  # Step 2: Configure two new columns, an ego, and an alter
  mutate(ego = case_when(grepl("participant", key) ~ participant,
                         grepl("p1_", key) ~ p1,
                         grepl("p2_", key) ~ p2,
                         grepl("p3_", key) ~ p3),
         alter = case_when(grepl("_p1", key) ~ p1,
                           grepl("_p2", key) ~ p2,
                           grepl("_p3", key) ~ p3)) %>%
  # Step 3: Clean up the data frame
  select(ego, alter, weight)

print(head(el))
```

In three steps, we go from adjacency list to edge list. In one more step, we have an igraph object to analyse and plot!

```{r, message=FALSE}
library(igraph)

graph <- graph_from_data_frame(el)
```

Happy analyzing, friends!


[^1]: I don't ask how often Jen sees James and how often Rene sees James or Jen because we assume that it takes two to tango in this respect.
