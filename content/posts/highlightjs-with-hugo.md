---
title: "Client-Side Syntax Highlighting with Highlight.js and Hugo"
date: 2020-08-23T10:00:00-04:00
draft: false
toc: false
images: ["img/posts/highlightjs-with-hugo/og-image.png"]
tags:
  - hugo
  - programming
  - javascript
keywords: [hugo, programming, javascript]
js: [/js/highlight.min.js, https://cdn.jsdelivr.net/npm/highlightjs-line-numbers.js@2.8.0/dist/highlightjs-line-numbers.min.js]
css: [/css/pygments.css]
description: "This post describes how to apply client-side syntax highlighting using highlight.js for content that is fetched on page load."
---

<!-- Need to include stylesheet here to load after that for highlightjs-line-numbers -->
<link rel="stylesheet" href="/css/pygments.css">

# Introduction

Most of the posts on this site contain at least a fair bit of code, in the form of blocks with syntax highlighting provided by Hugo's syntax highlighter [Chroma](https://github.com/alecthomas/chroma).
The Hugo docs on [Syntax Highlighting](https://gohugo.io/content-management/syntax-highlighting/) have more information.
This approach has worked great so far, where code is either written directly in the Markdown source, or included with a shortcode as described in the post [Single Page CRC Calculator with TypeScript](/posts/typescript-crc-calculator/#displaying-source-file-in-hugo-code-block).
However, I've found myself occasionally wanting to include code hosted externally, such as on Github, leading to this post.

As Hugo generates static HTML when built and syntax highlighting is provided by wrapping code in HTML tags with various classes that are then appropriately styled, two paths quickly became clear.
In the first, I could modify the current build process to fetch whichever external code files I wished to include and save them to a directory accessible to Hugo at build time and continue using Chroma.
In the second, I could fetch the desired code when loading the page and use a client-side syntax highlighter.
I considered embedded the code using a Gist, but that of course didn't solve the problem either.

The first approach would have been quite easy to do had I been willing to specify the files that I wished to include in a place other than where they were used - some kind of manifest perhaps.
But this defeated the point of what I was looking to achieve, which was to simply use a link to a particular file in a post and expect syntax highlighting to be applied.
Making tooling that would search through posts for such external links and fetch and save their contents at build time sounded like more trouble than it was worth.
Thus I opted for the second approach.

# Using highlight.js

The documentation for [highlight.js](https://highlightjs.org) was easy enough to follow, and in moments I'd written the few lines of JavaScript needed to fetch the external source code and apply syntax highlighting.
All that was left was a bit of styling to achieve the same look as the code I had included and styled with Chroma.

The first issue was the lack of line numbers with highlight.js.
I expected this to be something easy to configure and found in [the docs](https://highlightjs.readthedocs.io/en/latest/line-numbers.html) that the lack of line numbers was _actually_ a feature:

> Many highlighters, in my opinion, are overdoing it with such things as separate colors for every single type of lexemes, striped backgrounds, fancy buttons around code blocks and — yes — line numbers.
> The more fancy stuff resides around the code the more it distracts a reader from understanding it.
> ... this new feature will not just make highlight.js better, it might actually make it worse simply by making it look more bloated in blog posts around the Internet

While line numbers may not be critical for generally short blocks included in blog posts, I disagree with the author's opinion above that giving users the option to provide them is a net negative and thus shouldn't be supported.
In any case, this posed little problem as the [highlightjs-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js/) plugin made it trivial to add them.

With line numbers in place, only a bit more styling of the code block to match the code blocks styled by Chroma - mostly colors.
Everything that was done so far has been only:
1. Import the JavaScript for highlight.js and highlightjs-line-numbers.js and CSS
2. Include the div where the code block is to go
3. Fetch the code from whatever external source, set the div contents and apply the suntax highlighting.

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/highlight.js@9.12.0/styles/monokai.css" type="text/css">

<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/10.1.2/highlight.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/highlightjs-line-numbers.js@2.8.0/dist/highlightjs-line-numbers.min.js"></script>

<div class="highlight">
<pre class="chroma external">
<code class="js external" id="external-code">
loading...
</code>
</pre>
</div>

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

I picked a short simple URL in the example above, but when including source code from Github, it can be included from a particular branch such as [https://raw.githubusercontent.com/dpwiese/crc-calculator/master/src/crc-utils.ts](https://raw.githubusercontent.com/dpwiese/crc-calculator/master/src/crc-utils.ts) or a particular commit 
[https://raw.githubusercontent.com/dpwiese/crc-calculator/4df95cadac03bf402b58e661767d5a9c365926ce/src/crc-utils.ts](https://raw.githubusercontent.com/dpwiese/crc-calculator/4df95cadac03bf402b58e661767d5a9c365926ce/src/crc-utils.ts) both which I expect may be useful in the future.

# Styling

I expected styling to be very easy, but quickly realized many of the elements in a code block that were given classes from Chroma were _not_ given similar classes by highlight.js, making it difficult to style consistently.
For example, characters such as `+`, `?`, `&`, and others were set into a span with `class="o"` by Chroma, presumably where the `o` indicated operator.
This was not the case with highlight.js, making consistent styling impossible without changing the highlight.js parser, or wrapping them in markup as a post-processing step after highlight.js had already been run.
While I could have just as easily adjusted the style I used with Chroma to match what highlight.js was capable of, this would have resulted in many of the elements having the same color, which was not desirable.
I generally liked the scheme provided by Chroma and that so many elements were uniquely identified giving lots of flexibility.

So I had to [fork highlight.js](https://github.com/dpwiese/highlight.js/tree/match-chroma) and make some [small adjustments](https://github.com/dpwiese/highlight.js/commit/21f5cc298b3ae0608066691643f4d3978ca65223) for the language(s) that I wanted to use.
Understanding the parsing rules was a bit harder than I'd expected, but it was easy enough to make some basic changes and build a new package from the modified source with `node tools/build.js :common`.

Another option I considered as an alternative was using JavaScript's `.replace` in the code I was using to fetch and apply highlight.js styling as a very crude post-processing step.

```js {linenos=false}
// For example
.replace(/\=/g,'<span class="o">=</span>')
.replace(/\,/g,'<span class="p">,</span>');
```

# Sample Result

The results are below, and the second version highlighted with highlight.js matches quite well to the first highlighted with Chroma.
There are a few issues in this particular example:
1. Parentheses and curly brace in function definition
2. Regular expressions
3. Built-in function `parseInt` (see: chroma [types.go](https://github.com/alecthomas/chroma/blob/master/types.go))

Overall, these are minor issues that would be addressed without too much difficulty by learning highlight.js a bit better and making some additional modifications, but that'll be a project for another day.

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
