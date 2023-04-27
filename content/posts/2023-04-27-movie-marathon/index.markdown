---
title: Tidy Tuesday - Movie Marathon
author: Rich Posert
date: '2023-04-27'
slug: []
categories: ['Data Visualization']
tags: ['TidyTuesday']
summary: It turns out movies and marathons are now about the same length!
cover:
  image: movie_marathon.png
  alt: "Lineplots showing the median movie time and the winning London marathon time.
  The lines cross for the first time in 2022"
  relative: true
---

No time for a full walkthrough today! Here's the code, and then I tidied it up
a bit in Affinity designer. I will say, I was very surprised that the median movie
in the IMDB dataset is still around 90 minutes!

```
library(tidyverse)

ttues <- tidytuesdayR::tt_load('2023-04-25')
winners <- ttues$winners
marathon <- ttues$london_marathon

# yes, yes, everyone loves {here}. leave me alone.
movies_raw <- read_tsv('~/Downloads/title.basics.tsv')
movies <- movies_raw |> 
  filter(titleType == 'movie' & runtimeMinutes != "\\N") |> 
  mutate(
    startYear = as.numeric(startYear),
    Movie_Runtime = as.numeric(runtimeMinutes)
  ) |>
  group_by(startYear) |> 
  summarize(Movie_Runtime = median(Movie_Runtime)) |> 
  mutate(Movie_Runtime = lubridate::dminutes(Movie_Runtime))

winners |> 
  mutate(cat_color = if_else(
    str_detect(Category, 'Men'),
    '#5773c0',
    '#edb144'
  )) |> 
  left_join(movies, by = c('Year' = 'startYear')) |> 
  ggplot(aes(Year, Time, group = Category)) +
  geom_line(aes(color = cat_color)) +
  geom_line(aes(y = Movie_Runtime)) +
  scale_y_time(breaks = scales::breaks_width('1 hour'), labels = scales::label_time(format = "%l hours")) +
  scale_color_identity() +
  theme_minimal() +
  theme(
    panel.grid.major.x = element_blank(),
    panel.grid.minor.x = element_blank()
  ) +
  labs(
    title = 'Movie Marathon',
    subtitle = 'You can now run a marathon faster than you can watch a movie',
    caption = "Marathon data from Nicola Rennie's LondonMarathon R package. Movie data from IMDB."
  )
ggsave('movie_marathon.pdf', width = 9, height = 6)
```

![](movie_marathon.png)
