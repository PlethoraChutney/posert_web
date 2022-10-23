---
title: Tidy Tuesday - Stranger Things
author: Rich Posert
date: '2022-10-22'
slug: []
categories: ['Data Visualization']
tags: ['TidyTuesday']
summary: "A sentiment analysis of Stranger Things caption data."
cover:
  image: cover-image.png
  alt: "Circular plots of each episode's levels of sentiment in each of ten categories: anger, anticipation, disgust, fear, joy, negative, positive, sadness, surprise, and trust."
  relative: true
---




```r
library(tidyverse)

episodes <- read_csv('episodes.csv')
dialogue <- read_csv('dialogue.csv')
```

I watched season 1 of Stranger Things when it came out and I quite liked it! Now
reading the wikipedia summaries of the recent seasons it's clear there are many
new characters, so this will be interesting.


```r
glimpse(dialogue)
```

```
## Rows: 32,519
## Columns: 8
## $ season          <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
## $ episode         <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
## $ line            <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,…
## $ raw_text        <chr> "[crickets chirping]", "[alarm blaring]", "[panting]",…
## $ stage_direction <chr> "[crickets chirping]", "[alarm blaring]", "[panting]",…
## $ dialogue        <chr> NA, NA, NA, NA, NA, NA, NA, NA, "Something is coming. …
## $ start_time      <time> 00:00:07, 00:00:49, 00:00:52, 00:01:01, 00:01:09, 00:…
## $ end_time        <time> 00:00:09, 00:00:51, 00:00:54, 00:01:02, 00:01:10, 00:…
```

Oh wow, this is very detailed! Crickets chirping, what fun! Dialogue makes
me think sentiment analysis. Say it together: "How hard could it be?"
Let's try [tidytext](https://www.tidytextmining.com/tidytext.html), mostly
following the tutorial I've linked there. I think the first step for us
is to remove any lines which are just stage direction.


```r
dialogue <- dialogue |> 
  filter(!is.na(dialogue))
```

Then, we can break each line into tokens. Tokens are in our case words, since we'll
be using unigram lexicons, but they could be groups of words ("very nice") or
whatever. But anyway.


```r
library(tidytext)

dialogue <- dialogue |> 
  unnest_tokens(word, dialogue) |> 
  select(season, episode, line, word) |> 
  glimpse()
```

```
## Rows: 143,885
## Columns: 4
## $ season  <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
## $ episode <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
## $ line    <dbl> 9, 9, 9, 9, 9, 9, 9, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 1…
## $ word    <chr> "something", "is", "coming", "something", "hungry", "for", "bl…
```

I've dropped the columns I don't expect we'll need. You can see that all this has done
is split up sentences by word, which we could probably have done with regex or
whatever. But the function can make ngrams, handle tweets, etc.

Next we pick a "lexicon". This is the mapping from word to sentiment, i.e., "bad"
should map to "negative" or whatever. This is a very consequential choice. Ideally
you'd handle things like "not bad" going to positive instead, etc., but you can
see how that would rapidly spiral out of control and you end up with LaMDA or whatever.
So we can use one of the lexicons provided to us by tidytext,
[Saif Mohammad and Peter Turney](http://saifmohammad.com/WebPages/NRC-Emotion-Lexicon.htm). I'm
choosing this one because it assigns words not a +/- integer score, but a yes/no
value for a whole list of categories:

 * positive
 * negative
 * anger
 * anticipation
 * disgust
 * fear
 * joy
 * sadness
 * surprise
 * trust
 
and now I'm imagining one of those classic radar charts of different characters etc.
Which was one of the suggestions of the TidyTuesday that broke this blog for a bit,
the character emotion analysis week, becasue I fell down an "average color of image"
rabbit hole and then my computer exploded.

Anyway, downloading a lexicon. NRC requires a license for commercial use, but I
think we're okay. This dataset was published in Saif M. Mohammad and Peter Turney. (2013), "Crowdsourcing a Word-Emotion Association Lexicon." Computational Intelligence, 29(3): 436-465.


```r
nrc <- get_sentiments('nrc')
```


Now, to combine our two tables. I thought I'd have to do some clever list-column
shenanigans but after thinking about it more (and looking at the tutorial lol)
I realized that a good ol' `inner_join()` will do it all for us.


```r
sentiment_dialogue <- inner_join(
  dialogue,
  nrc,
  by = 'word'
) |> 
  glimpse()
```

```
## Rows: 30,823
## Columns: 5
## $ season    <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
## $ episode   <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
## $ line      <dbl> 9, 9, 9, 10, 10, 10, 10, 13, 13, 18, 18, 26, 26, 26, 26, 27,…
## $ word      <chr> "coming", "hungry", "hungry", "darkness", "darkness", "darkn…
## $ sentiment <chr> "anticipation", "anticipation", "negative", "anger", "fear",…
```

This is great but I'm noticing now that this dataset doesn't include......who
spoke the line?? And I didn't filter it out earlier, I just went back and checked.
What the hell. OK, I Just went and checked the [raw data for the first episode](https://8flix.com/assets/transcripts/s/tt4574334/Stranger-Things-transcript-101-Chapter-One-The-Vanishing-of-Will-Byers.pdf) and I think these are the captioning transcripts,
not scripts. So for the most part the character name isn't in the data at all to
begin with.

That blows my radar chart idea out of the water, but we could maybe look at season
or episode averages for each of our word categories and see if, for instance,
earlier in a season the episode average for "anticipation" is higher than for others.


```r
count_sentiment <- sentiment_dialogue |> 
  group_by(episode, season) |> 
  count(sentiment) |> 
  glimpse()
```

```
## Rows: 340
## Columns: 4
## Groups: episode, season [34]
## $ episode   <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
## $ season    <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, …
## $ sentiment <chr> "anger", "anticipation", "disgust", "fear", "joy", "negative…
## $ n         <int> 60, 76, 47, 65, 46, 127, 108, 61, 30, 69, 79, 79, 73, 83, 63…
```

```r
count_sentiment |> 
  mutate(
    season = as.character(season),
    sentiment = str_to_title(sentiment)
  ) |> 
  ggplot(aes(episode, n)) +
  theme_minimal() +
  geom_line(aes(color = season, group = season), size = 1) +
  facet_wrap(vars(sentiment), ncol = 3, scales = 'free') +
  MetBrewer::scale_color_met_d('Greek') +
  scale_x_continuous(breaks = seq(1, 15, by = 2)) +
  labs(
    x = 'Episode number',
    y = 'Word count',
    color = 'Sentiment'
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/ep-averages-1.png" width="672" />

Pretty noisy and pretty flat! It is interesting to note that season 4 seems to have
a much higher emotional sentiment across the board. That makes me wonder what these
would look like if we connected them all and looked at the show as a whole, rather
than season-by-season. Kinda like those [climate spirals](https://www.climate-lab-book.ac.uk/spirals/)
that don't give me a sense of foreboding at all.


```r
count_sentiment |> 
  ggplot(aes(episode, n, color = season, group = season)) +
  theme_minimal() +
  theme(
    axis.text = element_blank(),
    axis.title = element_blank(),
    panel.grid.major = element_blank(),
    legend.position = 'none'
  ) +
  geom_line() +
  facet_wrap(vars(sentiment), ncol = 5) +
  coord_polar()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

I don't like that the lines don't join because of the different numbers of
episodes in each season. We can fix that by using a proportional scaling
of the x axis.


```r
episodes |> 
  group_by(season) |> 
  filter(episode == max(episode)) |> 
  select(season, 'max_ep' = episode) |> 
  right_join(count_sentiment, by = 'season') |> 
  mutate(prop_season = episode / max_ep) |> 
  mutate(sentiment = paste0(str_to_title(sentiment), '\n')) |> 
  ggplot(aes(prop_season, n, color = season, group = season)) +
  theme_void() +
  theme(
    legend.position = 'none'
  ) +
  geom_line(size = 1, linejoin = 'round', lineend = 'round') +
  facet_wrap(vars(sentiment), ncol = 4) +
  coord_polar() +
  MetBrewer::scale_color_met_c('OKeeffe2')
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="576" />

I think these are really fun! I like that each season ends more-or-less
where it started on each sentiment, but then the next season bumps it up a bit.
Just for curiosity sake, what would this look like on a log scale?


```r
last_plot() +
  scale_y_log10()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="576" />

Huh, that's not bad either. Makes the jump in season 4 less obvious, but it shows
something I hadn't noticed before, which is just now much more surprise is in season 4
compared to the other evaluations.

One thing I'm worried about now is that even though each season has the same-ish
number of episodes, season 4 might just have way more *words* than the other seasons.
Let's make sure that's not what's happening:


```r
count_sentiment <- count_sentiment |> 
  group_by(episode, season) |> 
  mutate(prop_words = n / sum(n))

episodes |> 
  group_by(season) |> 
  filter(episode == max(episode)) |> 
  select(season, 'max_ep' = episode) |> 
  right_join(count_sentiment, by = 'season') |> 
  mutate(prop_season = episode / max_ep) |> 
  mutate(sentiment = paste0(str_to_title(sentiment), '\n')) |> 
  ggplot(aes(prop_season, prop_words, color = season, group = season)) +
  theme_void() +
  theme(
    legend.position = 'none',
    strip.text = element_text(family = 'Poppins')
  ) +
  geom_line(size = 1, linejoin = 'round', lineend = 'round') +
  facet_wrap(vars(sentiment), ncol = 4) +
  coord_polar() +
  MetBrewer::scale_color_met_c('OKeeffe2')
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="576" />

Ah hah! Should have known. Honestly I'm pretty happy with this as the final
plot!
