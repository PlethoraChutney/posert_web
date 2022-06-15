---
title: Tidy Tuesday - Drought
author: Rich Posert
date: '2022-06-15'
slug: []
categories: ['Data Visualization']
tags: ['TidyTuesday']
cover:
  image: 'header-plot.png'
  alt: 'An area chart showing the drought state of counties in CA over time. There are pink markers of increasing intensity to show when El Nino events occur'
  relative: true
---



As a completely unrelated thought to today's topic, when I watched
[First Reformed](https://en.wikipedia.org/wiki/First_Reformed) I identified so
closely with the protagonist that many of my friends became very worried for me.

Let's load up the data!


```r
library(tidyverse)
# tuesdata <- tidytuesdayR::tt_load('2022-06-14')
# drought <- tuesdata$drought
# fips <- tuesdata$`drought-fips`
load('.Rdata')

glimpse(drought)
```

```
## Rows: 73,344
## Columns: 14
## $ `0`   <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 72.8, 47.4, 53.5, 19.4, …
## $ DATE  <chr> "d_18950101", "d_18950201", "d_18950301", "d_18950401", "d_18950…
## $ D0    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 19.9, 51.6, 46.2, 80.7, …
## $ D1    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.7, 29.7, 24.4, 61.2, 5…
## $ D2    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.6, 0.0, 13.2, 5.5…
## $ D3    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 3.6, 0.0,…
## $ D4    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,…
## $ `-9`  <dbl> 100, 100, 100, 100, 100, 100, 100, 100, 0, 0, 0, 0, 0, 0, 0, 0, …
## $ W0    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 7.3, 1.0, 0.2, 0.0, 0.0,…
## $ W1    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.1, 0.0, 0.0, 0.0, 0.0,…
## $ W2    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ W3    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ W4    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ state <chr> "alabama", "alabama", "alabama", "alabama", "alabama", "alabama"…
```

```r
glimpse(fips)
```

```
## Rows: 3,771,791
## Columns: 4
## $ State <chr> "AK", "AK", "AK", "AK", "AK", "AK", "AK", "AK", "AK", "AK", "AK"…
## $ FIPS  <chr> "02013", "02013", "02013", "02013", "02013", "02013", "02013", "…
## $ DSCI  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date  <date> 2000-01-04, 2000-01-11, 2000-01-18, 2000-01-25, 2000-02-01, 200…
```

Huh. Not the best documentation for this dataset. I have to assume that `-9` is
"no data", since in January of 1895 the NOAA source has no data for 100% of the country.
Not sure what `0` is...


In any case, I've checked the first few months of Alabama's data against the NOAA
source and the drought levels, which is what we're interested in anyway, match up.

Additionally, any county which is, for instance, 'Exceptional Wet' will also be
counted in all the other wetness levels. For these trend lines, I'd like each
county to only be counted once, in its wettest level. So I need to subtract
W4 from W3, then that difference from W2, and that from W1, and finally that
from W0. Same for the D's.


```r
drought <- drought |> 
  # let's fix this psycho date format while we're at it
  mutate(DATE = str_replace(DATE, 'd_', '')) |> 
  mutate(DATE = lubridate::as_date(DATE))

single_count <- drought |> 
  mutate(W3 = W3 - W4, D3 = D4 - D3) |> 
  mutate(W2 = W2 - W3, D2 = D2 - D3) |> 
  mutate(W1 = W1 - W2, D1 = D1 - D2) |> 
  mutate(W0 = W0 - W1, D0 = D0 - D1)
```


So now let's make one of my favorite plots:

Spot!

Those!

Trends!!


```r
# test a plot which we know should have a good bit of drought
single_count |> 
  filter(`-9` != 100 & state == 'california') |> 
  pivot_longer(cols = D0:D4 | W0:W4, names_to = 'Wetness', values_to = 'Percent') |> 
  ggplot(aes(x = DATE, y = Percent)) +
  theme_minimal() +
  geom_area(aes(fill = Wetness)) +
  facet_wrap(vars('state'), nrow = 10, ncol = 5)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

Ooh...not good. Looks like we've got some negatives, and we also need to count
the counties which are not in any abnormal wetness. Now that I know the percents are,
in some cases, quadruple-counted, I bet that's what the `0` column is.

Let's figure out where our negative values come from...


```r
single_count |> 
  filter(`-9` != 100 & state == 'california') |> 
  pivot_longer(cols = D0:D4 | W0:W4, names_to = 'Wetness', values_to = 'Percent') |> 
  filter(Percent < 0) |> 
  pull(Wetness) |> 
  unique()
```

```
## [1] "D3" "D1"
```

OK, all the negative values come from D3 and D1. And looking at my original code,
I gave the wrong difference for D3. That makes D3 negative, D2 bigger than it should
be, which sometimes makes D1 negative as well. Easy fix!


```r
single_count <- drought |> 
  mutate(W3 = W3 - W4, D3 = D3 - D4) |> 
  mutate(W2 = W2 - W3, D2 = D2 - D3) |> 
  mutate(W1 = W1 - W2, D1 = D1 - D2) |> 
  mutate(W0 = W0 - W1, D0 = D0 - D1)

single_count |> 
  filter(`-9` != 100 & state == 'california') |> 
  pivot_longer(cols = `0` | D0:D4 | W0:W4, names_to = 'Wetness', values_to = 'Percent') |> 
  ggplot(aes(x = DATE, y = Percent)) +
  theme_minimal() +
  geom_area(aes(fill = Wetness)) +
  facet_wrap(vars('state'), nrow = 10, ncol = 5)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

Alright at least we got rid of the negative values, and we've filled in to 100%
with the `0` column as expected. But why are we still getting these damn peaks?
Let's fill in to a more readable color scheme and see if we can figure out what's going on.


```r
single_count |> 
  filter(`-9` != 100 & state == 'california') |> 
  pivot_longer(cols = `0` | D0:D4 | W0:W4, names_to = 'Wetness', values_to = 'Percent') |> 
  mutate(Wetness = fct_relevel(Wetness, 'D4', 'D3', 'D2', 'D1', 'D0', '0', 'W0', 'W1', 'W2', 'W3', 'W4')) |> 
  ggplot(aes(x = DATE, y = Percent)) +
  theme_minimal() +
  scale_fill_manual(
    values = c(
      '#67001f',
      '#b2182b',
      '#d6604d',
      '#f4a582',
      '#fddbc7',
      '#f7f7f7',
      '#d1e5f0',
      '#92c5de',
      '#4393c3',
      '#2166ac',
      '#053061'
    )
  ) +
  geom_area(aes(fill = Wetness)) +
  facet_wrap(vars('state'), nrow = 10, ncol = 5) +
  coord_cartesian(xlim = c(lubridate::as_date('1900-01-01'), lubridate::as_date('1910-01-01')))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Hmm...hard to tell what's wrong. Let's take a look at the data.


```r
troubleshoot <- single_count |> 
  filter(state == 'california' & DATE < lubridate::as_date('1905-01-01')) |>
  rowwise() |> # this is inefficient but we're just troubleshooting here
  mutate(total = sum(c_across(c(`0`, D0:D4, W0:W4))))
  
  
glimpse(troubleshoot)
```

```
## Rows: 120
## Columns: 15
## Rowwise: 
## $ `0`   <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 35.5, 58.5, 60.4, 44.5, …
## $ DATE  <date> 1895-01-01, 1895-02-01, 1895-03-01, 1895-04-01, 1895-05-01, 189…
## $ D0    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 2.4, 25.2, 26.7, 36.1, 1…
## $ D1    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.1, 15.3, 12.9, 19.2, 9…
## $ D2    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 3.6, 2.4, 3.1, 1.9,…
## $ D3    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 2.2, 1.3, 2.1, 0.4,…
## $ D4    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 2.1, 0.0, 0.0, 0.0,…
## $ `-9`  <dbl> 100, 100, 100, 100, 100, 100, 100, 100, 0, 0, 0, 0, 0, 0, 0, 0, …
## $ W0    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 36.2, 1.0, 0.1, 0.1, 26.…
## $ W1    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 25.8, 0.0, 0.0, 0.0, 16.…
## $ W2    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 3.7, 0.0, 0.0, 0.0, 0.0,…
## $ W3    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.8, 0.0, 0.0, 0.0, 0.0,…
## $ W4    <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,…
## $ state <chr> "california", "california", "california", "california", "califor…
## $ total <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 104.5, 107.9, 103.8, 105…
```

Hmm...everything is off by at least a few percent...

Ah hah!!! Let's look at this one row:


```r
troubleshoot |> 
  filter(total == 186.1) |>
  glimpse()
```

```
## Rows: 1
## Columns: 15
## Rowwise: 
## $ `0`   <dbl> 32.8
## $ DATE  <date> 1904-01-01
## $ D0    <dbl> 43.9
## $ D1    <dbl> 22.7
## $ D2    <dbl> 38.2
## $ D3    <dbl> 15.3
## $ D4    <dbl> 32.6
## $ `-9`  <dbl> 0
## $ W0    <dbl> 0.5
## $ W1    <dbl> 0.1
## $ W2    <dbl> 0
## $ W3    <dbl> 0
## $ W4    <dbl> 0
## $ state <chr> "california"
## $ total <dbl> 186.1
```

At this point, the state is very dry. And we're only subtracting the level
just above from the total. When the state is not very dry or very wet, this is very
close to correct, since we expect the level more toward average to be significantly
more populated than the more extreme level. However, when the state is very dry,
only subtracting the next level out from mean results in a big error! We have to subtract
everything outward from the level we're considering:


```r
single_count <- drought |> 
  mutate(W3 = W3 - W4, D3 = D3 - D4) |> 
  mutate(W2 = W2 - (W3 + W4), D2 = D2 - (D3 + D4)) |> 
  mutate(W1 = W1 - (W2 + W3 + W4), D1 = D1 - (D2 + D3 + D4)) |> 
  mutate(W0 = W0 - (W1 + W2 + W3 + W4), D0 = D0 - (D1 + D2 + D3 + D4))

single_count |> 
  filter(`-9` != 100 & state == 'california') |> 
  pivot_longer(cols = `0` | D0:D4 | W0:W4, names_to = 'Wetness', values_to = 'Percent') |> 
  mutate(Wetness = fct_relevel(Wetness, 'D4', 'D3', 'D2', 'D1', 'D0', '0', 'W0', 'W1', 'W2', 'W3', 'W4')) |> 
  ggplot(aes(x = DATE, y = Percent)) +
  theme_minimal() +
  scale_fill_manual(
    values = c(
      '#67001f',
      '#b2182b',
      '#d6604d',
      '#f4a582',
      '#fddbc7',
      '#f7f7f7',
      '#d1e5f0',
      '#92c5de',
      '#4393c3',
      '#2166ac',
      '#053061'
    )
  ) +
  geom_area(aes(fill = Wetness)) +
  facet_wrap(vars('state'), nrow = 10, ncol = 5) +
  coord_cartesian(xlim = c(lubridate::as_date('1900-01-01'), lubridate::as_date('1910-01-01')))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

Alright!! I think this is a pretty plot 😊. I bet we can take a look at seasonality
here too:


```r
last_plot() + 
  scale_x_date(date_breaks = '1 year', date_minor_breaks = '3 months', date_labels = '%Y')
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="672" />

Ah, that's just nice to look at. Anyway, I bet it gets a lot less readable when
zoomed out:


```r
single_count |> 
  filter(`-9` != 100 & state == 'california') |> 
  pivot_longer(cols = `0` | D0:D4 | W0:W4, names_to = 'Wetness', values_to = 'Percent') |> 
  mutate(Wetness = fct_relevel(Wetness, 'D4', 'D3', 'D2', 'D1', 'D0', '0', 'W0', 'W1', 'W2', 'W3', 'W4')) |> 
  ggplot(aes(x = DATE, y = Percent)) +
  theme_minimal() +
  scale_fill_manual(
    values = c(
      '#67001f',
      '#b2182b',
      '#d6604d',
      '#f4a582',
      '#fddbc7',
      '#f7f7f7',
      '#d1e5f0',
      '#92c5de',
      '#4393c3',
      '#2166ac',
      '#053061'
    )
  ) +
  geom_area(aes(fill = Wetness))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="672" />
Well, it does, but you can actually still see the seasonality really nicely.
It reminds me now of [these "film spectrum"](https://www.artpal.com/filmspectrum)
visualizations. You can see spikes of red coming down every summer and spikes of
blue pushing back. But more and more red over time...probably fine!

I also like that you can see regions (the 80s?) where there's a lot more blue than
the surrounding area. I wonder...what if we plot years with moderate-to-strong
El Niño events?


```r
# from https://ggweather.com/enso/oni.htm
el_nino <- tibble(
  'start' = c(1951, 1963, 1968, 1986, 1994, 2002, 2009, 1957, 1965, 1972, 1987, 1991,  1982, 1997, 2015),
  'end' = c(1952, 1964, 1969, 1987, 1995, 2003, 2010, 1958, 1966, 1973, 1988, 1992, 1983, 1998, 2016),
  'intensity' = c(rep('#e7e1ef', 7), rep('#c994c7', 5), rep('#dd1c77', 3))
) |> 
  mutate(
    start = lubridate::as_date(paste0(as.character(start), '-01-01'), format = '%Y-%m-%d'),
    end = lubridate::as_date(paste0(as.character(end), '-01-01'), format = '%Y-%m-%d')
  )

last_plot() +
  annotate('segment', x = el_nino$start, xend = el_nino$end, y = 105, yend = 105, size = 3, color = el_nino$intensity) +
  annotate('segment', x = el_nino$start, xend = el_nino$end, y = -5, yend = -5, size = 3, color = el_nino$intensity) +
  coord_cartesian(xlim = c(lubridate::as_date('1950-01-01'), lubridate::as_date('2018-01-01')))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="672" />

We do seem to be able to relate drought in California to El Niño events! Remember
we're not looking at monthly rainfall here, but cumulative rainfall over 9 months
relative to the average, so it makes sense to me that it lags behind in some cases.

I'm actually pretty happy with this as a final chart. Let's just update a few stylistic
things.


```r
wetness_converter <- function(wetness) {
  if (wetness == '0') {
    return ('Normal')
  }
  
  severity = as.numeric(str_extract(wetness, '[0-4]')) + 1
  severity = c('Abnormally', 'Moderate', 'Severe', 'Extreme', 'Exceptional')[severity]
  
  type = if_else(grepl('D', wetness), 'Dry', 'Wet')
  
  return(paste(severity, type, sep = '\n'))
}

# fonts in R are such a nightmare
library(showtext)
font_add('Poppins', regular = 'C:\\Users\\rich\\AppData\\Local\\Microsoft\\Windows\\Fonts\\Poppins-Regular.ttf')
showtext_auto()

single_count |> 
  filter(
    `-9` != 100, 
    state == 'california',
    DATE > lubridate::as_date('1944-01-01'),
    DATE < lubridate::as_date('2018-01-01')
  ) |> 
  pivot_longer(cols = `0` | D0:D4 | W0:W4, names_to = 'Wetness', values_to = 'Percent') |> 
  mutate(Wetness = purrr::pmap_chr(list(Wetness), wetness_converter)) |>
  mutate(Wetness = fct_relevel(
    Wetness,
    'Exceptional\nDry',
    'Extreme\nDry',
    'Severe\nDry',
    'Moderate\nDry',
    'Abnormally\nDry',
    'Normal',
    'Abnormally\nWet',
    'Moderate\nWet',
    'Severe\nWet',
    'Extreme\nWet',
    'Exceptional\nWet'
  )) |> 
  ggplot(aes(x = DATE, y = Percent)) +
  theme_minimal() +
  theme(
    panel.grid = element_blank(),
    axis.ticks.x = element_line(color = 'black'),
    text = element_text(family = 'Poppins', size = 20, lineheight = 0.4)
  ) +
  scale_x_date(
    date_breaks = '5 years',
    date_labels = '%Y'
  ) +
  scale_fill_manual(
    values = c(
      '#67001f',
      '#b2182b',
      '#d6604d',
      '#f4a582',
      '#fddbc7',
      '#f7f7f7',
      '#d1e5f0',
      '#92c5de',
      '#4393c3',
      '#2166ac',
      '#053061'
    )
  ) +
  geom_area(aes(fill = Wetness)) +
  annotate(
    'segment',
    x = el_nino$start,
    xend = el_nino$end,
    y = 105,
    yend = 105,
    size = 3,
    color = el_nino$intensity
  ) +
  annotate(
    'segment',
    x = el_nino$start,
    xend = el_nino$end,
    y = -5,
    yend = -5,
    size = 3,
    color = el_nino$intensity
  ) +
  labs(
    y = 'Percent of Counties',
    x = 'Date',
    fill = 'Drought Level',
    title = 'Drought and El Niño in California',
    subtitle = 'Darker El Niño marker is a more intense event',
    caption = 'Data from the National Integrated Drought Information System'
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="672" />

Please take a look at the [full size](ca-drought_full-size.png) image too! I think
these turned out really nicely. I had a lot of fun making the plots, and I always love
publicly documenting my troubleshooting process.

See you next week!
