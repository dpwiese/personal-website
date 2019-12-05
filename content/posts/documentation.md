---
title: "Documentation"
date: 2019-12-04T21:32:04-05:00
draft: true
toc: false
images:
tags:
  - untagged
---

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
