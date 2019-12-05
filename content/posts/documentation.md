---
title: "Documentation"
date: 2019-12-03
draft: true
toc: false
images:
tags:
  - untagged
lang: "en"
geometry:
- margin=1in
urlcolor: "cyan"
fontsize: "10pt"
papersize: "letter"
---

<!--
$ pandoc documentation.md \
  --number-sections \
  --from=markdown-markdown_in_html_blocks-native_spans \
  --template eisvogel \
  --pdf-engine xelatex \
  --listings \
  -o documentation.pdf
-->

Prior to creating this site, I'd always been quite good about documenting the things I've worked on.
While in grad school this was primarily in Latex, documenting hundreds of pages of in my <a href="../../mit-notes.pdf" target="_blank">grad school notes</a>.
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
  - Web via a static site generator like Hugo, used to <a href="../site-setup/" target="_blank">create this site</a>
  - PDF
- **Consistent styling** across the above platforms.
  For example, generating a PDF can be easily styled to look like a Github wiki.
- **Minimal tooling** required.
- **No dedicated app** required.
- Fast to compile or view
- Features/nice to have
  - Inclusion of references from .bib file?
  - Good syntax highlighting of the mixed (markdown with latex, html, etc.) in editor (e.g. sublime)

# Solution

For a simple, easy-to-read syntax, markdown was a good choice. Furthermore, it is widely supported on and offline, with easy tooling to generate PDFs such as the <a href="https://atom.io/packages/markdown-pdf" target="_blank">markdown-pdf</a> and <a href="https://atom.io/packages/language-markdown" target="_blank">language-markdown</a> packages for Atom and <a href="https://pandoc.org/" target="_blank">Pandoc</a>; good support online at Github, Gitlab, <a href="https://jekyllrb.com/" target="_blank">Jekyll</a>, and more.

For the generation of PDFs, Pandoc is easy to use, and supports Latex. Latex is not supported in Github, but seems to be in Gitlab.
Markdown is easy-to-read, reducing the need for real-time rendering, or a need to frequently and quickly compile.
References can be included from a .bib file.
Using the Sublime Text markdown syntax highlighting works well for markdown, although it does not do anything for Latex.
But as the primary goal was for a simple, easy syntax that *allowed* for the inclusing of equations rather than a focus on highly mathematical documents, this was not a big deal - equations should be relatively simple and infrequent.

Latex was an alternate source format considered, with Pandoc providing tools for conversion to markdown or html for use on the web, and easy generation of PDFs.
However, even with templates, Latex source is more verbose and and cumbersome to use for relatively simple note taking and documentation.
Jupyter was considered as well, but seemed much more heavyweight than desired, required an app, and is not as widely or easily supported as markdown, although it is supported in Github, for example.

## Pandoc

```bash
# Basic command for generating test.pdf from test.md
$ pandoc test.md -o test.pdf

# Use Github flavored markdown
$ pandoc test.md -f gfm -o test-gfm.pdf

# Include bibliography file test.bib
$ pandoc test.md --filter pandoc-citeproc --bibliography=test.bib -o test.pdf
```

### Styling

To apply style more closely resembling Github markdown, <a href="https://github.com/jgm/pandoc/wiki/User-contributed-templates" target="_blank">several options</a> were available. The <a href="https://github.com/Wandmalfarbe/pandoc-latex-template" target="_blank">Eisvogel</a> Pandoc Latex template was one of the simplest and easiest. Default pandoc template location: `~/.pandoc/templates/`. This template can be easily applied with Pandoc. The result is pretty good enough.

### Bibliography

The bibliography can be easily be included. To format the references, the <a href="https://citationstyles.org/" target="_blank">Citation Style Language</a> can be specified. Thousands of CSL files can be found <a href="https://github.com/citation-style-language/styles" target="_blank">here</a>.

### Command

```bash
$ pandoc test.md \
    --variable urlcolor=cyan \
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

with YAML:

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

## Markdown Processor of Pandoc versus Hugo

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
However, the PDF generated by Pandoc looks very bad.
The code is formatted as in-line code, and all of the statements are on a single line.
The reason for that is that Pandoc cannot intepret the `{lineos=false}`.
So we need a way to be able to specify for Pandoc to ignore this argument.
The result of the solution to this problem is the following:

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

Now this looks correct on the web via Hugo, and in the PDF output generated by Pandoc.
Of course, they will be different slightly in that the code block on the web will not have line numbering due to the `{linenos=false}`, but this will not be the case with Pandoc.

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
And what we are doing here is using the Pandoc option `--from=markdown-markdown_in_html_blocks-native_spans` to show the HTML `span` with `class="hide-me"`.
This `span` will show in the PDF generated by Pandoc, but on the web, CSS `display: none;` is applied to this class, hiding it.
A `p` element will naturally show in Hugo, but is hidden from Pandoc.
So basically via some cumbersome HTML and CSS, I can define parts that show up either in the Pandoc PDF output, or on the web, and thus can use each of these parts differently depending on how each markdown processor will use them.

This is at best horribly inelegant, requiring extra HTML elements and massive amounts of duplicated code ust because the markdown processor of Pandoc is different than that of Hugo.
I initially thought this would be a perfect use case for a Shortcode and then realized another problem: the Shortcodes I'd used already elsewhere could not be interpreted by Pandoc either.

Not to mention that until now, the `eisvogel.latex` style that I use with Pandoc is very different than the way this site is styled.
Reconciling these two via a new `.latex` style would be a huge pain, and require for any style changes be kept in sync between this file and the CSS used for this site.

Maybe there is a way to generate a PDF from Hugo?
The easiest way to do this is with the site open in a web browser, just print the page.
Honestly the output looks pretty good.
Really the only downsides I noticed (when printing from Chrome at least) was that there is some website navigation items present in the printed output, and that when including the headers and footers, they are not styled nicely at all - just some plain Times New Roman text.
