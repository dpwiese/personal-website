---
title: "Client-side Syntax Highlighting with Highlight.js and Hugo"
date: 2020-08-23T10:00:00-04:00
draft: true
toc: false
# ADD OG IMAGE WITH THIS POST:
images: ["img/posts/data-conversions/og-image.png"]
tags: 
  - hugo
  - programming
  - javascript
keywords: [hugo, programming, javascript]
---

<link rel="stylesheet" href="/css/pygments.css" type="text/css">

<script src="/js/highlight.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/highlightjs-line-numbers.js@2.8.0/dist/highlightjs-line-numbers.min.js"></script>

# Introduction

Most of the posts on this site contain at least a fair bit of code, in the form of blocks with syntax highlighting provided by Hugo's syntax highlighter [Chroma](https://github.com/alecthomas/chroma).
The Hugo docs on [Syntax Highlighting](https://gohugo.io/content-management/syntax-highlighting/) have more information.
This approach has worked great so far, where code is either written directly in the Markdown source, or included with a shortcode as described in the post [Single Page CRC Calculator with TypeScript](/posts/typescript-crc-calculator/#displaying-source-file-in-hugo-code-block).
However, I've found myself occasionally wanting to include code from external sources, e.g. Github, leading to this post.

As Hugo generates static HTML when built and syntax highlighting is provided only by wrapping code in HTML tags with various classes that are then appropriately styled, two paths quickly became clear.
In the first, I could modify the current build tools to fetch whichever external code files I wished to include and save them to a directory accessible to Hugo at build time and continue using Chroma.
In the second, I could fetch the desired when loading the page on the client side, and use a client-side syntax highlighter.
I could have embedded the code using a Gist, but that of course didn't solve the problem either.

The first approach would have been quite easy to do had I been willing to specify the files that I wished to include in a place other than where they were used - some kind of manifest perhaps.
But this defeated the point of what I was looking to achieve, which was simply copy a link to a particular file and use it in a post.
Making tooling that would search through posts for such external links and fetch and save their contents at build time sounded like more trouble than it was worth.
Thus I opted for the second approach.

# Using highlight.js

The documentation for [highlight.js](https://highlightjs.org) was easy enough to follow, and in moments I'd written the few lines of JavaScript needed to fetch the external source code file and apply syntax highlighting.
All that was left was a bit of styling to achieve the same look as the code included with Chroma.

The first issue was the lack of line numbers.
I expected this to be something easy to configure and found in [the docs](https://highlightjs.readthedocs.io/en/latest/line-numbers.html) that the lack of line numbers was a feature:

> Many highlighters, in my opinion, are overdoing it with such things as separate colors for every single type of lexemes, striped backgrounds, fancy buttons around code blocks and — yes — line numbers.
> The more fancy stuff resides around the code the more it distracts a reader from understanding it.
> ... this new feature will not just make highlight.js better, it might actually make it worse simply by making it look more bloated in blog posts around the Internet

While line numbers may not be critical for generally short blocks included in blog posts, I disagree with the author's opinion above that giving users the option to provide them is a net negative.
In any case, this posed little problem as the [highlightjs-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js/) plugin made it trivial to add them.

With line numbers in place, I was essentially done except for styling the code block to match the code blocks styled by Chroma.

```js
<script>
fetchAsync("https://danielwiese.com/crc-calculator/crc-utils.js");

async function fetchAsync (url) {
  let response = await fetch(url);
  let data = await response.text();

  document.getElementById("external-code").innerHTML = data

  document.querySelectorAll('pre code.external').forEach((block) => {
    hljs.highlightBlock(block);
    hljs.lineNumbersBlock(block);
  });
}
</script>
```

and

```html
<div class="highlight">
<pre class="chroma external">
<code class="js external" id="external-code">
loading...
</code>
</pre>
</div>
```

Source code from a particluar branch [https://raw.githubusercontent.com/dpwiese/crc-calculator/master/src/crc-utils.ts](https://raw.githubusercontent.com/dpwiese/crc-calculator/master/src/crc-utils.ts) or even a particular commit 
[https://raw.githubusercontent.com/dpwiese/crc-calculator/4df95cadac03bf402b58e661767d5a9c365926ce/src/crc-utils.ts](https://raw.githubusercontent.com/dpwiese/crc-calculator/4df95cadac03bf402b58e661767d5a9c365926ce/src/crc-utils.ts) could then be included as desired.

# Styling

I expected styling to be very easy, but quickly realized many of the components that were given classes from Chroma were not given similar classes by highlight.js, making it difficult to style consistently.
For example, characters such as `+`, `?`, `&`, and others were set into a span with `class="o"` by Chroma, presumabely where the `o` indicated operators.
This was not the case with highlight.js, making consistent styling impossible.
While I could have just as easily adjusted the style used with Chroma, this would have resulted in many of the elements having the same color, which was not desirable.
I generally liked the scheme provided by Chroma.

So I had to fork highlight.js and make some small adjustments for the language(s) that I wanted to use.
This was a bit harder than I expected and I only spent a few moments to make the larger changes.
Another option I considered was using JavaScript's `.replace` in the code I was using to fetch and apply highlight.js styling.

```js {linenos=false}
.replace(/\=/g,'<span class="o">=</span>')
.replace(/\,/g,'<span class="p">,</span>');
```

# Result

## Code Block with Shortcode and Chroma Highlighting

{{% code file="/static/crc-calculator/crc-utils.js" language="js" %}}

## Code Block from External Source and highlight.js Highlighting

<div class="highlight">
<pre class="chroma external">
<code class="js external" id="crc-utils.js">
loading...
</code>
</pre>
</div>

<script>
fetchAsync("https://danielwiese.com/crc-calculator/crc-utils.js");

async function fetchAsync (url) {
  let response = await fetch(url);
  let data = await response.text();

  document.getElementById("crc-utils.js").innerHTML = data

  document.querySelectorAll('pre code.external').forEach((block) => {
    hljs.highlightBlock(block);
    hljs.lineNumbersBlock(block);
  });
}
</script>

