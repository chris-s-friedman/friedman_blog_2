---
title: How often does the Senate Vote In Palindromes?
author: ''
template: "post"
date: '2018-01-31'
slug: how-often-does-the-senate-vote-in-palindromes
category: "Riddler"
tags:
  - five_thirty_eight
  - riddler
  - dplyr
  - stringr
  - purrr
description: "Every Friday, five thirty eight comes out with two logic, math, or probability based puzzles - one quick to solve and one that takes a long time to solve. Although I don't always get a chance to partake, thinking about them is always fun."
---

Every Friday, [five thirty eight](https://fivethirtyeight.com/) comes out with two logic, math, or probability based puzzles - one quick to solve and one that takes a long time to solve. Although I don't always get a chance to partake, thinking about them is always fun.

This week's quick puzzle related to a problem I'm trying to solve in the office, so I thought I would give this one a try. Taken from [their site](https://fivethirtyeight.com/features/how-often-does-the-senate-vote-in-palindromes/), the puzzle goes like this:

> On Monday, [the Senate voted](https://www.nytimes.com/interactive/2018/01/22/us/politics/live-senate-vote-government-shutdown2.html) 81-18 to end the government shutdown. This naturally grabbed the Riddler’s attention: It’s a palindrome! The vote tally reads the same forward and backward. This specific tally was made possible by the absence of John McCain. But do senators need to be absent to create palindrome tallies? If so, what numbers of absences will do the trick?

> _Extra credit:_ How many palindromic Senate votes have occurred in the [past three decades](https://www.senate.gov/legislative/votes.htm)?

In addition to potentially helping solve a problem in the office, I figure this is a good excuse to work on experimenting with [object oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming) and writing functions to work with [S3 methods](http://adv-r.had.co.nz/OO-essentials.html). As you will see, the function I wrote is more for fun than anything else, but I think it does the trick in showing how s3 methods work. In addition, the first solution to the puzzle I propose is verbose. At the end I show how I solved the problem in an even quicker

## Let's begin!

<center>

![](https://res.cloudinary.com/chrissfriedman/image/upload/v1517458326/drwho_allons_y_ruxxvw.gif)

</center>

The first thing I did when approaching this puzzle was to re-frame my question into something I can write code for.

##### What are the possible combinations of votes that can result in a palindrome?

When thinking about palindromes, the first thing I need to do is think about what the reversible part of the palindrome is. With senate votes, the first part of the palindrome can be considered the number of "yea" votes while the second part of it can be considered the number of "nay" votes.

Considering the number of seats in the senate, the number of yea votes can be any number between 0 and 100.

Considering the problem of palindromes will narrow down `0:100` in two ways.

First, a unanimous vote in the senate is impossible! Kidding aside, I also know that a unanimous vote can't result in a palindrome, so the case where there are 100 yea votes can be discarded. In addition, we won't look at cases were no senators vote!

Second, Looking at votes between 1 and 99 is doubling the amount of work. Votes in the yea column greater than or equal to 50 are palindromes of the number of votes below 50. So it's only necessary to look at yea votes where the number of votes is between 1 and 49.

For illustrative purposes, let's put that vector in a data frame

```{r, message=FALSE}
library(dplyr)

votes <- data_frame(yea = 1:49)
```

Now that we have a vector of all of the numbers between 0 and 49, we'll build a vector of their palindromes and put them in a new column for all of the "nay"" votes.

```{r}
library(stringr)
library(purrr)

votes <- votes %>%
  mutate(nay = formatC(yea, width = 2, format = "d", flag = "0") %>%
           str_split("") %>%
           map(rev) %>%
           map_chr(paste, collapse = "") %>%
           as.numeric())
```

In the above code chunk, I introduced a function it seems a lot of people don't get to play with very often, [`formatC`](https://www.rdocumentation.org/packages/base/versions/3.4.3/topics/formatC), an interface to the c function `printf`. Basically, it allows users to format a text input. Here, it's used to add leading zeros.

Above, there's also a call to [`str_split`](https://www.rdocumentation.org/packages/stringr/versions/1.1.0/topics/str_split) from the [`stringr`](http://stringr.tidyverse.org/) package. It's used here to split the two digits so that they can then be reversed.

`str_split` outputs a list of character vectors where each element is the two characters. To operate on this list, [`map`](https://www.rdocumentation.org/packages/purrr/versions/0.2.4/topics/map) from the [`purrr`](http://purrr.tidyverse.org/) package is used, to operate on each element in the list. In this instance `map` applies the [`rev`](https://www.rdocumentation.org/packages/base/versions/3.4.3/topics/rev) function to reverse each item in each element in the list so that `"8" "1"` becomes `"1" "8"`.

Similarly, [`map_chr`](http://purrr.tidyverse.org/reference/index.html#section-map-family) pastes the items in each element together (changing `"1" "8"` to `"18"`) and uses the `_chr` modifier to return a character vector that is then converted into a numeric one.

After computing the nay votes, we need to narrow down our list of yea's and nay's to votes that are possible in the 100 seat senate.

```{r}
votes <- votes %>%
  mutate(vote_count = yea + nay) %>%
  # Filter our impossible vote counts
  filter_at(vars(vote_count), all_vars(. <= 100))
```
If you've been playing along at home, you will notice that 10 of the vote counts were dropped.

Now we have all of the possible vote tallies that are palindromic!

## Do senators need to be absent to create palindrome tallies?

This question is answered by seeing if any of the items in `votes$vote_count` equal 100. If they do, then the answer to the question is no.

```{r}
any(votes$vote_count == 100)
```

YES! Senators DO need to be absent for a palindromic vote count!

What are the number of absences that will do the trick?
```{r}
100 - votes$vote_count %>%
  unique()
```

Above, I show the absences needed for a vote to be palindromic. That said, if 50 senators are absent, a quorum is not present, so business isn't being conducted.

In actuality, the number of absences can be 49, 34, 23, 12, or 1.
