---
title: Tidy Tuesday - Why bother
author: Rich Posert
date: '2022-05-26'
slug: []
categories: ['Data Visualization']
tags: ['TidyTuesday']
---



It's hard to focus on doing anything at all, really. I don't know anyone who thinks
the world will improve, without relying on (quite literally) divine intervention
or a more embarrassing belief in the "democratic process" under which we toil.


```r
# Uhh, import the data.

library(tidyverse)
full_tt_dataset <- tidytuesdayR::tt_load('2022-05-24')
```

```
## 
## 	Downloading file 1 of 2: `fifteens.csv`
## 	Downloading file 2 of 2: `sevens.csv`
```

```r
raw_sevens <- full_tt_dataset$sevens
raw_fifteens <- full_tt_dataset$fifteens

glimpse(raw_sevens)
```

```
## Rows: 7,966
## Columns: 16
## $ row_id     <dbl> NA, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17,…
## $ date       <date> 1997-03-15, 1997-03-15, 1997-03-15, 1997-03-15, 1997-03-15…
## $ team_1     <chr> "New Zealand Wild Ducks", "England", "Canada", "Hong Kong",…
## $ score_1    <chr> "54", "10", "31", "12", "26", "42", "29", "29", "36", "10",…
## $ score_2    <chr> "0", "7", "0", "5", "5", "0", "0", "0", "5", "7", "0", "15"…
## $ team_2     <chr> "Japan", "Australia", "Netherlands", "Fiji", "Scotland", "S…
## $ venue      <chr> "Hong Kong", "Hong Kong", "Hong Kong", "Hong Kong", "Hong K…
## $ tournament <chr> "Hong Kong Sevens", "Hong Kong Sevens", "Hong Kong Sevens",…
## $ stage      <chr> "Pool A", "Pool A", "Pool A", "Pool B", "Pool B", "Pool B",…
## $ t1_game_no <dbl> 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2, 3, 3, 3, 3, 3, 3, 4, 4,…
## $ t2_game_no <dbl> 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2, 3, 3, 3, 3, 3, 3, 4, 4,…
## $ series     <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
## $ margin     <dbl> 54, 3, 31, 7, 21, 42, 29, 29, 31, 3, 44, 2, 26, 41, 19, 21,…
## $ winner     <chr> "New Zealand Wild Ducks", "England", "Canada", "Hong Kong",…
## $ loser      <chr> "Japan", "Australia", "Netherlands", "Fiji", "Scotland", "S…
## $ notes      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
```

I think the thing that's most galling is the insistence, in the face of nearly
unlimited evidence to the contrary, that *more* guns and *more* cops will make us
safer. That making our schools prisons with armed SWAT teams patrolling them
in full tactical gear will build the society we want. That giving teachers weapons
will make the already-questionable hierarchy between student and teacher anything
but more menacing.


```r
# I wonder if there's something interesting in margins
raw_sevens |> 
  ggplot(aes(margin)) +
  theme_minimal() +
  geom_density()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />


Here in Portland, our chief of police
[put a high school girl in a headlock and handcuffed her](https://www.portlandmercury.com/blogtown/2020/06/08/28519546/portland-police-chief-jami-resch-resigns)
for talking back to the dean.


```r
# I guess, let's maybe look at who ties the most often
draws <- raw_sevens |> 
  filter(margin == 0) |> 
  pivot_longer( # I don't care who was team 1 and who was team 1, just that they both tied
    cols = c('team_1', 'team_2'),
    names_to = 'team',
    names_prefix = 'team_', # not that we're going to use this column, but let's be neat
    values_to = 'Country') |> 
  count(Country) |> 
  rename('Draws' = n) |> 
  arrange(-Draws)

draws
```

```
## # A tibble: 95 × 2
##    Country   Draws
##    <chr>     <int>
##  1 Russia       11
##  2 Spain        11
##  3 Canada        8
##  4 Colombia      8
##  5 Fiji          8
##  6 Hong Kong     8
##  7 Hungary       8
##  8 China         7
##  9 England       7
## 10 Portugal      7
## # … with 85 more rows
```

I was not really prepared for how often in my adult life the right thing to do, in so many ways and in so
many places, would be extremely obvious and how often people in power would deny
it.


```r
# Maybe funny to see the relationship between tying and winning? My heart's not
# really in this.

raw_sevens |> 
  left_join(draws, by = c('team_1' = 'Country')) |> 
  replace_na(list(Draws = 0)) |>  # countries who never draw should be 0
  filter(winner == team_1) |> 
  group_by(team_1) |> 
  summarize(Wins = n(), Draws = mean(Draws)) |> 
  ggplot(aes(Draws, Wins)) +
  theme_minimal() +
  geom_jitter(width = 0.1) +
  labs(
    x = 'Times a game has resulted in a draw',
    y = 'Total team wins',
    title = 'Whatever'
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

I doubt it can be like this forever, but I don't know whether to anticipate or fear what comes next.
