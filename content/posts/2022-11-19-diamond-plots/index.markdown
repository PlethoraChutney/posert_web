---
title: Diamond Plots
author: Rich Posert
date: '2022-11-19'
slug: []
categories: ['Data Visualization']
tags: []
summary: "Don't see causation where it ain't."
---

My dad showed me [this paper](https://arxiv.org/abs/1809.09328) by
Carl T. Bergstrom and Jevin D. West this morning. In it, the authors
rotate scatter plots by 45° to avoid suggesting causation when plotting
two correlated variables.

The plots are produced in Mathematica, but I thought I'd go ahead and try
recreating them in ggplot. This was my first time diving into `grid` and I
gotta say, it's given me a taste for learning more about the guts of my
favorite plotting library. Check it out!


```r
library(tidyverse)

data <- tibble(
  x = rnorm(250, 0, 4),
  y = 3.2 * x - (0.05 * x^2) + rnorm(250, 1, 1)
)

# calculate the ratio required to make the plot square
disp_ratio <- (max(data$x) - min(data$x)) / (max(data$y) - min(data$y))

p <- data |> 
  ggplot(aes(x, y)) +
  theme_bw() +
  geom_vline(xintercept = 0) +
  geom_hline(yintercept = 0) +
  geom_point(fill = '#2182FA', shape = 21, color = 'black') +
  theme(
    axis.title.y = element_text(angle = 0, vjust = 0.5),
    axis.title.x = element_text(hjust = 0.5)
  ) +
  coord_fixed(ratio = disp_ratio)
p
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />

Typical stuff, but I rotated the `y` label to match what we'll be doing next:


```r
library(grid)

# clear out the plot from the viewport
grid.newpage()

# rotate the entire viewport counterclockwise
pushViewport(viewport(angle = 45))
grid.draw(
  ggplotGrob(
    # draw the plot, but rotate the labels
    p +
      theme(
        text = element_text(angle = -45),
        axis.title.y = element_text(angle = -45, vjust = 0.5),
        axis.title.x = element_text(angle = -45, vjust = 0.5),
        # this is the hacky part. If we don't add a margin, our
        # viewport is too small and will cut off the edges of the
        # plot. There's probably a better way...
        plot.margin = unit(c(2,2,2,2), 'cm')
      )
  )
)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Fun!

