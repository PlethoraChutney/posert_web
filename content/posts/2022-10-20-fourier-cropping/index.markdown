---
title: Fourier Cropping
author: Rich Posert
date: '2022-10-20'
slug: []
categories: ['Explainers']
tags: ['cryoEM', 'Image Processing', 'Math']
math: mathjax
cover:
  image: 'cover.png'
  alt: 'An aliased wave.'
  relative: true
---

Fourier cropping can be a bit confusing, especially because there aren’t great
examples of what it all means online. I thought I’d make a few examples here.

## TLDR

When you scale an image from 100 pixels to 20 pixels wide, you’re sampling the same
image five times less frequently. This means there are many freuqncies (resolutions)
which you can no longer accurately represent. Rather than just “going away”,
these frequencies add an unreal, lower frequency wave to your image. Fourier cropping
prevents this from happening.

## Sampling

Let’s start with a simple sine wave. We’ll sample this sine wave at one hundred
points from 0 to 2π:

<div id="xzgadgzvnt" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#xzgadgzvnt .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#xzgadgzvnt .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#xzgadgzvnt .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#xzgadgzvnt .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#xzgadgzvnt .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#xzgadgzvnt .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#xzgadgzvnt .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#xzgadgzvnt .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#xzgadgzvnt .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#xzgadgzvnt .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#xzgadgzvnt .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#xzgadgzvnt .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#xzgadgzvnt .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#xzgadgzvnt .gt_from_md > :first-child {
  margin-top: 0;
}

#xzgadgzvnt .gt_from_md > :last-child {
  margin-bottom: 0;
}

#xzgadgzvnt .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#xzgadgzvnt .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#xzgadgzvnt .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#xzgadgzvnt .gt_row_group_first td {
  border-top-width: 2px;
}

#xzgadgzvnt .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#xzgadgzvnt .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#xzgadgzvnt .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#xzgadgzvnt .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#xzgadgzvnt .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#xzgadgzvnt .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#xzgadgzvnt .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#xzgadgzvnt .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#xzgadgzvnt .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#xzgadgzvnt .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-left: 4px;
  padding-right: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#xzgadgzvnt .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#xzgadgzvnt .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#xzgadgzvnt .gt_left {
  text-align: left;
}

#xzgadgzvnt .gt_center {
  text-align: center;
}

#xzgadgzvnt .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#xzgadgzvnt .gt_font_normal {
  font-weight: normal;
}

#xzgadgzvnt .gt_font_bold {
  font-weight: bold;
}

#xzgadgzvnt .gt_font_italic {
  font-style: italic;
}

#xzgadgzvnt .gt_super {
  font-size: 65%;
}

#xzgadgzvnt .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 75%;
  vertical-align: 0.4em;
}

#xzgadgzvnt .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#xzgadgzvnt .gt_indent_1 {
  text-indent: 5px;
}

#xzgadgzvnt .gt_indent_2 {
  text-indent: 10px;
}

#xzgadgzvnt .gt_indent_3 {
  text-indent: 15px;
}

#xzgadgzvnt .gt_indent_4 {
  text-indent: 20px;
}

#xzgadgzvnt .gt_indent_5 {
  text-indent: 25px;
}
</style>
<table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">x</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1" scope="col">y</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td class="gt_row gt_right">0</td>
<td class="gt_row gt_right">0</td></tr>
    <tr><td class="gt_row gt_right">0.0628</td>
<td class="gt_row gt_right">0.0628</td></tr>
    <tr><td class="gt_row gt_right">0.1257</td>
<td class="gt_row gt_right">0.1253</td></tr>
    <tr><td class="gt_row gt_right">0.1885</td>
<td class="gt_row gt_right">0.1874</td></tr>
    <tr><td class="gt_row gt_right">0.2513</td>
<td class="gt_row gt_right">0.2487</td></tr>
    <tr><td class="gt_row gt_right">0.3142</td>
<td class="gt_row gt_right">0.309</td></tr>
    <tr><td class="gt_row gt_right">...</td>
<td class="gt_row gt_right">...</td></tr>
  </tbody>
  
  
</table>
</div>

We live in a discrete world, so we necessarily represent waves by taking a
discrete number of *samples* and interpolating between them. In an image,
the samples would be pixels. If we plot the samples, they look like this:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="768" />

It looks nice, which should be no surprise. It’s quite well sampled.
What does it look like if we take one of every ten samples?

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="768" />

The blue line doesn’t look as good, but the overall shape is the same. And now
we can represent the curve in 1/10th the space. In fact, when we do that, the
curve looks much less choppy:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="76.8" />

Fundamentally, when we resize an image (or a graph of a wave), we’re reducing how often we sample
the subject of that image. We’re representing the same *physical object* with
fewer pixels. With something as simple as a single frequency sine
wave, it’s not that difficult. But what happens if we add two sine waves together?

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="768" />

OK! What if we now take every tenth sample of that?

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="768" />

OK, it looks choppy again, and the higher frequency component (the little humps
in the larger sine wave) seems to be damaged more than the lower frequency
component. But when we display it at a tenth the size, it still looks okay:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="76.8" />

Remember that when we resample, we’re simulating what happens automatically
when you display the *same image* using *fewer pixels*. I’m just showing it at the
same size so it’s easier to see what’s going on.

## Aliasing

In the cases above, even when we were sampling down by 10, we still had enough
pixels to more-or-less accurately represent the waves involved. What happens if
we *don’t* have room?

Below is a wave which cycles five times every cm:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="768" />

Another way to think of this “image” of this sine wave is that we’ve sampled it
*twenty times per cycle*:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="768" />

Instead of thinking about samples along a wave, let’s instead think about pixels
on a detector. The two representations are identical, but it makes it
a bit easier to remember we’re talking about taking *static samples* at a *fixed spacing*
along a signal:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="768" />

In every cm, we have 100 pixels. This means our pixel size is 0.1 mm. Let’s look
at what happens when we try to represent waves with higher and higher frequencies:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="768" />

Whoa!! What’s going on here? The wave with a true frequency of 200 cycles per
centimeter looks like it’s only cycling twice per centimeter! Also, let’s take
a closer look at the waves cycling at 10 and 90 cm$^{-1}$:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="768" />

They look almost identical! But if we compare the true waves, not just what our pixels
detected:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="768" /><img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-2.png" width="768" />

These are clearly different waves! Here’s where the pixel size comes in — remember
we’re only “checking” the value of the wave over the width of our pixel. If we only
update the value of the wave when we move to a new pixel:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="768" />

Ah hah! Because we’re sampling this fast wave so infrequently, the points happen
to line up and look like a wave of a much much lower frequency! This is called
**aliasing**.

For any wave sampled at a finite frequency, there are actually an
infinite number of waves which could have given rise to the exact same sample
values. These are called *aliases*. One way to think about aliasing is to imagine
the Fourier transform of your signal, rather than the signal itself. Any signal
with a frequency **greater than half your sampling frequency** will “fold over”
to the same value on the other side of half your sampling frequency. This critical
“folding point” is the Nyquist frequency.

For instance, in our case, we’re sampling 100 times per cm, so our Nyquist frequency
is 50/cm. Our 10/cm wave is well below that, so it is unchanged. However, the
90/cm wave is 1 + 0.8 the Nyquist frequency, and it “folds over” to 1 - 0.8 Nyquist,
which is 10/cm.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-15-1.png" width="768" />

As another example, let’s look at the addition of two waves. I’ll
go back to representing samples as points, since the graph is less busy that way.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-16-1.png" width="768" />

I’ve added 10/cm and 90/cm waves (in grey) together to get our final wave in black.
What happens if we sample this wave with a Nyquist frequency of 50/cm?

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-17-1.png" width="768" />

The 90/cm wave has folded over the Nyquist frequency, aliasing from 1 + 0.8 Nyquist (90/cm)
to 1 - 0.8 Nyquist (10/cm), so the sampled wave looks
like a single 10/cm wave! Interestingly, even though we’re adding two waves,
the sampled wave looks like it has a lower amplitude. This is becasue the phase
of a flipped wave is also flipped to be negative, so it subtracts from the
unflipped wave.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-18-1.png" width="768" />

In fact, if we used a 10/cm and 90/cm wave of the same amplitude:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-19-1.png" width="768" />

Our samples line up exactly at 0! What happens if we use a 110/cm wave, which is
2.1 times Nyquist, instead of 90/cm?

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-20-1.png" width="768" />

Now it looks like a 10/cm wave with an amplitude of two! The 110/cm wave has
its phase flipped when it crosses each integer multiple of the Nyquist frequency.
So first it aliases to a 90/cm wave with flipped phase (flipping from 2 + 0.2 Nyquist
to 2 - 0.2 Nyquist), then a 10/cm wave with the same phase (flipping from 1 + 0.8
Nyquist to 1 - 0.8 Nyquist).

## cryoEM Image Processing

That’s fine, but what does this all have to do with cryoEM? Well, remember ages
ago when I said that

> when we resize an image (or a graph of a wave), we’re reducing how often we sample
> the subject of that image. We’re representing the same *physical object* with
> fewer pixels.

This means that when we take an image and scale it down to make processing faster,
we’re inherently changing the sampling frequency as well.

Let’s use a sine wave as an example again. We’ll sample a 5/cm wave 50 times every
cm. That means our wave starts at 0.2 Nyquist:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-21-1.png" width="768" />

But suppose this is an image of a very large particle, and maybe to do some
initial cleaning I want to make it smaller. If I just binned the particle by 8 (that is,
took every group of 8x8 pixels and averaged them to a single pixel), that’s like
reducing the sampling rate of this wave by a factor of 8, down to 6.25/cm.
Here’s what the samples look like now:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-22-1.png" width="768" />

Uh oh! When we scaled down the particle, the Nyquist frequency changed from
25/cm to 3.125/cm. So the 5/cm wave is at 1 + 0.6 Nyquist now, meaning it aliases
to the wave at 1 - 0.6 times the new Nyquist:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-23-1.png" width="768" />

Remember, this happens automatically as a result of resizing the image! When we
take 1/8th as many samples of this wave, there is **no way** to prevent this aliasing
from happening! That’s why instead of just binning our particles, we **Fourier crop**
them.

### Fourier cropping

Let’s take the example above, with our 5/cm wave, and add a 1/cm wave and a 3/cm wave. We’ll perform
the same binning operation, meaning we’ll start with a sampling frequency of 50/cm
and reduce it to 6.25/cm.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-24-1.png" width="768" />

Now, when we naively bin this image down to our smaller size, the Nyquist changes.
The 1 and 3/cm waves are still below the new Nyquist of 3.125, but the 5/cm wave
aliases to 1.25/cm (as above).

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-25-1.png" width="768" />

You can see that both the true wave `\((F_R = [1, 3, 5])\)` and the aliased wave
`\((F_A = [1, 1.25, 3])\)` both fit the data just as well. Because there are an infinite
number of higher-frequency waves which fit any given set of samples, reconstruction
algorithms always take the *lowest frequency wave* which fits the samples. Generally,
this is the right thing to do — you wouldn’t want your 1/nm “where is my particle”
wave to look like a 1/Å “where is this electron” wave.

What this means for us is that if we just bin an image, it will give us **incorrect
results**! However, if we *first* filter out any frequencies we won’t be able to
represent with the new sampling rate (in this case, just the 5/cm wave), the aliased signals won’t show up.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-26-1.png" width="768" />
Now, this wave is clearly not *correct*, but at this sampling rate it’s impossible
to prefectly capture the true information. At at least we can sample this wave
sufficiently to accurately represent it!

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-27-1.png" width="768" />

And the resulting wave is much closer to the true data than the aliased wave!

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-28-1.png" width="768" />

## Conclusion

I hope with these contrived examples, it’s obvious why we scale images down in
Fourier space, rather than real space. You can imagine in a real image, with
thousands of frequencies of waves, the effects of binning are even more severe.

To summarize:

1)  Discrete samples taken of a continuous signal are fit equally well by an infinite
    number of waves, related by their distance from the Nyquist frequency, which is
    half the sampling frequency (or, the Nyquist rate, which is twice the pixel size)
2)  When we bin an image, we’re taking fewer samples of the same phenomenon
3)  Changing the Nyquist frequency aliases waves to lower frequency information
    which is not present in the “true” image
4)  Removing these frequencies first, in the Fourier domain, gives us a result
    that is truer to the original image.
