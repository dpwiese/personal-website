---
title: "Documentation"
date: 2019-12-06
draft: true
toc: false
tags:
  - pandoc
  - programming
  - latex
keywords: [pandoc, programming, latex]
---

<!--
---
images:
lang: "en"
geometry:
- margin=1in
urlcolor: "cyan"
fontsize: "10pt"
papersize: "letter"
---
-->

<!--
  https://wkhtmltopdf.org/
  This essentially seems equivalent to printing from a web browser
  wkhtmltopdf http://google.com google.pdf
  wkhtmltopdf http://localhost:1313/posts/documentation/index.html documentation-3.pdf
-->

<!--
  https://www.mkyong.com/mac/sed-command-hits-undefined-label-error-on-mac-os-x/
  $ sed -i '.bak' 's/ {linenos=false}&nbsp;//g' test.md
  $ sed 's/ {linenos=false}&nbsp;//g' test.md | tee test-out-2.md
-->

# Introduction

Prior to creating this site, I would often document the things I was working on.
While in grad school this was primarily in Latex, documenting hundreds of pages of in my <a href="https://danielwiese.com/mit-notes.pdf" target="_blank">grad school notes</a>.
For the past few years my work has been less academic, less mathematical, and involved more programming.
It was also not often distributed or published.
As such my documentation lately has been primarily in Markdown.
I found myself using Atom with the <a href="https://atom.io/packages/markdown-pdf" target="_blank">markdown-pdf</a> plugin to periodically generate PDFs when necessary.
This approach was fine, but it provided no real support for equations, and Atom was not my prefered editor.

I was looking for a better solution than this that provided the following:

- **Short syntax** that is easily human readable (e.g. markdown headings as `#` instead of Latex `\section{}` or HTML `<h1></h1>`).
  This was to make my writing quicker and more efficient, and document source easier to read.
  From this perspective, markdown was an attractive option.
  For example, bullets in markdown source look like bullets, versus those in Latex or HTML.
  - This was could be facilitated further by good syntax highlighting in an editor, e.g. Sublime bolding headings `#` and `**bold text**`.
- **Latex equation** support.
  While it is expected that equations be infrequent and generally simple, they needed to be supported.
- **Cross platform** should allow the source document to be easily deployed across many platforms.
  For example:
  - Github wikis
  - Web via a static site generator like Hugo, used to <a href="https://danielwiese.com/posts/site-setup/" target="_blank">create this site</a>
  - PDF
  I don't often intend for documents to be duplicated across these mediums, *rather I wanted to be able to adopt a standard way of writing without thinking what the destination format would be when writing*.
- **Consistent styling** across the above platforms.
  For example, I wanted to be able to style a generated PDF to look similar to the theme used on this site, or Github.
- **Minimal tooling** required.
- **No dedicated app** required.
- **Fast to compile or view**. However, the need for this was inversely proportional to the complexity of the document syntax. That is, if I could adopt a solution with a sufficiently simple syntax that allowed it to be easily read, then it would reduce the frequency of compiling to view the generated output.
- Features/nice to have
  - Inclusion of references from .bib file?
  - Good syntax highlighting of the source in editor

# Solution: Markdown Pandoc

After considering the above requirements, for a simple, easy-to-read syntax, markdown was a good choice. Furthermore, it is widely supported on and offline, with easy tooling to generate PDFs including <a href="https://pandoc.org/" target="_blank">Pandoc</a> and good support online at Github, Gitlab, <a href="https://gohugo.io/" target="_blank">Hugo</a>, Jekyll, and more.
Markdown supports embedded HTML and Latex, with tools like KaTex and MathJax. 

For the generation of PDFs, Pandoc is easy to use, and supports Latex.
Latex is not supported in Github, but seems to be in Gitlab.
Markdown is easy-to-read, reducing the need for real-time rendering, or a need to frequently and quickly compile.
References can be included from a `.bib` file.
Using the Sublime Text markdown syntax highlighting works well for markdown, although it does not do anything for Latex.
But as the primary goal was for a simple, easy syntax that *allowed* for the inclusing of equations rather than a focus on highly mathematical documents, this was not a big deal - equations should be relatively simple and infrequent.

Latex was an alternate source format considered, with Pandoc providing tools for conversion to markdown or HTML for use on the web, and easy generation of PDFs.
However, even with templates, Latex source is more verbose and and cumbersome to use for relatively simple note taking and documentation, most of what I do now.
Jupyter was considered as well, but seemed much more heavyweight than desired, required an app, and is not as widely or easily supported as markdown, although it is supported in Github, for example.

Styling consistency on the web (e.g. via Hugo) and generated PDFs (e.g. via Pandoc and Latex) may not be easily maintained though.
The CSS used for the web could not necessarily be applied to PDFs and vice versa.
But of the above requirements styling is not the most important.
Once the desired styling is set for each of these outputs, it will likely not often be changed.
There may even be options to use CSS with Pandoc and Latex.

## Using Pandoc

The Basic Pandoc command for generating `doc.pdf` from `doc.md` is:

```bash {linenos=false}&nbsp;
$ pandoc doc.md -o doc.pdf
```

For more about Pandoc check out the <a href="https://pandoc.org/MANUAL.html" target="_blank">Pandoc User’s Guide</a>.

## Bibliography

A bibliography, in the form of a `.bib` can be easily be included with Pandoc.
To format the references, the <a href="https://citationstyles.org/" target="_blank">Citation Style Language</a> can be specified.
Thousands of CSL files can be found <a href="https://github.com/citation-style-language/styles" target="_blank">here</a>.
The following Pandoc options can be used to include the bibliography.

```bash {linenos=false}&nbsp;
--filter pandoc-citeproc \
--bibliography=test.bib \
--csl ieee.csl \
```

Citations are accomplished by `[@my-citation]`.

## Styling

To style the Pandoc generated output, <a href="https://github.com/jgm/pandoc/wiki/User-contributed-templates" target="_blank">several options</a> for templates were available that can be used with the `--template` option.
The <a href="https://github.com/Wandmalfarbe/pandoc-latex-template" target="_blank">Eisvogel</a> Pandoc Latex template was one of the simplest and easiest.
Just required it to be downloaded, put in the default pandoc template location `~/.pandoc/templates/`.and used with `--template eisvogel`.
The result out-of-the-box is quite good, additional styling options will be described below.

Docs on Pandoc's different flavor of markdown described in the docs: <a href="https://pandoc.org/MANUAL.html#pandocs-markdown" target="_blank">Pandoc’s Markdown</a>
It says in the post <a href="https://learnbyexample.github.io/tutorial/ebook-generation/customizing-pandoc/" target="_blank">Customizing pandoc to generate beautiful pdfs from markdown</a>:

> GitHub style markdown is recommended if you wish to use the same source (or with minor changes) in multiple places.

I chose to use `markdown` instead, as `yaml_metadata_block` not supported by `gfm`, nor does not work with Eisvogel template.

## Bash Script

With the Pandoc options above, the command to run Pandoc was becoming quite long.
As was well described in the blog post <a href="https://learnbyexample.github.io/tutorial/ebook-generation/customizing-pandoc/" target="_blank">Customizing pandoc to generate beautiful pdfs from markdown</a> using a simple script to call Pandoc was an obvious soultion.
Calling Pandoc to convert a markdown to PDF required the following command:

```bash {linenos=false}&nbsp;
$ ~/.pandoc/md2pdf.sh doc.md ~/Desktop/doc.pdf
```

The contents of the script `md2pdf.sh` with all of the final options will be listed below.

# Markdown Processing in Pandoc and Hugo

With the above, the first problem I encountered when attempting to generate a PDF from the source from this site was in the differences between the markdown processor of Hugo versus that of Pandoc.
With the markdown processor in Hugo I can pass extra arguments to a code block, for example specifying that I want line numbers turned off, as shown below.
However, when code such as this is used in the markdown file and Pandoc is called, it does not know how to interpret these extra arguments and ends up rendering the code weirdly.
*Again, it was not necessarily a primary use case that I generate PDFs from the posts on this site, but I wanted to have the flexibility to do so.*

````md
```js {linenos=false}
var your = "code here"
```
````

Here is what it looks like:

<!-- intentionall leave this {linenos=false} and do not remove with sed -->
```js {linenos=false}
var your = "code here"
```

As we can see, it looks just fine in Hugo - a nicely formatted code block without line numbers.
However, the PDF generated by Pandoc using the command below looks very bad.

In the PDF, the code is formatted as in-line code rather than in a block, all of the statements are on a single line, and `{linenos=false}` appears in the output as if it were inside the code block.
The reason for that is that Pandoc cannot intepret the `{lineos=false}`.
So we need a way to be able to specify for Pandoc to ignore this argument.

## Solution Option 1: Don't use Pandoc

This is the obvious solution.
For markdown that is used to generate content on the web, I don't necessarily need a separate way to generate a document.
I can always just print the website to PDF.
This was again important to acknowledge, but not a viable solution to the underlying problem - how to handle flavors or features of markdown that may not be supported by Pandoc.

## Solution Option 2: HTML/CSS Tricks and Pandoc Arguments

One solution is to segregate markdown that is to be processed by Pandoc versus that that is to be processed by Hugo and the corresponding markdown engine.
This could be accomplished using the following code, and using the Pandoc option `--from markdown-markdown_in_html_blocks-native_spans`.
This tells Pandoc to process the HTML `span` with `class="hide-me"` thus showing its contents in the PDF generated by Pandoc.
At the same time, when processed by Hugo for the web, the HTML `span` with `class="hide-me"` will be hudden using CSS `display: none;`.
And the `p` element will naturally show in Hugo, but is hidden from Pandoc.

````html
<!-- this shows in Pandoc -->
<span class="hide-me">

```js
var your = "code here"
```

</span>

<!-- this shows in Hugo -->
<p>

```js {linenos=false}
var your = "code here"
```

</p>
````

And the result of the solution to this problem are as desired.

So basically via some cumbersome HTML and CSS, I can define parts that show up either in the Pandoc PDF output, or on the web, and thus can use each of these parts differently depending on how each markdown processor will use them.
Now this looks correct on the web via Hugo, and in the PDF output generated by Pandoc.
Of course, they will be different slightly in that the code block on the web will not have line numbering due to the `{linenos=false}`, but this will not be the case with Pandoc.

This is at best horribly inelegant, requiring extra HTML elements and significant duplicated code just because the markdown processor of Pandoc is different than that of Hugo.
I initially thought this would be a perfect use case for a Shortcode and then of course realized that Shortcodes could not be interpreted by Pandoc either.

## Solution Option 3: Pandoc Filters

This seemed to be a very viable option.
There is lots of information on Pandoc filters written in Python, php, Lua, etc. online.
This solution also seemed to the most elegant and flexible.
This case would require creating a filter called, for example, `filter.lua` and use the Pandoc option `--lua-filter=filter.lua`.

Spent a few minutes looking at the filters, but realized it might be a bit involved, so set this option aside to come back to after seeing if there may be more easier options.
The <a href="https://pandoc.org/filters.html" target="_blank">Pandoc Filters</a> docs were a useful reference.
As filters were an interesting option, I was particularly interested in 
<a href="https://pandoc.org/lua-filters.html" target="_blank">Pandoc Lua Filters</a>, as they seemed to be frequently used with success.
Can also write filters in Python using Panflute, as described in the blog post <a href="https://lee-phillips.org/panflute-gnuplot/" target="_blank">Technical Writing with Pandoc and Panflute</a>.

## Solution Option 4: Replace Offending Argument with sed

As the current problem was limited to one particular problem, the occurance of `{linenos=false}`, the stream editor sed could be easily used to look through whole markdown file, replace offending code, and plumb the output into Pandoc.
This way was very fast to understand and implement, more elegant than the HTML/CSS hacking above, and seemed somewhat flexible.
But in order to ensure I could specify which parts of code I actually wanted to remove (and not occurences in the text, like `{linenos=false}`) I tacked on a `&nbsp` at the end.
Then can run sed and Pandoc as follows.

```bash
$ sed 's/ {linenos=false}&nbsp;//g' documentation.md > temp.md; \
  pandoc temp.md \
  -o documentation.pdf; rm temp.md
```

The result is the following, no line numbers for the desired code blocks on the web, and correctly rendered PDF output from Pandoc.

````md
```js {linenos=false} &nbsp;
var test
```
````

Of course the doesn't enable the code block line numbers to be selectively turned off in the Pandoc output.

It's not the most elegant solution as it requires remembering to include a superfluous `&nbsp;` after each `{linenos=false}` and as such will not scale well depending on how many other commands I rely on in the future that are not compatible with Pandoc, and adds a bit of an ugly extra step to the pandoc script.
But for now it's a decent solution with low overhead.
Should I revisit this later to come up with a better solution, I can simply `grep` my notes and remove this `&nbsp;` or just leave it in there, as it doesn't really hurt anything.
And yet again, it is unlikely that markdown from this site will be given to Pandoc anyway.

## Solution Options Recap

Something like using a filter probably the best.
Not sure how fast compared to sed.
Especially with the way I have now with file writes.

# Styling Pandoc Output

Using a downloaded template such as with `--template eisvogel` got us a PDF output that looked pretty decent.
But further customization was needed.

## Styling Fonts

Use Pandoc options to change fonts:

```bash {linenos=false}&nbsp;
-V mainfont="SFNS Display" \
-V monofont="Menlo Regular" \
```

Can use latex header, for example `headings.tex` and add custom font:

```tex {linenos=false}&nbsp;
\newfontfamily\sfnsdisplaybold{SFNS Display Bold}
\sectionfont{\fontsize{16pt}{16pt}\selectfont\sfnsdisplaybold}
```

On Mac, font is in `~/Library/Fonts`.
Include this header with `--include-in-header ~/.pandoc/headings.tex`

Can see what fonts are installed with:
```bash {linenos=false}&nbsp;
fc-list | grep "SF-Pro-Text-Regular"
```

San Francisco font, for example can be downloaded in `.ttf` <a href="https://github.com/supermarin/YosemiteSanFranciscoFont" target="_blank">here</a>.

## Styling Code

### Listings

First option is to use the `--listings` option with Pandoc.
This made block code look quite nice, but in-line code I was not happy with.
The use of the `--listings` option put inline code in `lstinline` and block code in a `lstlisting` environment.
Easiest way to see this was by just outputting `.tex` document from Pandoc.
This could even then be altered and output generated with whatever Latex engine, in this case I was using XeLatex.

The solution when using `--listings` was to use a latex header to style these environment(s) as desired.
For example, made `listings-code.tex` and included with `--include-in-header ~/.pandoc/listings-code.tex` as with our header for heading fonts.

The contents of this header were overly complex.
Not going to go into detail here, but some helpful references were:

A gist with a bunch of options for `lstset`: <a href="https://gist.github.com/nhtranngoc/88b72d9bfb656a3de227eea38ed80627" target="_blank">LaTex settings for embedding Python with Monokai theme</a>.

<a href="https://tex.stackexchange.com/questions/30845/how-to-redefine-lstinline-to-automatically-highlight-or-draw-frames-around-all/30851#30851" target="_blank">How to redefine &#92;lstinline to automatically highlight or draw frames around all inline code snippets?</a>

<a href="https://tex.stackexchange.com/questions/357227/adding-background-color-to-verb-or-lstinline-command-without-colorbox" target="_blank">Adding background color to &#92;verb or &#92;lstinline command without &#92;Colorbox
</a>

<a href="https://tex.stackexchange.com/questions/64750/avoid-line-breaks-after-lstinline" target="_blank">Avoid line breaks after &#92;lstinline</a>

<a href="https://tex.stackexchange.com/questions/28179/colored-background-in-inline-listings" target="_blank">Colored background in inline listings</a>

The result was not good, the inline code just didn't look quite right.

`lstset{keepspaces=true}`

### Highlight Style

When not using the `--listings` option, inline code is in `texttt{}` and highlight env? for block code.

Can use a Pandoc style.
Can export it and modify it.
Can use modified version as:

`--highlight-style ~/.pandoc/pygments-mod.theme`

This provides a very basic styling only to highlight environment.
This is well described in the Docs: <a href="https://pandoc.org/MANUAL.html#syntax-highlighting" target="_blank">Syntax highlighting</a>.

Not really a viable option.

Using the `fancyvrb` package, there weren't any great options to color the background.
Most of them like the ones in the Stack Overflow thread <a href="https://tex.stackexchange.com/questions/163412/how-fancyvrb-background-color-fill-completely-with-fillcolor" target="_blank">how fancyvrb background color fill completely with fillcolor?</a> suggested using `listings`.

### Custom Latex Header

Can easily apply styling to `texttt`.

Looks like `fvextra` package is used.
(Is that particular to Eisvogel theme?)
Can then easily use `fvset` to apply options to verbatim environment used for block code.
Now things are looking pretty good.

The Latex header below with Pandoc works great, coloring the background of both inline and block code, when `listings` is not used.
The biggest downsite is that in block code (without `listings`) there doesn't seem to be a way to add line numbers.
If there were, this would probably be the desired solution for me.

```tex
\definecolor{inline-background}{RGB}{244,244,245}
\let\oldtexttt\texttt
\renewcommand{\texttt}[1]{
  \colorbox{inline-background}{\fontsize{8pt}{8pt}\oldtexttt{#1}}
}
```

When not using `listings` the `.tex` output of Pandoc has, for inline code:

```tex {linenos=false}&nbsp;
\texttt{**bold\ text**}
```

and for block code:

```tex
\begin{Shaded}
\begin{Highlighting}[]
\CommentTok{\# Basic command for generating documentation.pdf from documentation.md}
\NormalTok{$ }\ExtensionTok{pandoc}\NormalTok{ documentation.md {-}o documentation.pdf}
\end{Highlighting}
\end{Shaded}
```


## Styling Hyperlinks

Lua filter to handle HTML links from markdown document and keep the links in generated PDF.
Found a suitable filter in the Stack Overflow answer <a href="https://stackoverflow.com/questions/52958312/html-formatted-hyperlinks-not-preserved-in-bookdown-pdf" target="_blank">HTML-formatted hyperlinks not preserved in bookdown PDF</a>.
Then use another latex header to style the links. `--include-in-header ~/.pandoc/link-color.tex`

There were also similar filters using Panflute as described here: <a href="https://gist.github.com/dixonsiu/28c473f93722e586e6d53b035923967c" target="_blank">How to convert markdown link to html using Pandoc</a>


## Styling Recap


# Conclusions

Between this and the website setup, do a few things.

* Links as HTML with attribute `target="_blank"`.
* When the link text is a valid hyperlink itself, break it up inside with an empty `<span></span>`.
* Use MathJax to render Latex.
* When omitting line numbers in Hugo, follow the argument with a character such as a non-breaking space: `{linenos=false} &nbsp;`.
* Use sed before Pandoc to remove the argument Pandoc doesn't understand.
* Make sure to not use relative links for anything that will end up in PDF, or the relative link will be broken.
```html
<a href="../../mit-notes.pdf" target="_blank">grad school notes</a>
Use this instead:
<a href="https://danielwiese.com/mit-notes.pdf" target="_blank">grad school notes</a>
```












https://tex.stackexchange.com/questions/13515/how-to-add-paragraph-line-number-in-right-margin

```tex
\modulolinenumbers[1]

\begin{mdframed}
\internallinenumbers
\begin{Shaded}
\begin{Highlighting}[]
\CommentTok{\# Basic command for generating documentation.pdf from documentation.md}
\NormalTok{$ }\ExtensionTok{pandoc}\NormalTok{ documentation.md {-}o documentation.pdf}
\end{Highlighting}
\end{Shaded}
\end{mdframed}

\begin{mdframed}
\internallinenumbers
test \\
test
\end{mdframed}

\begin{linenumbers}
When you watch the news and see pictures of weather from around the United States 
or the world, you are seeing data from NOAA's environmental satellites. NOAA's 
environmental satellites provide data from space to monitor the Earth to analyze the 
coastal waters, relay life-saving emergency beacons, and track tropical storms and 
hurricanes. 
\end{linenumbers}
```
