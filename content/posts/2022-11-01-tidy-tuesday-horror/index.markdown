---
title: Tidy Tuesday - Horror
author: Rich Posert
date: '2022-11-01'
slug: []
categories: ['Data Visualization']
tags: ['TidyTuesday']
summary: A quick investiation into what horror subgenres are most popular.
cover:
  image: cover.png
  alt: "Ridgelines showing the density of mean movie rating for various horror
  subgenres"
  relative: true
  
---



Horror movies! Spooky.


```r
library(tidyverse)

movies <- read_csv('data/horror_movies.csv')

glimpse(movies)
```

```
## Rows: 32,540
## Columns: 20
## $ id                <dbl> 760161, 760741, 882598, 756999, 772450, 1014226, 717…
## $ original_title    <chr> "Orphan: First Kill", "Beast", "Smile", "The Black P…
## $ title             <chr> "Orphan: First Kill", "Beast", "Smile", "The Black P…
## $ original_language <chr> "en", "en", "en", "en", "es", "es", "en", "en", "en"…
## $ overview          <chr> "After escaping from an Estonian psychiatric facilit…
## $ tagline           <chr> "There's always been something wrong with Esther.", …
## $ release_date      <date> 2022-07-27, 2022-08-11, 2022-09-23, 2022-06-22, 202…
## $ poster_path       <chr> "/pHkKbIRoCe7zIFvqan9LFSaQAde.jpg", "/xIGr7UHsKf0URW…
## $ popularity        <dbl> 5088.584, 2172.338, 1863.628, 1071.398, 1020.995, 93…
## $ vote_count        <dbl> 902, 584, 114, 2736, 83, 1, 125, 1684, 73, 1035, 637…
## $ vote_average      <dbl> 6.9, 7.1, 6.8, 7.9, 7.0, 1.0, 5.8, 7.0, 6.5, 6.8, 7.…
## $ budget            <dbl> 0, 0, 17000000, 18800000, 0, 0, 20000000, 68000000, …
## $ revenue           <dbl> 9572765, 56000000, 45000000, 161000000, 0, 0, 289259…
## $ runtime           <dbl> 99, 93, 115, 103, 0, 0, 88, 130, 90, 106, 98, 89, 97…
## $ status            <chr> "Released", "Released", "Released", "Released", "Rel…
## $ adult             <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FAL…
## $ backdrop_path     <chr> "/5GA3vV1aWWHTSDO5eno8V5zDo8r.jpg", "/2k9tBql5GYH328…
## $ genre_names       <chr> "Horror, Thriller", "Adventure, Drama, Horror", "Hor…
## $ collection        <dbl> 760193, NA, NA, NA, NA, NA, 94899, NA, NA, 950289, N…
## $ collection_name   <chr> "Orphan Collection", NA, NA, NA, NA, NA, "Jeepers Cr…
```
This is a big ol' table of all the horror movies in
[The Movie Database](https://www.themoviedb.org/).
Votes seem to be called Ratings on the website, and instead of the decmimal out
of ten they're instead given on a percent scale, but whatever. Also, I'm assuming
that when a metric is `0` it's actually `NA` (doubt any of these were
made for free).

I like that we have combination genres --- we could see what genres go well with
horror by comparing ratings by genre. I'm going to use ratings, not 
[popularity](https://developers.themoviedb.org/3/getting-started/popularity),
because popularity involves a recency component that I'm not interested in 
for this purpose.


```r
unnest_genres <- movies |> 
  mutate(genre_names = str_split(genre_names, ', ')) |> 
  unnest(genre_names)

unnest_genres |> 
  filter(genre_names != 'Horror' & vote_average != 0) |>
  ggplot(aes(vote_average)) +
  theme_minimal() +
  geom_histogram(
    aes(y = ..ncount.., fill = genre_names),
    bins = 20
  ) +
  facet_wrap(vars(genre_names), ncol = 3)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Hard to tell with these plots, but this might be fun. Let's try another way of
showing the data:


```r
library(ggridges)

unnest_genres |> 
  filter(genre_names != 'Horror' & vote_average != 0) |> 
  mutate(genre_names = fct_reorder(
    genre_names, vote_average, .fun = median
    )) |> 
  ggplot(aes(vote_average, genre_names, fill = genre_names)) +
  theme_minimal() +
  geom_density_ridges(
    show.legend = FALSE
  ) +
  coord_cartesian(xlim = c(0,10)) +
  scale_x_continuous(
    breaks = 1:10
  ) +
  labs(
    x = 'Rating',
    y = 'Genre'
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Let's add back in the "All horror" category to compare against:


```r
sysfonts::font_add_google("Creepster","Creepster")
showtext::showtext_auto()

red_scale <- scales::seq_gradient_pal(
  low = '#701108',
  high = '#F02511'
)(seq(0,1,length.out = 18))
red_scale <- c(red_scale, '#888888')

unnest_genres |> 
  mutate(genre_names = if_else(
    genre_names == 'Horror',
    'All Horror', genre_names
  )) |> 
  filter(vote_average != 0) |>
  mutate(genre_names = fct_reorder(genre_names, vote_average)) |> 
  mutate(genre_names = fct_relevel(genre_names, 'All Horror', after = Inf)) |> 
  ggplot(aes(vote_average, genre_names, fill = genre_names)) +
  theme_minimal() +
  geom_density_ridges(
    scale = 2,
    show.legend = FALSE,
    color = '#111111'
  ) +
  coord_cartesian(xlim = c(1,10)) +
  scale_x_continuous(
    breaks = 1:10
  ) +
  scale_fill_manual(
    values = red_scale
  ) +
  theme(
    plot.title.position = 'plot',
    panel.grid.major.x = element_blank(),
    panel.grid.minor.x = element_blank(),
    panel.background = element_rect(fill = 'black'),
    plot.background = element_rect(fill = 'black'),
    text = element_text(
      family = 'Creepster',
      size = 20,
      color = '#c2c2c2'
    ),
    axis.text = element_text(color = '#AAAAAA')
  ) +
  labs(
    title = "If you're making a horror movie, animate it",
    x = 'Average movie rating ',
    y = 'Genre'
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

A fun little exercise! I also like that there's a consistent desire for people
to give 10/10 ratings to movies at all genres, even if the median rating is a full
point-and-a-half different! Happy Halloween!
