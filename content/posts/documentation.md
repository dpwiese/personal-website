---
title: "Documentation"
date: 2019-12-06
draft: true
toc: false
images:
tags:
  - pandoc
  - programming
  - latex
keywords: [pandoc, programming, latex]
lang: "en"
geometry:
- margin=1in
urlcolor: "cyan"
fontsize: "10pt"
papersize: "letter"
---

<!-- To print this page with Pandoc:
  $ ~/.pandoc/md2pdf.sh documentation.md ~/Desktop/documentation.pdf
-->

<!-- To print this page with Pandoc:
  $ sed 's/ {linenos=false}&nbsp;//g' documentation.md > documentation-out.md; \
  pandoc documentation-out.md \
  --variable fontsize=10pt \
  --number-sections \
  --from markdown \
  --template eisvogel \
  --pdf-engine xelatex \
  --listings \
  -o ~/Desktop/documentation.pdf; rm documentation-out.md
-->

<!--
  https://wkhtmltopdf.org/
  This essentially seems equivalent to printing from a web browser
  wkhtmltopdf http://google.com google.pdf
  wkhtmltopdf ../../public/index.html index.pdf
  wkhtmltopdf ../../public/posts/documentation/index.html documentation-2.pdf
  wkhtmltopdf http://localhost:1313/posts/documentation/index.html documentation-3.pdf
-->

<!--
  https://www.mkyong.com/mac/sed-command-hits-undefined-label-error-on-mac-os-x/
  $ sed -i '.bak' 's/ {linenos=false}&nbsp;//g' test.md
  $ sed 's/ {linenos=false}&nbsp;//g' test.md | tee test-out-2.md
-->

<!--
  pandoc template location
  ~/.pandoc/templates
-->

<!--
  fc-list | grep "DejaVu Sans"
  fc-list | grep "SF-Pro-Text-Regular"
  -V mainfont="SFNS Display" \
  -V mainfont="San Francisco Display" \

  ~/Library/Fonts/
-->

# Introduction

Prior to creating this site, I'd always been quite good about documenting the things I've worked on.
While in grad school this was primarily in Latex, documenting hundreds of pages of in my <a href="https://danielwiese.com/mit-notes.pdf" target="_blank">grad school notes</a>.
For the past few years my work has been less academic, less mathematical, and involved more programming.
As such my documentation lately has been primarily in Markdown.

But no real support for equations, and Atom was not my prefered browser.
I was looking for a better solution, that provided the following:

- **Short syntax** that is easily human readable (e.g. markdown headings as `#` instead of latex `\section{}` or html `<h1></h1>`).
  This was to facilitate fast writing by maximizing the source code that is actually content, and also having syntax that doesn't get in the way of the content.
  For example, bullets in markdown source look like bullets, versus those in latex or html.
  - This was could be facilitated further by good syntax highlighting in an editor, e.g. Sublime bolding headings `#` and `**bold text**`.
- **Latex equation** support.
  While it is expected that equations be infrequent and generally simple, they needed to be supported.
- **Cross platform** should allow the source document to be easily deployed across many platforms.
  For example:
  - Github wikis
  - Web via a static site generator like Hugo, used to <a href="https://danielwiese.com/posts/site-setup/" target="_blank">create this site</a>
  - PDF
- **Consistent styling** across the above platforms.
  For example, generating a PDF can be easily styled to look like a Github wiki.
- **Minimal tooling** required.
- **No dedicated app** required.
- Fast to compile or view
- Features/nice to have
  - Inclusion of references from .bib file?
  - Good syntax highlighting of the mixed (markdown with latex, html, etc.) in editor (e.g. sublime)

# Solution: Pandoc

For a simple, easy-to-read syntax, markdown was a good choice. Furthermore, it is widely supported on and offline, with easy tooling to generate PDFs such as the <a href="https://atom.io/packages/markdown-pdf" target="_blank">markdown-pdf</a> and <a href="https://atom.io/packages/language-markdown" target="_blank">language-markdown</a> packages for Atom and <a href="https://pandoc.org/" target="_blank">Pandoc</a>; good support online at Github, Gitlab, <a href="https://jekyllrb.com/" target="_blank">Jekyll</a>, and more.

For the generation of PDFs, Pandoc is easy to use, and supports Latex. Latex is not supported in Github, but seems to be in Gitlab.
Markdown is easy-to-read, reducing the need for real-time rendering, or a need to frequently and quickly compile.
References can be included from a .bib file.
Using the Sublime Text markdown syntax highlighting works well for markdown, although it does not do anything for Latex.
But as the primary goal was for a simple, easy syntax that *allowed* for the inclusing of equations rather than a focus on highly mathematical documents, this was not a big deal - equations should be relatively simple and infrequent.

Latex was an alternate source format considered, with Pandoc providing tools for conversion to markdown or html for use on the web, and easy generation of PDFs.
However, even with templates, Latex source is more verbose and and cumbersome to use for relatively simple note taking and documentation.
Jupyter was considered as well, but seemed much more heavyweight than desired, required an app, and is not as widely or easily supported as markdown, although it is supported in Github, for example.

## Using Pandoc

```bash
# Basic command for generating documentation.pdf from documentation.md
$ pandoc documentation.md -o documentation.pdf
```

## Styling

To apply style more closely resembling Github markdown, <a href="https://github.com/jgm/pandoc/wiki/User-contributed-templates" target="_blank">several options</a> were available. The <a href="https://github.com/Wandmalfarbe/pandoc-latex-template" target="_blank">Eisvogel</a> Pandoc Latex template was one of the simplest and easiest. Default pandoc template location: `~/.pandoc/templates/`. This template can be easily applied with Pandoc. The result is pretty good enough.

## Bibliography

The bibliography can be easily be included. To format the references, the <a href="https://citationstyles.org/" target="_blank">Citation Style Language</a> can be specified. Thousands of CSL files can be found <a href="https://github.com/citation-style-language/styles" target="_blank">here</a>.

## Command

```bash
$ pandoc test.md \
    --variable fontsize=10pt \
    --number-sections \
    --filter pandoc-citeproc \
    --bibliography=test.bib \
    --csl ieee.csl \
    --from markdown \
    --template eisvogel \
    --listings \
    -o test.pdf
```

with YAML metadata:

```yaml
---
title: "Test Notes"
date: "November 2019"
author: "Daniel Wiese"
subject: "Markdown"
keywords: [Markdown, Example]
lang: "en"
geometry:
- margin=1in
urlcolor: "cyan"
fontsize: "10pt"
papersize: "letter"
---
```

# Markdown Processing in Pandoc and Hugo

The first problem I encountered was in differences between the markdown processor of Hugo versus that of Pandoc.
With the markdown processor in Hugo I can pass extra arguments to a code block, for example specifying that I want line numbers turned off, as shown below.
However, when code such as this is used in the markdown file and Pandoc is called, it does not know how to interpret these extra arguments and ends up rendering the code weirdly.

````md
```js {linenos=false}
var your = "code here"
```
````

Here is what it looks like:

```js {linenos=false}
var your = "code here"
```

As we can see, it looks just fine in Hugo - a nicely formatted code block without line numbers.
However, the PDF generated by Pandoc using the command below looks very bad.

In the PDF, the code is formatted as in-line code rather than in a block, all of the statements are on a single line, and `{linenos=false}` appears in the output as if it were inside the code block.
The reason for that is that Pandoc cannot intepret the `{lineos=false}`.
So we need a way to be able to specify for Pandoc to ignore this argument.

## Solution Option 1: Print the Website from Browser

Not to mention that until now, the `eisvogel.latex` style that I use with Pandoc is very different than the way this site is styled.
Reconciling these two via a new `.latex` style would be a huge pain, and require for any style changes be kept in sync between this file and the CSS used for this site.

Maybe there is a way to generate a PDF from Hugo?
The easiest way to do this is with the site open in a web browser, just print the page.
Honestly the output looks pretty good.
Really the only downsides I noticed (when printing from Chrome at least) was that there is some website navigation items present in the printed output, and that when including the headers and footers, they are not styled nicely at all - just some plain Times New Roman text.

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

## Solution Option 3: Pandoc (Lua) Filters

Seemed a viable option.
Lots of information on filters, python, php, lua, etc.
Create filter called, for example, `filter.lua`.
Run Pandoc with filter applied as shown below.


```bash
$ pandoc documentation.md \
  --lua-filter=filter.lua \
  --variable fontsize=10pt \
  --number-sections \
  --from markdown-markdown_in_html_blocks-native_spans \
  --template eisvogel \
  --pdf-engine xelatex \
  --listings \
  -o documentation-pandoc.pdf
```

Spent a few minutes looking at the filters, but realized it might be a bit involved, so set this option aside to come back to after seeing if there may be more easier options.

https://pandoc.org/filters.html

https://lee-phillips.org/panflute-gnuplot/

https://pandoc.org/lua-filters.html

https://tewarid.github.io/2018/12/28/pandoc-lua-filter-to-convert-math-block-type.html

## Solution Option 4: Replace Offending Argument with sed

sed could be easily used to just look through whole file, replace offending code, and plumb the output into Pandoc.
This way was very fast to understand and implement, more elegant than the HTML/CSS hacking above, and seemed somewhat flexible.
But in order to ensure I could specify which parts of code I actually wanted to remove (and not occurences in the text, like `{linenos=false}`) I talked on a `&nbsp` at the end.
Then can run sed and then Pandoc as follows.

```bash
$ sed 's/ {linenos=false}&nbsp;//g' documentation.md > documentation-out.md; \
  pandoc documentation-out.md \
  --variable fontsize=10pt \
  --number-sections \
  --from markdown \
  --template eisvogel \
  --pdf-engine xelatex \
  --listings \
  -o documentation.pdf; rm documentation-out.md
```

The result is the following, no line numbers on the web, and correctly rendered PDF output from Pandoc.

````md
```js {linenos=false} &nbsp;
var test
```
````

## Solution Options Recap

Something like using a filter probably the best.
Not sure how fast compared to sed.
Especially with the way I have now with file writes.

# Conclusions

Between this and the website setup, do a few things.

* Links as HTML with attribute `target="_blank"`.
* When the link text is a valid hyperlink itself, break it up inside with an empty `<span></span>`.
* Use MathJax to render Latex.
* When omitting line numbers in Hugo, follow the argument with a character such as a non-breaking space: `{linenos=false} &nbsp;`.
* Use sed before Pandoc to remove the argument Pandoc doesn't understand.

https://gist.github.com/nhtranngoc/88b72d9bfb656a3de227eea38ed80627

https://learnbyexample.github.io/tutorial/ebook-generation/customizing-pandoc/

> GitHub style markdown is recommended if you wish to use the same source (or with minor changes) in multiple places.

https://github.com/supermarin/YosemiteSanFranciscoFont

https://tex.stackexchange.com/questions/25249/how-do-i-use-a-particular-font-for-a-small-section-of-text-in-my-document

https://stackoverflow.com/questions/39220389/embed-indented-html-in-markdown-with-pandoc

https://gist.github.com/dixonsiu/28c473f93722e586e6d53b035923967c

THIS FINALLY HAD A NICE LUA FILTER:

https://stackoverflow.com/questions/52958312/html-formatted-hyperlinks-not-preserved-in-bookdown-pdf

<a href="http://google.com" target="_blank">Google Link</a>

Obviously can't use relative links when generating the PDF, they'll be broken:
<a href="../../mit-notes.pdf" target="_blank">grad school notes</a>
Use this instead:
<a href="https://danielwiese.com/mit-notes.pdf" target="_blank">grad school notes</a>
