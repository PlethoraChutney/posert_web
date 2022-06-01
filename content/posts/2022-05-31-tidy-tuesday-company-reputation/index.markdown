---
title: Tidy Tuesday - Company reputation
author: Rich Posert
date: '2022-05-31'
slug: []
categories: ['Data Visualization']
tags: ['TidyTuesday']
cover:
  image: 'cover.png'
  alt: 'A bar chart showing that Americans generally do not trust banks, but they trust them more than you would think.'
  relative: true
---



This week we're diving into company reputation scores. I famously love companies,
so this should be fun.


```r
library(tidyverse)
library(ggokabeito)

full_data <- tidytuesdayR::tt_load('2022-05-31')
```

```
## 
## 	Downloading file 1 of 2: `reputation.csv`
## 	Downloading file 2 of 2: `poll.csv`
```

```r
raw_rep <- full_data$reputation
raw_poll <- full_data$poll
```

As always, there's a good discussion of methodology on the
[TidyTuesday github](https://github.com/rfordatascience/tidytuesday/tree/master/data/2022/2022-05-31)
but here's my brief synopsis: this dataset is the result of two polls. The first found
which were America's 100 most-visible companies. These companies were then ranked by
another set of people on a couple categories.

The `poll` dataframe gives us the rank, score, and change in rank of each company.
Additionally, we get their industry and their RQ (higher is better) over the course
of several years. The `reputation`
dataframe gives us more in-depth information (with the ranks of each company in each
category), but only for this year.


```r
glimpse(raw_poll)
```

```
## Rows: 500
## Columns: 8
## $ company     <chr> "Trader Joe's", "Trader Joe's", "Trader Joe's", "Trader Jo…
## $ industry    <chr> "Retail", "Retail", "Retail", "Retail", "Retail", "Retail"…
## $ `2022_rank` <dbl> 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 3, 3, 3, 3, 3, 4, 4, 4, 4, 4…
## $ `2022_rq`   <dbl> 82.4, 82.4, 82.4, 82.4, 82.4, 82.0, 82.0, 82.0, 82.0, 82.0…
## $ change      <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, -2, -2, -2, -2, -2…
## $ year        <dbl> 2017, 2018, 2019, 2020, 2021, 2017, 2018, 2019, 2020, 2021…
## $ rank        <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, 9, 3, 32, 1, N…
## $ rq          <dbl> NA, NA, 78.20, 80.70, NA, NA, 81.10, 82.50, 83.10, NA, NA,…
```

```r
glimpse(raw_rep)
```

```
## Rows: 700
## Columns: 5
## $ company  <chr> "Trader Joe's", "Trader Joe's", "Trader Joe's", "Trader Joe's…
## $ industry <chr> "Retail", "Retail", "Retail", "Retail", "Retail", "Retail", "…
## $ name     <chr> "TRUST", "ETHICS", "GROWTH", "P&S", "CITIZENSHIP", "VISION", …
## $ score    <dbl> 82.7, 82.5, 84.1, 83.5, 80.0, 81.9, 83.1, 83.7, 81.8, 83.6, 8…
## $ rank     <dbl> 3, 2, 2, 9, 3, 13, 1, 1, 4, 4, 11, 1, 19, 8, 4, 5, 15, 5, 2, …
```

# Reputation Trends

OK we have to start with some obvious line charts --- has anyone had any fun trends?


```r
raw_poll |> 
  ggplot(aes(x = year, y = rq, group = company)) +
  theme_minimal() +
  geom_line() +
  facet_wrap(vars(industry))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

Hahahaha I'm sorry, "Financial Services" is going UP?? Amazing. Tech going down
makes sense, and I'm not surprised that Pharma gets a boost in 2020 and onward.
Bet that line is a maker of a vaccine:


```r
raw_poll |> 
  filter(industry == 'Pharma') |>  
  ggplot(aes(x = year, y = rq, color = company)) +
  theme_minimal() +
  geom_line(size = 2)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Hahaha amazing. Yeah fuck that one-dose vaccine.

## Pfizer vs. J&J

Actually wait that's wild. Pfizer's
score is 15 points higher than it was in 2019, while J&J has continued its slow
decline. Where is Pfizer beating out J&J?


```r
raw_rep |> 
  filter(industry == 'Pharma') |> 
  mutate(company = if_else(company == 'Johnson & Johnson', 'J&J', company)) |> 
  ggplot(aes(x = company, y = score)) +
  theme_minimal() +
  geom_col() +
  facet_wrap(vars(name), ncol = 4)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

Huh...that doesn't look as dramatic as the line chart does it. Guess that's why
you want bars to start at zero when you're not looking at a trend! It would be
easier to compare differences this small with a table:


```r
raw_rep |> 
  filter(industry == 'Pharma') |> 
  select(company, score, name) |> 
  pivot_wider(names_from = company, values_from = score) |> 
  mutate(Difference = Pfizer - `Johnson & Johnson`) |> 
  arrange(-Difference)
```

```
## # A tibble: 7 × 4
##   name        Pfizer `Johnson & Johnson` Difference
##   <chr>        <dbl>               <dbl>      <dbl>
## 1 GROWTH        80.8                74.5       6.3 
## 2 CITIZENSHIP   73.4                68         5.40
## 3 VISION        79.1                74.8       4.30
## 4 TRUST         73.9                70.1       3.80
## 5 CULTURE       76.5                73.3       3.20
## 6 ETHICS        74.6                71.4       3.20
## 7 P&S           78.6                75.9       2.70
```

'GROWTH' could honestly be a corporate mission statement. Their scores are closest
for "Product and Service" so I rescind my making fun of people not liking J&J's
vaccine. It would be fun to try to figure out what saved Pfizer's reputation,
since J&J doesn't seem to have enjoyed the same benefit, but we'd need the broken-out
data by year for that I think.

## Media
At the risk of infuriating myself:


```r
raw_poll |> 
  filter(industry == 'Media') |>  
  ggplot(aes(x = year, y = rq, color = company)) +
  theme_minimal() +
  geom_line(size = 2) +
  scale_color_okabe_ito()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

Not as bad as I would have guessed! People like Comcast? Wild.

## What the hell is going on
Why are all of these trust numbers going *up*? My function of Company Trust over
time is very different. Groceries, Ecommerce, Retail, and Tech all seem to have
some downward trends I suppose. But Financial Services...jesus.


```r
raw_poll |> 
  filter(industry == 'Financial Services') |>  
  ggplot(aes(x = year, y = rq, color = company)) +
  theme_minimal() +
  geom_line(size = 2) +
  scale_color_okabe_ito()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />

Haha. I guess Wells Fargo is recovering slowly from the 
["We gave you a ton of accounts you didn't want"](https://www.nytimes.com/2020/02/21/business/wells-fargo-settlement.html) stuff.

OK we have to look at this. If people say these companies are like, ethical bastions
I'm going to fucking lose it.


```r
raw_rep |> 
  filter(industry == 'Financial Services' | company == "Trader Joe's") |> 
  filter(name %in% c('CITIZENSHIP', 'ETHICS', 'TRUST')) |> 
  mutate(company = fct_reorder(company, score)) |> 
  ggplot(aes(x = company, y = score, fill = company)) +
  theme_minimal() +
  geom_col() +
  facet_wrap(vars(name), ncol = 4) +
  scale_x_discrete(labels = NULL) +
  scale_fill_okabe_ito() +
  coord_cartesian(ylim = c(50,NA))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

I've adjusted the Y axis here, because anything below 50 and you're "Critical",
which to me feels like 0. I have a hard time believing that any of these banks
should break that threshold, but at least nobody is beating the paragon of Suburban
Virtue, Trader Joe's.

This is good enough for a final plot for me. Let's customize
the breaks here and tidy up the labels.


```r
raw_rep |> 
  filter(industry == 'Financial Services' | company == "Trader Joe's") |> 
  filter(name %in% c('CITIZENSHIP', 'ETHICS', 'TRUST')) |> 
  mutate(name = str_to_title(name)) |> 
  mutate(company = fct_reorder(company, score)) |> 
  ggplot(aes(x = company, y = score, fill = company)) +
  theme_minimal() +
  geom_col() +
  facet_wrap(vars(name), ncol = 4) +
  scale_x_discrete(labels = NULL) +
  scale_fill_okabe_ito() +
  coord_cartesian(ylim = c(50,NA)) +
  scale_y_continuous(
    breaks = seq(50, 80, 5),
    labels = c('Critical', 'Very Poor', 'Poor', 'Fair', 'Good', 'Very Good', 'Excellent')
  ) +
  labs(
    title = 'How much do Americans trust Banks?',
    subtitle = '...compared to Trader Joes',
    y = 'RQ Score',
    x = 'Company',
    fill = 'Company'
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="672" />

# Conclusion

OK, that was fun. All of the executives of these banks should be put against the wall.

Haha.
