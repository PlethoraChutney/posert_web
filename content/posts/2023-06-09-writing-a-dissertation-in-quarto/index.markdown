---
title: Writing a dissertation in Quarto
author: Rich Posert
date: '2023-06-09'
slug: []
categories: ['Guidance']
tags: ['Writing', 'Quarto']
---

I’ve really been enjoying writing my dissertation using [Quarto](quarto.org),
a nice new typesetting/computing wrapper around pandoc developed by the good
folks at Posit. I thought I’d write up the tips and tricks I had to develop to
get my document looking how I wanted it, in case anyone else ran into the same
problems as me. I’m sure there are smarter ways to solve all the problems I
list here, but this is what worked! I’ll update this post as I keep writing.

## Who is this guide for?

I’m aiming this guide at someone who is interested in getting their document
looking *just so*. If you want to just get words down on a page, use the
default Quarto settings, or just use Word. There’s no shame in caring about
substance over style.

Okay they’re gone. That’s insane to me. If it’s got my name on it, I want it to
look right.

Oh, also, this is about writing a dissertation — a book. If you’re working on a
paper, basically everything here applies except all the stuff I put in `_quarto.yml`
you’ll put in the YAML header of your document. And when you run `quarto render`
you’ll have to tell it to render the document instead of just running it without
specification.

Finally, I assume you have [passing familiarity with markdown](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) and you’ve glanced at
how Quarto works.

# Setup

## Before you start

1.  [Download Quarto](https://quarto.org/docs/get-started/)
2.  Pick your editor. I use [Visual Studio Code](https://code.visualstudio.com/)
    because I’ve got it set up just how I like it for writing,
    but the tips I’ve got here should mostly carry over. I’ll try to make it clear
    when something only applies to VS Code. Whatever you end up picking, be
    sure to install the Quarto plugin.
3.  [Download and use Zotero](https://www.zotero.org/) and the Zotero plugin,
    [Better BibTeX](https://github.com/retorquere/zotero-better-bibtex). Zotero
    really is the best citation management software, and Better BibTeX makes it easier
    to work with Quarto by automatically generating your citation keys and `.bib` files,
    and keeping those up to date whenever you add new references.

## Project Organization

I’ve got my project organized like this:

``` default
dissertation/
├── _quarto.yml
├── index.qmd
├── cell.csl
├── dissertation-reference.docx
├── henac.bib
├── parts/
│   ├── appendecies/
│   │   ├── code.qmd
│   │   ├── mat-meth.qmd
│   │   ├── processing-flow.qmd
│   │   ├── validation.qmd
│   │   └── figures/
│   │       └── {figure files}
│   ├── introduction/
│   │   ├── intro_enac-in-the-body.qmd
│   │   └── figures/
│   │       └── {figure files}
│   ├── introduction.qmd
│   ├── results/
│   │   ├── results_activating-conditions.qmd
│   │   ├── results_transmembrane-domain.qmd
│   │   └── figures/
│   │       └── {figure files}
│   └── results.qmd
├── todo.md
└── update-todo.sh
```

My basic thinking is to put non-writing stuff in or near the root directory, and put
all the actual writing into `parts/`, except `index.qmd`, which has
to be in the root directory. As you can see, you end up generating a ton of files.
Even though you could have everything in one big `writing/` directory, or even just
dump it all in the root dir, you’d end up taking ages to find everything. I also like
to know which figure goes with which `.qmd`.

Essentially, I want to have separation
of concerns. If something is wrong with *code* or *styling*, I look in the root dir.
If something is wrong with *what I’ve written* (and there’s always something), I
can quickly find the `.qmd` file by following the logic of the book itself. If you’re
young
[you’re probably more used to searching instead of navigating a file tree](https://www.theverge.com/22684730/students-file-folder-directory-structure-education-gen-z),
so this probably doesn’t matter as much and you can just dump stuff wherever you want.
I honestly wish I were you.

# Writing

## Citations

To get your citations working nicely with Quarto, I recommend you set up an auto-export
in Better BibTeX. To do that, hit `File > Export Library...` in Zotero to open the
export dialogue. Select `Better BibLaTeX` as the format and be sure that `Keep updated`
is checked. Then hit export and give it a name and save it into your Quarto directory.
Zotero will now automatically re-export your `.bib` file whenever you modify, add,
or remove a reference. I have found that by the time I alt-tab back to VS Code and have
typed in the first few characters of a reference key it’s already done. The thing is
magic.

`cell.csl` is a [Citation Style Language](https://citationstyles.org/) file. This
sets up your citations to look (in this case) like they would in Cell. There are
literally thousands of these to choose from. Quarto also has a default style, but
I find author/year citations to be unwieldy.

You point to your citations and style with these YAML keys:

``` default
bibliography: henac.bib
csl: cell.csl
```

I also like my PDF format to have clickable citation links, but I *don’t* want
those links to look blue (PDFs are printed, and blue on a printed document looks
so unprofessional). Here’s the YAML:

``` default
format:
  pdf:
    link-citations: true
    colorlinks: false
```

## Figures

I have all my figures in subdirectories of the parts rather than in the root directory
mostly for organizational purposes, as I said. If you prefer to have everything lumped
together this is less important. If, however, you like having your writing separated
out but will frequently be re-using figures across chapters/parts, you might consider
a root level `figures/` directory and telling Quarto to execute from the root, rather
than from the file location. To execute at the root rather than wherever the `.qmd`
file is, you’d want to add this to `quarto.yml`.

``` default
project:
  execute-dir: project
```

What this changes is what it means when you enter a path to a figure or data.
By default (and how I have my project set up), if my project looks like this:

``` default
root/
└── intro/
    ├── intro.qmd
    └── figures/
        └── {figure files}
```

and I tell Quarto to include `figures/enac.png` in the file `root/intro/intro.qmd`,
it’ll look for the file at
`root/intro/figures/enac.png`. However, with the `execute-dir` set to `project`,
it would instead look for `root/figures/enac.png`. That way you can use the same
path in multiple different files and all refer to the same figure.

Once you’ve decided on a structure, writing is pretty frictionless. Unlike writing
in LaTeX, I spend almost no time fighting with Quarto. I write my markdown, render it,
and it looks how I like it. That is, of course, after I did the initial setup to
style my document.

## Asides

I have ADHD, which means I get easily distracted and love going on tangents. Since
my dissertation is something that only a few people will ever read and has only to
conform to the most basic requirements, I decided I wanted
to capture that by writing asides in the margins, like so:

<figure>
<img src="hamsters-sweet.png" alt="A screenshot of my dissertation PDF with an aside about hamsters’ ability to taste artificial sweetener." />
<figcaption aria-hidden="true">A screenshot of my dissertation PDF with an aside about hamsters’ ability to taste
artificial sweetener.</figcaption>
</figure>

It’s very easy (perhaps too easy) to incude these asides in Quarto, so I’ve written
quite a few. Both PDF and HTML outputs handle them well, but Word dumps them into
the paragraph as a normal sentence. This really breaks up the flow of the writing,
since asides are usually self-contained thoughts. I wrote a [Lua filter](https://pandoc.org/lua-filters.html) to remove
them automatically from the DOCX output:

``` lua
return {
    Span = function (elem)
        if quarto.doc.is_format('docx') then
            if elem.classes:includes('aside') then
                return ""
            else
                return elem
            end
        end
    end
}
```

I put this filter in my root dir as `no_asides.lua`, then added this YAML to the
quarto project:

``` default
format:
  docx:
    filters:
      - no_asides.lua
```

# Styling

## Handling fonts

First off, you should [pick a nice font](https://practicaltypography.com/free-fonts.html).
However, many fonts (even ones you pay for!) don’t have all the glyphs we need as scientists.
For instance, I’m using Charter. But Charter doesn’t have greek characters or mathematical
symbols! This is fine for HTML and DOCX formats, as they’ll handle fallback all on their own.
However, LaTeX does exactly what you tell it and nothing more. When it runs into a glyph
your font doesn’t have, it will warn you (an warning like “Missing character:” in the log)
and just put a blank space there.

To get around this problem, I installed [Fira Sans](https://fonts.google.com/specimen/Fira+Sans/glyphs)
which looks pretty good and has most of the glyphs you could want. Then, I set up
the [ucharclasses](https://ctan.org/pkg/ucharclasses?lang=en) package, which lets you
define behavior when entering and exiting a Unicode block. For instance, I know
that when LaTeX encounters a Greek glyph, it should switch to Fira Sans, then switch
back when it encounters a Latin glyph. Here’s the YAML:

``` default
format:
  pdf:
    include-in-header:
      - text: |
        \usepackage{fontspec}
        \newfontfamily\symbolfont{Fira Sans}
        \setmainfont{Charter}
        \setsansfont{Montserrat}
        \setmonofont{JetBrains Mono NL}
      
        \usepackage{newunicodechar}
        \DeclareTextFontCommand{\textfallback}{\symbolfont}
        \newunicodechar{→}{\textfallback{→}}
        \newunicodechar{₂}{\textfallback{₂}}
        \newunicodechar{⁻}{\textfallback{⁻}}
        \newunicodechar{ă}{\textfallback{ă}}
        \newunicodechar{≠}{\textfallback{≠}}
      
        \usepackage{ucharclasses}
        \setTransitionsForGreek{\begingroup\symbolfont}{\endgroup}
```

The first block loads the fonts I’m using throughout the document.
The second block uses the `newunicodechar` package to define behavior for specific
symbols that don’t neatly fall into a block for `ucharclasses` to handle.
The final two lines set the behavior to and from Greek.

# Conclusion

That’s most of what I’ve done so far! I’ll reproduce my full YAML config here so you
can see how it all comes together.

``` default
project:
  type: book

book:
  title: ENaC isn't ASIC
  subtitle: "A Study of Various Activating Mutations and Conditions of the Epithelial Sodium Channel"
  author:
    - name: "Richard Posert"
      affiliation: Baconguis Lab, Vollum Institute, OHSU
  date: "7/21/2023"
  chapters:
    - index.qmd
    - part: parts/introduction.qmd
      chapters:
       - parts/introduction/intro_enac-in-the-body.qmd
    - part: parts/results.qmd
      chapters:
        - parts/results/results_activating-conditions.qmd
        - parts/results/results_transmembrane-domain.qmd
    - parts/references.qmd
  appendices: 
    - parts/appendices/mat-meth.qmd
    - parts/appendices/processing-flow.qmd
    - parts/appendices/validation.qmd
    - parts/appendices/code.qmd
  navbar: 
    logo: figures/other/domain-cartoon_ECD.png
  search:
    location: navbar
    type: overlay
  downloads:
    - pdf
    - docx
  output-file: posert-dissertation

csl: cell.csl
toc: true
bibliography: henac.bib
format-links: true

format:
  html: 
    filters:
      - lightbox
    lightbox:
      match: auto
    mainfont: Charter
    theme:
      light: flatly
      dark: darkly
  pdf:
    documentclass: scrbook
    pdf-engine: latexmk
    pdf-engine-opt: --xelatex
    latex-max-runs: 3
    keep-tex: false
    link-citations: true
    colorlinks: false
    geometry:
      - left=2in
      - right=2in
      - marginparwidth=1.5in
      - twoside=true
    include-in-header:
      - text: |
          \usepackage{fontspec}
          \usepackage{newunicodechar}
          \newfontfamily\symbolfont{Fira Sans}
          \setmainfont{Charter}
          \setsansfont{Montserrat}
          \setmonofont{JetBrains Mono NL}

          \DeclareTextFontCommand{\textfallback}{\symbolfont}
          \newunicodechar{→}{\textfallback{→}}
          \newunicodechar{₂}{\textfallback{₂}}
          \newunicodechar{⁻}{\textfallback{⁻}}
          \newunicodechar{ă}{\textfallback{ă}}
          \newunicodechar{≠}{\textfallback{≠}}
          % naughty naughty but who cares, just don't use a non-breaking hyphen
          \newunicodechar{‑}{\textfallback{-}}

          \usepackage{ucharclasses}
          \setTransitionsForGreek{\begingroup\symbolfont}{\endgroup}

          \usepackage{lscape}
          \newcommand{\blandscape}{\begin{landscape}}
          \newcommand{\elandscape}{\end{landscape}}
  docx:
    reference-doc: dissertation-reference.docx
    filters:
      - no_asides.lua
```
