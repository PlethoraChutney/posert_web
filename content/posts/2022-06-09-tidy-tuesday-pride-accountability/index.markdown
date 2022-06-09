---
title: Tidy Tuesday - Pride Accountability
author: Rich Posert
date: '2022-06-09'
slug: []
categories: ['Data Visualization']
tags: ['TidyTuesday']
cover:
  image: 'pride.png'
  alt: 'A donut chart showing the top five donors to transphobe Greg Abbott which are also Pride donors.'
  relative: true
---



Hopefully it does not shock anyone that companies who Donate To Pride Events
are also Donating To Bigots because it is All About the Bottom Line.


```r
library(tidyverse)
tuesdata <- tidytuesdayR::tt_load('2022-06-07')
```

```
## 
## 	Downloading file 1 of 8: `contribution_data_all_states.csv`
## 	Downloading file 2 of 8: `corp_by_politician.csv`
## 	Downloading file 3 of 8: `donors.csv`
## 	Downloading file 4 of 8: `fortune_agg.csv`
## 	Downloading file 5 of 8: `fortune_aggregates.csv`
## 	Downloading file 6 of 8: `pride_aggregates.csv`
## 	Downloading file 7 of 8: `pride_sponsors.csv`
## 	Downloading file 8 of 8: `static_list.csv`
```

Lots of tables in this one. You can check the
[documentation](https://github.com/rfordatascience/tidytuesday/tree/master/data/2022/2022-06-07)
for more info on what they all are. Right away, an idea that jumps out at me is
to look at which companies have donated the most to Greg Abbott, who
[rather famously hates trans kids](https://gov.texas.gov/uploads/files/press/O-MastersJaime202202221358.pdf)
(and honestly [kids in general](https://www.sacurrent.com/news/abbott-draws-swift-rebuke-after-calling-for-legislative-committees-to-study-uvalde-school-shooting-29030846)).


```r
tuesdata$corp_by_politician |> 
  group_by(Politician) |> 
  summarize(pol_sum = sum(`SUM of Amount`)) |> 
  arrange(-pol_sum)
```

```
## # A tibble: 103 × 2
##    Politician              pol_sum
##    <chr>                     <dbl>
##  1 Grand Total            4088534.
##  2 Greg Abbott            2219301.
##  3 Kay Ivey                302538.
##  4 Ken Paxton              192501.
##  5 Bill Lee                 99250 
##  6 Cameron Sexton           77600 
##  7 William Lamberth         48250 
##  8 Juan Fernandez-Barquin   42500 
##  9 Jeremy Faison            40700 
## 10 Ron Gant                 39350 
## # … with 93 more rows
```

Oh! Would you look at that! He has the most donations of any single politician
on this list! Over **half** of the total donations to all politicians went to
Abbott. WhY wOuLD tHe COmpAnIeS Do THaT???


```r
abbott <- tuesdata$contribution_data_all_states |> 
  filter(Politician == 'Greg Abbott')

glimpse(abbott)
```

```
## Rows: 2,253
## Columns: 13
## $ Company                                                                     <chr> …
## $ `Pride and Sponsor Match?`                                                  <chr> …
## $ `Pride?`                                                                    <lgl> …
## $ `HRC Business Pledge`                                                       <lgl> …
## $ `Donor Name`                                                                <chr> …
## $ Politician                                                                  <chr> …
## $ State                                                                       <chr> …
## $ Amount                                                                      <dbl> …
## $ Date                                                                        <dttm> …
## $ Citation                                                                    <lgl> …
## $ `Donor Type`                                                                <chr> …
## $ Comments                                                                    <lgl> …
## $ `ARCHIVE - Company Manually Determined (May Not Match Pride Sponsors List)` <lgl> …
```

OK, lots of data here. All these `NA` companies are just trusts or PACs or whatever.
Let's filter those out to focus on the companies who Care About You. Also, Companies
appear multiple times. Let's just look at lifetime donations. I'll also make sure everyone
in the list contributed to a pride campaign.


```r
abbott <- abbott |>
  filter(!is.na(Company) & `Pride?`) |> 
  group_by(Company) |> 
  summarize(sum = sum(Amount), beginning = min(Date), ending = max(Date))

abbott |> 
  arrange(-sum)
```

```
## # A tibble: 15 × 4
##    Company             sum beginning           ending             
##    <chr>             <dbl> <dttm>              <dttm>             
##  1 Toyota          580000  2019-06-28 00:00:00 2021-12-16 00:00:00
##  2 AT&T            180000  2019-12-20 00:00:00 2021-06-24 00:00:00
##  3 Comcast          40000  2019-10-02 00:00:00 2020-12-08 00:00:00
##  4 General Motors   35000  2019-12-20 00:00:00 2020-12-10 00:00:00
##  5 ConocoPhillips   25002. 2020-06-26 00:00:00 2021-10-19 00:00:00
##  6 Walmart          20000  2019-12-09 00:00:00 2021-11-01 00:00:00
##  7 Chevron          15000  2020-12-12 00:00:00 2022-02-22 00:00:00
##  8 Bank of America  10000  2019-12-11 00:00:00 2019-12-11 00:00:00
##  9 Enterprise       10000  2020-12-04 00:00:00 2020-12-04 00:00:00
## 10 JPMorgan Chase   10000  2020-09-18 00:00:00 2020-09-18 00:00:00
## 11 PNC              10000  2020-12-04 00:00:00 2020-12-04 00:00:00
## 12 State Farm       10000  2020-03-09 00:00:00 2020-03-09 00:00:00
## 13 Sysco             3000  2021-12-20 00:00:00 2021-12-20 00:00:00
## 14 Capital One       2500  2020-06-30 00:00:00 2020-06-30 00:00:00
## 15 FedEx             2500  2020-10-27 00:00:00 2020-10-27 00:00:00
```

Haha. Toyota is number one. Guarantee they released some LGBTQ ad this year.

<iframe src="//cdn.jwplayer.com/players/o8uGFllb-gXuUOJFF.html" width="640" height="360" frameborder="0" scrolling="auto"></iframe>

"Haha".

Anyway. What's the distribution of donations look like for these pieces of shit?


```r
abbott |> 
  ggplot(aes(x = sum)) +
  theme_minimal() +
  geom_histogram() +
  scale_x_continuous(labels = scales::label_dollar())
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

Wow. Toyota is a huge fucking outlier. If we only look at the other worms?


```r
abbott |> 
  filter(Company != 'Toyota') |> 
  ggplot(aes(x = sum)) +
  theme_minimal() +
  geom_histogram() +
  scale_x_continuous(labels = scales::label_dollar())
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Looks like the majority of the donations are less than $25k. Let's group those all
together.


```r
abbott <- abbott |> 
  mutate(Company = case_when(
    sum <= 25000 ~ 'Other',
    TRUE ~ Company
  )) |> 
  group_by(Company) |> 
  summarize(sum = sum(sum), beginning = min(beginning), ending = max(ending))

abbott |> 
  mutate(is_other = Company == 'Other') |> 
  mutate(Company = fct_reorder(Company, sum)) |> 
  ggplot(aes(y = Company, x = sum, fill = is_other)) +
  theme_minimal() +
  theme(
    legend.position = 'none',
    plot.margin = margin(0, 20, 0, 0) # prevent x axis label cutoff
  ) +
  scale_x_continuous(labels = scales::label_dollar()) +
  geom_col()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

This is very against my nature, but I kind of want to make a donut chart and put
Abbott's stupid face right in the middle. I read a (paper recently)[https://kosara.net/papers/2016/Skau-EuroVis-2016.pdf]
showing that people are not as bad at reading pie and donut charts as I had recently
thought! Fun.

You can make a donut/pie chart with `geom_rect()` and polar coordinates, but that's
unintuitive and prone to breaking. Let's just use ggforce.


```r
library(ggforce)
abbott |> 
  filter(Company != 'Other') |> 
  mutate(Company = fct_reorder(Company, sum)) |> 
  arrange(Company) |> 
  ggplot(aes(fill = Company)) +
  theme_no_axes() +
  theme(
    panel.border = element_blank()
  ) +
  coord_fixed() +
  geom_arc_bar(
    aes(x0 = 0, y0 = 0, r0 = 0.75, r = 1, amount = sum),
    stat = 'pie', # converts values to start/end positions for us
    size = 1
    ) +
  scale_fill_manual(
    values = c(
      '#E0837C',
      '#E0AB7B',
      '#E0DF8E',
      '#98E0C6',
      '#9DB2DF'
    )
  ) +
  labs(
    title = 'Top 5 Donors to Greg Abbott who are also Pride sponsors',
    caption = 'Data provided by Data for Progress: https://www.dataforprogress.org/accountable-allies'
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />

I'll do some final customization in Affinity Designer. Here's the final product.

![Greg Abbott's pride donors](pride.png)
