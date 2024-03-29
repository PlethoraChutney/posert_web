---
title: Flux assay
author: Rich Posert
date: '2022-05-14'
slug: []
categories: ['Data Visualization']
tags: []
cover:
  image: 'final-plot.png'
  alt: 'A plot of a sodium flux assay which, unfortunately, did not work.'
  relative: true
---



I'm developing a new flux assay, which means I have an excuse
to process and display the data coming off the plate reader. I wrote a python
script to convert the format from the plate reader into a more manageable,
tidy version. That's not particularly interesting, mostly just list comprehensions.
If you like, you can find more info on that part of the process [here](https://github.com/PlethoraChutney/flux_processor).

Let's take a look at the data my python script spits out:

```r
library(tidyverse)

raw_data <- read_csv('processed_data.csv', col_types = 'dffffd')

glimpse(raw_data)
```

```
## Rows: 4,788
## Columns: 6
## $ Time         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …
## $ Plate        <fct> Equilibration, Equilibration, Equilibration, Equilibratio…
## $ Row          <fct> A, A, A, B, B, B, C, C, C, D, D, D, E, E, E, F, F, F, G, …
## $ Column       <fct> 4, 5, 6, 4, 5, 6, 4, 5, 6, 4, 5, 6, 4, 5, 6, 4, 5, 6, 4, …
## $ Condition    <fct> No-ENaC_POPE-POPG, No-ENaC_POPE-POPG, No-ENaC_POPE-POPG, …
## $ Fluorescence <dbl> 38443, 39156, 38974, 38166, 38833, 38789, 39825, 40253, 4…
```

For this flux assay, you prepare proteoliposomes with Na+ inside and K+ outside.
Importantly, the proteoliposomes have ACMA, a fluorescent dye quenched by a proton
gradient across the bilayer. When you add the proteoliposomes to the external buffer
nothing will happen, unless you have a functional ion channel. In this case, the
channel will allow Na+ through the membrane but will *not* allow K+ back across,
creating an electrochemical gradient.

After some equilibration reads, you add CCCP, a proton ionophore. If you have no
functional channels, again, nothing happens here. But if you've already established
an electrochemical gradient, then CCCP will let protons flow down the charge gradient,
establishing a pH gradient and quenching ACMA fluorescence. Finally, a Na+ ionophore
is added as a positive control, effectively sending each vesicle to their minimal
fluorescence level by the same mechanism as a functional ion channel.

This processed dataframe has five columns:
 * Time, in seconds, at which this read was taken.
 * Which "plate" the read is from (i.e., Equilibration, CCCP, or Na+ Iono)
 * Which row the well read is from
 * Which column the well read is from
 * The condition for that well (for averaging)
 * The absolute fluorescence, as measured by the plate reader
 
My python script has already added some gaps to the time column for when the plate
is removed to add reagents and mix.

The first step to plotting this data is to put everything into relative fluroescence.
We will normalize flourescence from 0 to 1, with 0 being a true read of 0 fluorescence
and 1 being the final fluorescence value during the equilibration stage.


```r
data <- raw_data %>% 
  filter(Plate == 'Equilibration') %>% 
  filter(Time == max(Time)) %>% 
  select(Row, Column, 'Equil_Flr' = Fluorescence) %>% 
  right_join(raw_data, by = c('Row', 'Column')) %>% 
  mutate(Fluorescence = Fluorescence / Equil_Flr) %>% 
  separate(Condition, into = c('ENaC', 'Lipids'), sep = '_', fill = 'right') %>% 
  mutate(ENaC = str_replace(ENaC, '-', ' '))

glimpse(data)
```

```
## Rows: 4,788
## Columns: 8
## $ Row          <fct> A, A, A, A, A, A, A, A, A, A, A, A, A, A, A, A, A, A, A, …
## $ Column       <fct> 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, …
## $ Equil_Flr    <dbl> 36493, 36493, 36493, 36493, 36493, 36493, 36493, 36493, 3…
## $ Time         <dbl> 0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70,…
## $ Plate        <fct> Equilibration, Equilibration, Equilibration, Equilibratio…
## $ ENaC         <chr> "No ENaC", "No ENaC", "No ENaC", "No ENaC", "No ENaC", "N…
## $ Lipids       <chr> "POPE-POPG", "POPE-POPG", "POPE-POPG", "POPE-POPG", "POPE…
## $ Fluorescence <dbl> 1.0534349, 1.0529965, 1.0544214, 1.0608884, 1.0576275, 1.…
```

The trick I like here is using `filter` to pull the value we want from each
well ('Equilibration' plate and max Time), then `right_join`ing it back into
the original data frame using the row and column. It saves the step of using
summarize and making a new object.

I also use `separate` to split my condition column into the concentration of
ENaC and the lipid mix, since those were my variables for this assay.


Next, we compute the mean for each condition. B5 had an outlier in it that I
remove here.


```r
processed <- data %>% 
  filter(!(Row == 'B' & Column == '5')) %>% 
  group_by(ENaC, Lipids, Time, Plate) %>% 
  summarize(Mean_Fluorescence = mean(Fluorescence), .groups = 'drop_last')

glimpse(processed)
```

```
## Rows: 1,596
## Columns: 5
## Groups: ENaC, Lipids, Time [1,596]
## $ ENaC              <chr> "1:200", "1:200", "1:200", "1:200", "1:200", "1:200"…
## $ Lipids            <chr> "POPE-POPG", "POPE-POPG", "POPE-POPG", "POPE-POPG", …
## $ Time              <dbl> 0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65…
## $ Plate             <fct> Equilibration, Equilibration, Equilibration, Equilib…
## $ Mean_Fluorescence <dbl> 1.0543934, 1.0428864, 1.0366178, 1.0236230, 1.023274…
```

At this point, we can make our first diagnostic plot:


```r
processed %>% 
  ggplot(aes(x = Time, y = Mean_Fluorescence, color = ENaC)) +
  theme_minimal() +
  geom_line() +
  facet_grid(cols = vars(Lipids))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

OK, there's some obvious annoyances here. Most obviously, the buffer control is
in its own facet instead of paired with each experimental plot. Second, we'd like
to be able to highlight where each plate/reagent is added. And finally, the ENaC
concentrations are not ordered by ENaC amount. Let's fix those before
moving on to aesthetics.


```r
plates <- data %>% 
  group_by(Plate) %>% 
  summarize(start = min(Time), end = max(Time))

buffer <- data %>% 
  filter(ENaC == 'Buffer') %>% 
  mutate(Lipids = 'POPE-POPG')

buffer <- buffer %>% 
  mutate(Lipids = 'POPE-POPG-PIP2') %>% 
  bind_rows(buffer)

data <- data %>% 
  filter(ENaC != 'Buffer') %>% 
  bind_rows(buffer) %>% 
  mutate(ENaC = fct_relevel(ENaC, 'Buffer', 'No ENaC', '1:600', '1:200'))

processed <- data %>% 
  filter(!(Row == 'B' & Column == '5')) %>% 
  group_by(ENaC, Lipids, Time, Plate) %>% 
  summarize(Mean_Fluorescence = mean(Fluorescence), .groups = 'drop_last')

processed %>% 
  ggplot(aes(x = Time, y = Mean_Fluorescence, color = ENaC)) +
  theme_minimal() +
  geom_rect(
    inherit.aes = FALSE,
    data = plates,
    aes(xmin = start, xmax = end, fill = Plate),
    ymin = -Inf,
    ymax = Inf,
    alpha = 0.05
  ) +
  geom_line() +
  facet_grid(cols = vars(Lipids))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />

That's much better. But this plot makes it look like we know what's happening when
the plates are out and we're adding reagents, which of course we don't. We can
make use of the `group` aesthetic along with `interaction()` to separate the lines
by both ENaC concentration *and* plate reagent, like so:

```r
processed %>% 
  ggplot(aes(
    x = Time,
    y = Mean_Fluorescence,
    color = ENaC,
    group = interaction(ENaC, Plate)
    )) +
  theme_minimal() +
  geom_rect(
    inherit.aes = FALSE,
    data = plates,
    aes(xmin = start, xmax = end, fill = Plate),
    ymin = -Inf,
    ymax = Inf,
    alpha = 0.05
  ) +
  geom_line() +
  facet_grid(cols = vars(Lipids))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

Still not happy with that. I'd like some dotted lines to guide the eye between
the plates.


```r
joiners <- processed %>% 
  group_by(Plate, ENaC, Lipids) %>% 
  filter(Time == max(Time) | Time == min(Time)) %>% 
  group_by(ENaC, Lipids) %>% 
  mutate(Plate = lead(Plate)) %>% 
  drop_na()

glimpse(joiners)
```

```
## Rows: 40
## Columns: 5
## Groups: ENaC, Lipids [8]
## $ ENaC              <fct> Buffer, Buffer, Buffer, Buffer, Buffer, Buffer, Buff…
## $ Lipids            <chr> "POPE-POPG", "POPE-POPG", "POPE-POPG", "POPE-POPG", …
## $ Time              <dbl> 0, 95, 276, 671, 877, 0, 95, 276, 671, 877, 0, 95, 2…
## $ Plate             <fct> Equilibration, CCCP, CCCP, Na+ Iono, Na+ Iono, Equil…
## $ Mean_Fluorescence <dbl> 1.0300249, 1.0000000, 0.8747263, 0.8663702, 0.890211…
```

To make our joiner lines, we first get the first and last read from each plate.
Then, we lag the plate name. Doing so associates the last read from one plate
and the first read from the next, thus giving two points per gap per line. Adding
them to the plot is straightforward:


```r
processed %>% 
  ggplot(
    aes(
      x = Time,
      y = Mean_Fluorescence,
      color = ENaC,
      group = interaction(ENaC, Plate)
      )
    ) +
  theme_minimal() +
  geom_rect(
    inherit.aes = FALSE,
    data = plates,
    aes(xmin = start, xmax = end, fill = Plate),
    ymin = -Inf,
    ymax = Inf,
    alpha = 0.05
  ) +
  geom_line(
    data = joiners,
    aes(
      x = Time,
      y = Mean_Fluorescence,
      group = interaction(ENaC, Plate),
      color = ENaC
      ),
    inherit.aes = FALSE,
    linetype = 'dotted',
    lineend = 'round'
  ) +
  geom_line() +
  facet_grid(cols = vars(Lipids))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

I like the structure of the plot. Now it's just labeling and coloring by increasing
ENaC. I also think the lines look better if they're a bit thicker. We should
also probably make sure the limits include 0.0 to make clear the extent of fluroescence
loss during the ionophore addition --- non-functional proteoliposomes might still
look functional with the current scaling.


```r
processed %>% 
  ggplot(
    aes(
      x = Time,
      y = Mean_Fluorescence,
      color = ENaC,
      group = interaction(ENaC, Plate)
      )
    ) +
  theme_minimal() +
  geom_rect(
    inherit.aes = FALSE,
    data = plates,
    aes(xmin = start, xmax = end, fill = Plate),
    ymin = -Inf,
    ymax = Inf,
    alpha = 0.05
  ) +
  geom_line(
    data = joiners,
    aes(
      x = Time,
      y = Mean_Fluorescence,
      group = interaction(ENaC, Plate),
      color = ENaC
      ),
    size = 2,
    inherit.aes = FALSE,
    linetype = 'dotted',
    lineend = 'round'
  ) +
  geom_line(size = 2) +
  facet_grid(cols = vars(Lipids)) +
  scale_color_manual(values = c(
    '#DDDDDD',
    '#fee6ce',
    '#fdae6b',
    '#e6550d'
  )) +
  scale_fill_manual(values = c(
    "#0077bb",
    "#009988",
    "#cc3311",
    "#ee3377"
  )) +
  scale_y_continuous(
    breaks = seq(0, 1, 0.25)
  ) +
  labs(
    y = 'Relative Fluorescence',
    x = 'Time (s)',
    color = 'Condition',
    fill = 'Reagent added'
  ) +
  expand_limits(y = c(0, 1)) +
  labs(
    color = 'ENaC',
    subtitle = '1:200 no-PIP outlier removed'
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />

Clearly, in this case, the assay didn't work, but I'm happy with the plots!
