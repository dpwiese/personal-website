---
title: "Markdown Images"
date: 2024-03-21T07:40:58-04:00
draft: true
toc: false
images:
tags:
  - untagged
---

# Introduction

As I described four years ago in my post on [Documentation](/posts/documentation/), I had discovered [Pandoc](https://pandoc.org) and transitioned to writing my notes almost exclusively in Markdown.
These notes were written either to be posted to this site, or to generate a PDF with Pandoc.
The initial configuration of Hugo and Pandoc to meet my needs was generally pretty straightforward, and hasn't changed much over the years.
My Pandoc user data directory can be found at [https://github.com/dpwiese/.pandoc](https://github.com/dpwiese/.pandoc).

Achieving the result I wanted was generally pretty straightforward -- the vast majority of what I wrote was Markdown, with the use of a little bit of HTML or LaTeX sprinkled in.
Initially my goal was to be able to write in such a way that the same file, without modification, could be used in Hugo or rendered to a PDF with Pandoc, but I quickly realized I didn't really have a *need* to do this (I would almost always write for one destination or another, not both).
Furthermore, I didn't want to accept add constraints to how I wrote the document in order to enable consistency.
For example, when writing with the intention of generating a PDF I wanted to be able to use LaTeX commands like `\newpage` without worrying about what to do with these in Hugo.
I also wanted to keep the associated tooling as simple as practicable and not have to add an abundance of shortcodes and Lua filters to transform the underlying source in order to make it work for both destinations.
In the end I was very happy with the workflow: write almost exclusively in Markdown, use a bit of HTML and LaTeX when needed, and use some simple Lua filters to enable basic HTML to work with Pandoc.

This approach worked very well, having used it to write the content on this site, as well as well over a thousand pages of generated PDFs.
Whenever I found myself needing to do something in a way that wasn't obvious, I'd quickly pull up previous document source to remind myself what I had done before.
As the volumes of notes grew I found myself struggling a bit to remember what I had done before and where I had done it.
Specifically, when it came to including images I would find myself not using Markdown syntax to give me the control I need.
**This post started as place to document a few snippets of code to include images in Markdown which can be used on the web or rendered to a PDF with Pandoc, but ended up evolving to include a bit of additional shortcodes and Lua filters to simplifying this a bit.**

# The Problems

Simple syntax that, while need not be common for use on the web (e.g. with Hugo) and when rendering PDFs with Pandoc, should aim to be as similar or obvious as possible.

1. Be able to center images _without captions_ in Pandoc
2. Use hyperlinks in captions
3. Control image width

# 1. Centering Images in Pandoc

Here is a basic command to generate a PDF from Markdown source.

```sh
pandoc test.md -o test.pdf
```

Using this command with the following Markdown source produces a document with two images, where the first is left-justified and the second is centered with a caption.

```html
<!-- No caption, not centered -->
![](./test.png)

<!-- Caption, is centered -->
![Caption](./test.png)
```

If

```sh
pandoc test.md -o test.tex
```

Then

```tex
\includegraphics{./test.png}

\begin{figure}
\centering
\includegraphics{./test.png}
\caption{Caption}
\end{figure}
```

Using only the above, using HTML to include images in the Markdown source is not possible.
Using a Lua filter can allow the use of HTML.

```sh
pandoc test.md -o test.pdf --lua-filter ./lua-filter.lua
```

Using this command with the following Markdown source produces a document with the image and without a caption, but that will not be centered.

```html
<!-- No caption, will not be centered -->
<img src="./test.png"/>
```

With some more filtering and Pandoc arguments, we can define the figure and its caption using HTML and be able to generate a PDF.
For example

```sh
pandoc test.md -o test.pdf --lua-filter ./lua-filter.lua --lua-filter ./parse-html.lua --from markdown-markdown_in_html_blocks
```

with source below.
This will render the image with the caption, and because it is wrapped in `figure` it will be centered.

```html
<!-- Will be centered due to being wrapped in figure -->
<figure>
  <img src="./test.png"/>
  <figcaption>
    Caption here.
  </figcaption>
</figure>
```

However, none of this yet has given an easy way in Markdown to easily center an image *without a caption (or associated figure label) when generating a PDF with Pandoc.*
So while the use of HTML might be helpful in some cases, it wasn't really the obvious solution to the issue of centering images.
LaTeX commands can be used to center the image.

```html
<!-- No caption, will be centered -->
\centering

![](./test.png)

\raggedright
\flushleft
```

Combining HTML and LaTeX to be closer to how I can include images for Hugo, while centering.

```html
<!-- No caption, WILL be centered -->
\centering

<img src="./test.png" width="5in"/>

\raggedright
\flushleft
```
