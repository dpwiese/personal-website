---
title: "Markdown Images"
date: 2024-03-21T20:38:58-04:00
draft: false
toc: false
images: ["img/posts/markdown-images/og-image.png"]
tags:
  - markdown
  - pandoc
  - hugo
keywords: [markdown, pandoc, hugo]
description: "This post describes the use of a Lua filter and Hugo layout to use a consistent Markdown image syntax with both Hugo and Pandoc."
---

# Introduction

As I described in [Documentation](/posts/documentation/), I had discovered [Pandoc](https://pandoc.org) and transitioned to writing my notes almost exclusively in Markdown.
These notes were written either to be posted to this site, or to generate a PDF with Pandoc.
The initial configuration of Hugo and Pandoc to meet my needs was generally pretty straightforward, and hasn't changed much over the years.
My Pandoc user data directory can be found at [https://github.com/dpwiese/.pandoc](https://github.com/dpwiese/.pandoc).

Achieving the result I wanted, whether in a Pandoc--generated PDF or on the web with Hug, was generally pretty straightforward -- the vast majority of what I wrote was in pure Markdown, with the use of a little bit of HTML or LaTeX sprinkled in.
Initially my goal was to be able to write in such a way that the same Markdown source, without modification, could be used in Hugo or rendered to a PDF with Pandoc, but I quickly realized I didn't really have a *need* to do this (I would almost always write for one destination or another, not both).
Furthermore, I didn't want to accept add constraints to how I wrote the document in order to enable this strict consistency.
For example, when writing with the intention of generating a PDF I wanted to be able to use LaTeX commands like `\newpage` without worrying about what to do with these in Hugo.
I also wanted to keep the associated tooling as simple as practicable and not have to add an abundance of shortcodes, layouts, and Lua filters to transform the underlying source in order to make it work for both destinations.
In the end I was very happy with the workflow: write almost exclusively in Markdown, use a bit of HTML and LaTeX when needed, and use some simple Lua filters to enable basic HTML to work with Pandoc.

This approach worked very well, having used it to write the content on this site, as well as well over a thousand pages of generated PDFs.
Whenever I found myself needing to do something in a way that wasn't obvious, I'd quickly pull up previous document source to remind myself what I had done before.
As the volumes of notes grew I found myself struggling a bit to remember what I had done before and where I had done it.
Specifically, when it came to including images I would find myself struggling to remember what approach I used to achieve the result I wanted.
**This post started as place to document a few snippets of code to include images in Markdown source which can then be used on the web with Hugo or rendered to a PDF with Pandoc, but ended up evolving to include a bit of additional shortcodes and Lua filters to simplifying this a bit.**

# The Problems

So, I set out trying to meet the following primary goals with a simple syntax that, while need not be common for use on the web via Hugo and when rendering PDFs with Pandoc, *should aim to be as similar and obvious as possible*.

1. Be able to center images _whether or not they are captioned_ in Pandoc.
2. Control image width (and other attributes).
3. Use hyperlinks in captions.

Below each of these problems is described in a bit more detail.

## 1. Centering Images in Pandoc

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

To be able to see *why* the use of a caption causes the second image to be centered, use Pandoc to generate the LaTeX with the following command.

```sh
pandoc test.md -o test.tex
```

In the generated `.tex` file we can see that when a caption is used, the image is wrapped in `figure` and `\centering` is applied.

```tex
\includegraphics{./test.png}

\begin{figure}
\centering
\includegraphics{./test.png}
\caption{Caption}
\end{figure}
```

While a Lua filter can be used to enable HTML be used with Pandoc, there wasn't an obvious path with HTML that enabled the centering of an image.
Some suggestions I found were to use a zero--width space to create an empty caption, but this still leaves the figure label, as opposed to no label at all.
**The best solution as of yet was just use LaTeX commands as below to center the image without a caption.**

```html
<!-- No caption, will be centered -->
\centering

![](./test.png)

\raggedright
\flushleft
```

## 2. Control Image Width

Building on the solution above, image width can be set for use with Pandoc with the following syntax.

```html
<!-- No caption, will be centered, image width is set -->
\centering

![](./test.png){width=4in}

\raggedright
\flushleft
```

This syntax does *not* work out-of-the-box with Hugo though.
The easiest solution was just use HTML.
Given I could use HTML in Hugo, I thought it would be worth investigating if it made sense to use HTML in Pandoc.

This combining of HTML and LaTeX would allow, for example, the setting of width in a way that is more consistent with Hugo, as at least without additional shortcodes, and still easy to use with Pandoc if I wanted a consistent syntax to use when writing documents for both use in Hugo and with Pandoc.
This was an option that may solve the problems identified above.

Using a Lua filter can allow the use of HTML, which I have found helpful when writing documents to be compiled to PDF with Pandoc.
One of the main reasons is familiarity with HTML makes it just about as fast to write as the equivalent Markdown syntax, but provides more flexibility.
To accomplish this, my poorly named [`lua-filter.lua`](https://github.com/dpwiese/.pandoc/blob/main/filters/lua-filter.lua) can be passed to Pandoc.

```sh
pandoc test.md -o test.pdf --lua-filter ./lua-filter.lua
```

Using this command with the following Markdown source produces a document with the image but without a caption.
This is expected, as it was the use of a caption that resulted in the centering of the image via inclusion inside of a `figure` with associated `centering` and `caption`.

```html
<!-- No caption, will not be centered -->
<img src="./test.png"/>
```

With some more filtering and Pandoc arguments, we can define a figure using HTML and be able to generate a PDF.

```sh
pandoc test.md -o test.pdf --lua-filter ./lua-filter.lua --lua-filter ./parse-html.lua --from markdown-markdown_in_html_blocks
```

Using the above command with the source below will render the image with the caption, and because it is wrapped in `figure` the corresponding LaTeX will also use a `figure` with associated `centering`, *with a figure label but no caption text*.

```html
<!-- Will be centered in figure environment with blank caption -->
<figure>
  <img src="./test.png"/>
</figure>
```

This would allow me to use nearly identical HTML with both Hugo and Pandoc and control image attributes like width, although it didn't solve the problem of centering uncaptioned images in Pandoc.
The existing solution to this problem was combine HTML and LaTeX as below.

```html
<!-- No caption, will be centered, and can set width for example -->
\centering

<img src="./test.png" width="4in"/>

\raggedright
\flushleft
```

**This was essentially the solution I had been using in Pandoc for a little while and overall it worked well, due partly to the fact that I very rarely included images that I didn't want to caption.**
It allowed basically the same HTML to be used for both Hugo and Pandoc, but with the addition of a bit of LaTeX in the rare case I wanted to center an uncaptioned image with Pandoc.

## 3. Use Hyperlinks in Captions

This was not a problem that I initially encountered either with Hugo or Pandoc, but as I was prototyping solutions was something I had broken with one of them, so I wanted to note it down as a requirement and make sure I was testing that any of my prototype solutions supported this.

# The Solution

Having gotten this far, simply having written down as a reference the various ways to manage images in Markdown for use with Hugo or Pandoc was already a huge help.
I probably could have stopped here and used this post as a very occasional reference.
I really didn't feel like the use of a little LaTeX or HTML was that big of a deal, and the impact on my productivity was measured in single digit minutes over the course of weeks.
However, it was annoying enough that I decided I would take at least a little bit of time to come up with a slightly better solution.

My intention was to:

1. Allow the use of Markdown to the extent possible, with the ability to e.g. set image size.
2. Make syntax more similar between that used with Hugo and Pandoc.
3. Remove the need for LaTeX commands to center images in Pandoc.
4. To the extent that non-Markdown syntax is required, keep it as simple and obvious as possible. For example, needing to use a bit of HTML, especially for the web is fine, but I didn't want to have to remember specific shortcodes for use with Hugo.

**The solution took the form of a Lua filter for use in Pandoc, and a layout in Hugo.**

## Pandoc

The solution for Pandoc was the following [`remove-empty-captions.lua`](https://github.com/dpwiese/.pandoc/blob/main/filters/remove-empty-captions.lua) Lua filter.
It finds images with empty captions (which ordinarily would just be added with `\includegraphics`) and forces them into a `figure` environment with `\centering`.
This immediately solved the problem of centering un-captioned images.
Nothing else with existing Markdown syntax was broken.

```lua
--[[
Adapted from: https://github.com/pandoc/lua-filters/blob/master/short-captions/short-captions.lua
]]--

if FORMAT ~= "latex" then
  return
end

function figure_image (elem)
  local image = elem.content and elem.content[1]
  return (image.t == 'Image')
    and image
    or nil
end

function Para (para)
  local img = figure_image(para)
  if not img or not img.caption or img.caption == '' then
    return nil
  end

  return pandoc.Para {
    pandoc.RawInline('latex', '\\begin{figure}[H]\n\\centering\n'),
    img,
    pandoc.RawInline('latex', '\n\\end{figure}\n\n')
  }
end
```

This solution does assume that I always want images to be centered and removes flexibility to set this when including the image, but this has never not been true for the documents I've written so far.
Furthermore, if I ever decide I want to change the behavior, the existence of this Lua filter gives a good place to work from.

## Hugo

The solution in Hugo was to use the following `render-image.html` layout which lives in `layouts/_default/_markup/render-image.html`.
This solution was a little more messy than the above.
With Markdown with Pandoc, image attributes such as width, for example, can be set using `![](./test.png){width=4in}` syntax as described in [8.17 Images](https://pandoc.org/chunkedhtml-demo/8.17-images.html) of the Pandoc docs.
This syntax does not work in Hugo.

Instead the answers as in [How to use image width in markdown?](https://discourse.gohugo.io/t/how-to-use-image-width-in-markdown/32508/6) and [Doing What Markdown Can't: Specifying Image Width and Height](https://teknikaldomain.me/code/markdown-image-sizes/) are to pass image attributes using the existing fields of a Markdown image.
Approaches like that described in [Responsive and optimized images with Hugo](https://www.brycewray.com/posts/2022/06/responsive-optimized-images-hugo/) from [brycewray.com](https://www.brycewray.com) is probably something to look into later, but for now I still wanted a simple, straightforward way to pass image attributes when using Markdown with Hugo.

Hugo's [Image render hooks](https://gohugo.io/render-hooks/images/) documentation describes the pieces of an image defined in Markdown, and more information on Markdown can be found [here](https://www.markdownguide.org/basic-syntax/).
Given how the image description is what is used by Pandoc to generate a figure caption, I wanted to keep this the same when including images using Markdown for use with Hugo.
This meant that `.Text` would always be used to contain the image caption (and `alt` text) and `.Title` could be used hijacked to pass the style attributes.

The `render-image.html` layout below parses image attributes in the `.Title` component of the form `{width=400}` and puts them the `img` tag.
This layout is not elegant *at all* -- it's the first working solution I arrived at during the prototyping process, and is provided below as-is.
At some point I'll go back and clean it up, but for now I'm prioritizing getting this post published and moving on with a working, albeit inelegant, implementation.

```html
{{ if .Title }}
{{ $styling := index (findRE `\{(.*?)\}` .Title) 0 }}
{{ $titleText := trim .Title $styling }}
  {{ if .Text }}
  <figure>
    <img src="{{ .Destination | safeURL }}" alt="{{ default "alt" .Text }}" {{ trim $styling "{}" | safeHTMLAttr }}>
    <figcaption>{{ .Text | safeHTML }}</figcaption>
  </figure>
  {{ else }}
    <img src="{{ .Destination | safeURL }}" {{ trim $styling "{}" | safeHTMLAttr }}>
  {{ end }}
{{ else }}
  {{ if .Text }}
  <figure>
    <img src="{{ .Destination | safeURL }}" alt="{{ default "alt" .Text }}">
    <figcaption>{{ .Text | safeHTML }}</figcaption>
  </figure>
  {{ else }}
    <img src="{{ .Destination | safeURL }}">
  {{ end }}
{{ end }}
```

# Usage

The following subsections describe how, using the solutions above, I can now include and set attributes on images when used with both Pandoc and Hugo.

## Usage in Hugo

With Hugo, images can be included as intended, with `.Title` being used to pass image attributes.
HTML is still an option as well for situations that exceed what can be done with the Markdown syntax.

```html
<!-- Centered image with width set using .Title string and .Text sets caption -->
![.Text is caption](/test.png "{width=400}")

<!-- Centered image with width set using .Title string and no caption -->
![](/test.png "{width=400}")

<!-- Caption with a link works -->
![Here's a caption with Markdown link: [danielwiese](https://danielwiese.com)](/test.png "{width=400}")

<!-- HTML works too, where captions and figure width are supported, as well as really any other HTML and CSS -->
<img src="/test.png" width="400" />
```

## Usage in Pandoc

With Pandoc, standard Markdown image syntax is used, with attributes tacked on.

```html
<!-- Centered image with image width set and no caption -->
![This is the caption](./test.png){width=4in}

<!-- Centered image with image width set and no caption -->
![](./test.png){width=4in}

<!-- Caption with a link works -->
![Here's a caption with Markdown link: [danielwiese](https://danielwiese.com)](/test.png){width=4in}

<!-- At least some HTML works, where captions and figure width are supported -->
<img src="./test.png" width="4in"/>
```

If a common syntax between Hugo and Pandoc was truly desired, the Lua filter could be modified to pull styling from the [title](https://www.markdownguide.org/basic-syntax/#adding-titles) and use that.
But for now, sticking with Pandoc's ability to handle extra image attributes is a good solution.

# Conclusion

All of this started out rather messy, with existing Lua filters and various Hugo and Pandoc configuration set.
It seemed clear that there is not one best, elegant way to solve all the problems and enable exactly the same syntax for Markdown used with Pandoc and Hugo, and there need not be.
As long as I can continue writing Markdown and achieve the desired result with "obvious" syntax, then I am happy.
The solutions above give flexibility I need to write productively in Markdown, both for use on the web with Hugo and to generate PDFs with Pandoc.
Finally, the existence of this post itself will be a much more helpful resource to consult than looking through old Markdown source, keeping everything I might want to reference in one place.
