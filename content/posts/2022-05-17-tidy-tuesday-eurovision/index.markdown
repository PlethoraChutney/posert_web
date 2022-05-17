---
title: Tidy Tuesday - Eurovision
author: Rich Posert
date: '2022-05-17'
slug: []
categories: ['Data Visualization']
tags: ['TidyTuesday']
---




I know next to nothing about Eurovision, so this will be a learning exercise
in more ways than one. I know it's a song competition, that each country submits
one contestant, and that Australia is in it. I also think that there's a meme
about the UK being bad at it.

## First Impressions

Let's load up the data and see what we're dealing with:


```r
library(tidyverse)
full_tt_dataset <- tidytuesdayR::tt_load('2022-05-17')
```

```
## 
## 	Downloading file 1 of 2: `eurovision.csv`
## 	Downloading file 2 of 2: `eurovision-votes.csv`
```

```r
eurovision <- full_tt_dataset$eurovision
votes <- full_tt_dataset$`eurovision-votes`

glimpse(eurovision)
```

```
## Rows: 2,005
## Columns: 18
## $ event          <chr> "Turin 2022", "Turin 2022", "Turin 2022", "Turin 2022",…
## $ host_city      <chr> "Turin", "Turin", "Turin", "Turin", "Turin", "Turin", "…
## $ year           <dbl> 2022, 2022, 2022, 2022, 2022, 2022, 2022, 2022, 2022, 2…
## $ host_country   <chr> "Italy", "Italy", "Italy", "Italy", "Italy", "Italy", "…
## $ event_url      <chr> "https://eurovision.tv/event/turin-2022", "https://euro…
## $ section        <chr> "first-semi-final", "first-semi-final", "first-semi-fin…
## $ artist         <chr> "Kalush Orchestra", "S10", "Amanda Georgiadi Tenfjord",…
## $ song           <chr> "Stefania", "De Diepte", "Die Together", "Saudade, Saud…
## $ artist_url     <chr> "https://eurovision.tv/participant/kalush-orchestra-22"…
## $ image_url      <chr> "https://static.eurovision.tv/hb-cgi/images/963164d0-06…
## $ artist_country <chr> "Ukraine", "Netherlands", "Greece", "Portugal", "Bulgar…
## $ country_emoji  <chr> ":flag_ua:", ":flag_nl:", ":flag_gr:", ":flag_pt:", ":f…
## $ running_order  <dbl> 6, 8, 15, 10, 7, 5, 17, 16, 3, 9, 4, 14, 11, 1, 12, 2, …
## $ total_points   <dbl> 337, 221, 211, 208, 29, 15, 187, 177, 159, 154, 118, 10…
## $ rank           <dbl> 1, 2, 3, 4, 16, 17, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, …
## $ rank_ordinal   <chr> "1st", "2nd", "3rd", "4th", "16th", "17th", "5th", "6th…
## $ qualified      <lgl> TRUE, TRUE, TRUE, TRUE, FALSE, FALSE, TRUE, TRUE, TRUE,…
## $ winner         <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE,…
```

```r
glimpse(votes)
```

```
## Rows: 56,312
## Columns: 8
## $ year               <dbl> 1975, 1975, 1975, 1975, 1975, 1975, 1975, 1975, 197…
## $ semi_final         <chr> "f", "f", "f", "f", "f", "f", "f", "f", "f", "f", "…
## $ edition            <chr> "1975f", "1975f", "1975f", "1975f", "1975f", "1975f…
## $ jury_or_televoting <chr> "J", "J", "J", "J", "J", "J", "J", "J", "J", "J", "…
## $ from_country       <chr> "Belgium", "Belgium", "Belgium", "Belgium", "Belgiu…
## $ to_country         <chr> "Belgium", "Finland", "France", "Germany", "Ireland…
## $ points             <dbl> 0, 0, 2, 0, 12, 1, 6, 0, 7, 0, 0, 0, 4, 0, 8, 3, 0,…
## $ duplicate          <chr> "x", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
```

Wow! The first ideas that jump out to me are:
 1. How often does the host country win? Is there a home field advantage?
 
This question
would be difficult to answer though, since I have no idea how often a country is expected
to win in the first place. Presumably if you (as a country) love Eurovision, you're more
likely to both find good acts and to want to host it, so I doubt the two events can
be treated as independent.

 2. Which artists have competed the most times? And when?
 3. Are the trajectories of countries' placements (i.e., winner, second place, etc.) interesting?
 4. Are voters more likely to vote for their own country if they're the host?
 
Ignoring question 1 since I don't think I'd be able to answer it, let's explore the
data:

## Prolific Artists


```r
eurovision |> 
  count(artist) |> 
  count(n)
```

```
## # A tibble: 4 × 2
##       n    nn
##   <int> <int>
## 1     1  1159
## 2     2   355
## 3     3    32
## 4     4    10
```

Alright, nobody has performed more than 4 times, and ten people have performed
4 times. Not particularly interesting.

## Rags to Riches

I wonder if we can see any rags-to-riches or fall-from-grace stories in how
well the countries do each year:


```r
eurovision |> 
  ggplot(aes(x = year, y = rank, group = artist_country)) +
  geom_line()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/country-placements-1.png" width="672" />

Hm...that's hard to read and I'm realising it's also upside down (1st place should be
at the top).

Maybe if we filter to just countries who've ever won a Eurovision contest we can
see a bit more of what I'm looking for.


```r
winners <- eurovision |> 
  filter(winner)

winners <- unique(winners$artist_country)

eurovision |>
  filter(artist_country %in% winners) |> 
  ggplot(aes(x = year, y = rank, group = artist_country)) +
  geom_line() +
  scale_y_reverse()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/filter-first-1.png" width="672" />

Turns out only three countries have never won Eurovision. Actually, that's funny.
Who are they?


```r
eurovision |> 
  filter(!(artist_country %in% winners)) |> 
  ggplot(aes(x = year, y = rank, color = artist_country)) +
  geom_line() +
  scale_y_reverse()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

You'll get there soon Andorra, Morocco, and Slovakia!

As a last-ditch attempt to look at this question, let's see if we can target
countries who started Eurovision in the bottom third and have recently placed
in the top third.


```r
# find bottom third cutoffs for each year
loser_cutoffs <- eurovision |> 
  filter(section == 'grand-final') |> 
  group_by(year) |> 
  summarise(bottom_third = as.integer(n() - (n() / 3)))

# find each country's "early years"
early_years <- eurovision |> 
  group_by(artist_country) |> 
  summarize(first_year = min(year)) |> 
  mutate(maturity = first_year + 5)

# mark whether a country was in the bottom third for each row
rags_to_riches <- eurovision |> 
  left_join(loser_cutoffs, by = 'year') |> 
  mutate(loser = rank >= bottom_third) |> 
  left_join(early_years, by = 'artist_country') |> 
  mutate(is_early_year = year >= first_year & year < maturity) |> 
  select(-first_year, -maturity)

# filter just to grand final...not sure what the section is exactly
rags_to_riches <- rags_to_riches |> 
  filter((is_early_year | year > 2022-5) & section == 'grand-final') |> 
  group_by(artist_country, is_early_year) |> 
  summarize(num_losses = sum(loser, na.rm = TRUE))

rags_to_riches |> 
  mutate(is_early_year = if_else(is_early_year, 'Early', 'Late')) |> 
  pivot_wider(names_from = is_early_year, values_from = num_losses) |> 
  replace_na(list(Early = 0)) |> 
  filter(Early > Late)
```

```
## # A tibble: 2 × 3
## # Groups:   artist_country [2]
##   artist_country  Late Early
##   <chr>          <int> <int>
## 1 Australia          0     1
## 2 Moldova            0     1
```

That was a lot of work! And all to find a major shortcoming of my knowledge of Eurovision --- I didn't realize this competition
had multiple rounds!

OK, that question is a bust.

## Voter Bias

Alright, one final try at this week's tidy tuesday on a topic I know nothing about.

Unfortunately, there's not really any documentation on this "duplicate" column,
so I'll drop anything with a value in there.


```r
votes |> 
  mutate(self_vote = from_country == to_country) |> 
  filter(self_vote)
```

```
## # A tibble: 1,530 × 9
##     year semi_final edition jury_or_televoting from_country to_country points
##    <dbl> <chr>      <chr>   <chr>              <chr>        <chr>       <dbl>
##  1  1975 f          1975f   J                  Belgium      Belgium         0
##  2  1975 f          1975f   J                  Finland      Finland         0
##  3  1975 f          1975f   J                  France       France          0
##  4  1975 f          1975f   J                  Germany      Germany         0
##  5  1975 f          1975f   J                  Ireland      Ireland         0
##  6  1975 f          1975f   J                  Israel       Israel          0
##  7  1975 f          1975f   J                  Italy        Italy           0
##  8  1975 f          1975f   J                  Luxembourg   Luxembourg      0
##  9  1975 f          1975f   J                  Malta        Malta           0
## 10  1975 f          1975f   J                  Monaco       Monaco          0
## # … with 1,520 more rows, and 2 more variables: duplicate <chr>,
## #   self_vote <lgl>
```
Ah. The duplicate column is because country voters cannot vote for themselves.

## Conculsion
Well, I hope you enjoyed this exercise of "Futility three ways". In the interest
of practicing in public I'll still post this, but obviously no real plots came out
of it. Some of the tricks from the "Rags to Riches" section are probably still interesting,
though! And the main lesson here is to cut your losses if you're in over your head.
