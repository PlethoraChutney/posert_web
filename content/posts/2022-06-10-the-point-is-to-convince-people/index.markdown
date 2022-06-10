---
title: The point is to convince people
author: Rich Posert
date: '2022-06-10'
slug: []
categories: ['Opinion']
tags: []
cover:
  image: 'stop-and-frisk.jpg'
  alt: 'A chart of stop and frisk events and actual guns found over time.'
  relative: true
---



I [joked about this](https://twitter.com/PosertInLab/status/1532780189263941632)
but the other day a [paper](https://arxiv.org/abs/2204.09548) came out which
purports to help us "better understand the landscape of misleading visualizations".
This is an admirable goal --- I have definitely unintentionally made some misleading
visualizations! Let's take a look at their [gallery of misleading charts](http://leoyuholo.com/bad-vis-browser/).

Wait...

![A chart of stop and frisk events and actual guns found over time](stop-and-frisk.jpg)

The authors classify this chart as being misleading because it plots data of
different magnitudes on the same axis. According to the authors,

> The series with a larger magnitude will dominate the one with a
> smaller magnitude, and the reader can hardly see its changes. While
> plotting the two series with two different axes, the chart becomes
> misleading if the reader is unaware.

Now, to be fair to them, they don't say that plotting different magnitudes is *always*
misleading. Nor do they claim that there's a universal solution. Choice of axes is hard.
But they did include this graph specifically in their gallery (it's the fifth of only
six examples they give for this "error"!). And I don't think it's misleading at all.

The point of data visualization is not to disinterestedly present data without bias.
Use a table for that. Even then, the way you collected the data and even the questions
you're asking in the first place are biased. We make charts to emphasize certain
facets of the data and to lead the audiences to certain conclusions. This is *especially*
true in talks, but that's another post.

So, what do the authors propose we do? Helpfully, they do not recommend a solution.
They point out that there's not really a neat way to not be biased. Something like
this:

```r
stop_and_frisk <- tibble(
  year = c(2002:2011),
  stops = c(97296, 160851, 313523, 398191, 506491, 472096, 540302, 581168, 601285, 685724),
  guns = c(NA, 604, 688, 631, 670, 637, 798, 726, 736, 780)
) |> 
  pivot_longer(c('stops', 'guns'), names_to = 'Type', values_to = 'Value')

stop_and_frisk |> 
  # you have to be pretty hacky to get two axes in ggplot
  mutate(Value = case_when(
    Type == 'guns' ~ Value * 1000,
    TRUE ~ Value
  )) |> 
  ggplot(aes(x = year, y = Value, color = Type)) +
  theme_minimal() +
  theme(
    legend.position = 'top'
  ) +
  geom_line() +
  geom_point(size = 2) +
  ggokabeito::scale_color_okabe_ito() +
  labs(
    title = 'Bad chart that you would only use if you love the fucking cops',
    y = 'Stops',
    x = 'Year',
    color = 'Count'
  ) +
  scale_y_continuous(
    # Here's where we undo the mutation on the
    # gun counts from above
    sec.axis = sec_axis(
      'Guns Found',
      trans = ~ . / 1000
    )
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

will lead the reader to a causal conclusion --- when we increased the stops,
we found more guns. Which, I mean, no. Why are you making this visualization?
Presumably, hopefully, it's to drive home the fact that
[cops don't prevent crime](https://www.portlandmercury.com/blogtown/2021/11/09/36865079/portlands-crime-rate-isnt-impacted-by-size-of-police-force-data-finds),
they have
[no duty to respond to crime](https://www.nytimes.com/2005/06/28/politics/justices-rule-police-do-not-have-a-constitutional-duty-to-protect.html),
they don't
[solve crime](https://theconversation.com/police-solve-just-2-of-all-major-crimes-143878)
and when they claim to, they're just
[throwing random people in jail](https://innocenceproject.org/dna-exonerations-in-the-united-states/).
Well, I shouldn't even say random --- they find the 
[nearest non-white](https://www.sentencingproject.org/publications/color-of-justice-racial-and-ethnic-disparity-in-state-prisons/)
person and lock them up.

Making a chart to emphasize the fact that when cops perform over 600,000 stops,
almost none of which turn up anything. You're leading the reader to wonder ---
what's the point of this, then? Why are we creating six hundred thousand interactions
with racist people holding guns if we're not getting anything out of it? What would
we have to get to make this worth it? Your aim, hopefully, is not to get them to mindlessly compare stop numbers and
guns found.

Do I think this is a perfect chart? No. It's lacking context, for one. Worth thinking
about the geography of these stops. How many are done in the Bronx vs. Manhattan?
What happened to the number of stops in Brooklyn when white people started moving
there in droves?

And, maybe more importantly, you could not show me a number of
guns that would justify the police. So I am bringing my own bias to this, which
shouldn't be surprising to you. All of this is just to say, the point of doing
all of this --- collecting data, visualizing it, making talks and slides, designing
experiments or studies --- is to convince someone to agree with you. There's no other
reason to do it. And, similarly, there's no "neutral" way to present data either.
The "view from nowhere" is not real, and it's less honest to pretend that it is.
So be honest, and have an opinion.
