---
title: "Setting up this Site"
date: 2019-12-06
draft: false
toc: false
images: ["img/posts/site-setup/og-image.png"]
tags:
  - hugo
  - programming
  - latex
keywords: [hugo, programming, latex]
description: "This post describes configuring a Hugo site on Github pages with LaTeX rendered with MathJax."
---

# Introduction

Personal websites may have many uses, and at the time of writing, the uses for this site are not yet fully clear.
However, **the two primary motivations for creating this site were:**

1. Create a place to document and share my learnings
2. Learn some new tooling to support motivation 1.

A tertiary motivation was to provide a place for people to learn more about me than might be possible via the information available on social media.

With this in mind, I began to think about the form that the items of motivation 1. would take, whether it be blog posts, standalone documents, etc. and thus which tooling would best support this.
The outcome of this was that the majority of the items I expected to create for motivation 1. would be single page posts, and that the best tooling would be in the form of a static site generator.

The reasons for using a static site generator based on current and anticipated needs are as follows:

1. I want something fast and simple to setup, edit, host, and maintain.
2. I don't need a content management system such as WordPress to facilitate making changes to the site, nor does anyone else - I will be the sole content creator and manager of the site.
3. I don't need any server-side functionality.
4. I don't need access to the extent of plugins offered by a CMS like WordPress, such as those to facilitate SEO, marketing automation, or otherwise.

No tooling was precluded due to a lack of programming ability as may be the case for a non-technical blogger.

Speed and simplicity meant I did not want a solution with an involved build process, or that required me to manage a database, or install and update plugins.
The changes introduced when updating plugins are not always fully known, and are often necessary to provide security to the site.
I wanted a solution in which content creation was done in markdown as much of my existing documentation is already written in markdown and it is well suited to the type of writing I do.
This all meant that solutions with content management systems, such as WordPress, or building a web application like React from scratch would not be best suited to my needs.

The benefits of static site generators and comparisons to CMS are well documented online and align well with my needs.
All the particular SSGs I considered were very fast to setup, edit, and host.
With just a few simple commands I could install an SSG, create and populate a new page with content, apply a theme, and host.
It was a matter of a few minutes to get a basic version of this site hosted online.
I didn't need to provision a database or sign up for some new hosting provider.
This site is currently hosted using Github Pages, but it just as easily could have been dropped into my existing S3 bucket.

# Picking a Static Site Generator

Once I had decided that a static site generator was the best choice for my needs, I had to pick one.
There are quite a few popular options such as [Gatsby](https://www.gatsbyjs.org/), [Jekyll](https://jekyllrb.com/), [Hugo](https://gohugo.io/), and [Pelican](https://getpelican.com/) to name a few, and many comparisons between these and others can be found online.
When evaluating these and other options almost all of them were at least reasonable choices.
However, I didn't need the capabilities of React or want the associated complexity of using it and GraphQL which turned me away from Gatsby.
Jekyll was a strong consideration.
I ultimately went with Hugo for two primary reasons: benchmarks indicated it to be significantly faster than most alternatives (very important for me and my 8 year old MacBook!) and I wanted something built on Go, as opposed to Ruby, Python, or JavaScript.

# Setting up Hugo

This section will provide a brief overview of setting up Hugo, and my process and workflow for creating content and deploying the site.
The best place to start is the [Hugo Docs](https://gohugo.io/documentation/).
Themes can be found at: [https://themes.gohugo.io/](https://themes.gohugo.io/).

For this site I chose the [hermit](https://themes.gohugo.io/hermit/) theme.
With the following few commands the site was setup and I was ready to start writing content.

```bash
# Install Hugo
$ brew install hugo

# Create a new site
$ hugo new site my-website

# Install a theme
$ cd my-website
$ git submodule add https://github.com/Track3/hermit.git themes/hermit

# Edit config, setting author, title, theme, etc.
$ vi config.toml

# Create my first post
$ hugo new posts/site-setup.md
```

The site can then be viewed locally while editing by running

```bash {linenos=false}&nbsp;
$ hugo server -D
```

and when you are ready to build the `public` folder for deployment simply type

```bash {linenos=false}&nbsp;
$ hugo
```

## Hosting

The choice to host on Github Pages impacted this portion of the setup.
Github Pages automatically generates Jekyll sites, but hosting Hugo sites on Github Pages simply requires Hugo's output (the `public` folder) be plumbed to the Github Pages repository, which is easily accomplished using submodules.
In this case, my Github Pages user repository is `dpwiese.github.io`.
The site generated above should be commited to a different repository, for example `personal-website`.
Then, the `public` folder should be made a submodule from the `dpwiese.github.io` repository using the following command.

```bash {linenos=false}&nbsp;
$ git submodule add -b master git@github.com:dpwiese/dpwiese.github.io.git public
```

This is described more completely in the Hugo docs: [Host on Github](https://gohugo.io/hosting-and-deployment/hosting-on-github/).
Note the following in [About GitHub Pages](https://help.github.com/en/github/working-with-github-pages/about-github-pages#user--organization-pages) that says, for a user or organization page, it must be hosted from the `master` branch, hence the use of `-b master` in the command above.

> The default publishing source for user and organization sites is the master branch.
If the repository for your user or organization site has a master branch, your site will publish automatically from that branch.
You cannot choose a different publishing source for user or organization sites.

Note as well that Github Pages are subject to the limitations described under their [Guidelines for using GitHub Pages](https://help.github.com/en/github/working-with-github-pages/about-github-pages#guidelines-for-using-github-pages).
These limitations should not be a limitation for the purposes of this site.
In any case, another natural alternative was S3, to which this site could be easily deployed by appropriate configuration of `config.toml` as described in the [Hugo Deploy](https://gohugo.io/hosting-and-deployment/hugo-deploy/) docs.

## Workflow

With the Hugo site generated and now configured appropriately, with a first page of content and theme applied, the following workflow can be adopted.
A new branch can be created within the `personal-website` superproject repository to support whatever site changes are currently being made.
The `hugo server` command can be used to monitor the changes to the site locally as they are being made without generating output to the submodule.
Commits to the superproject repository can be made as needed.
When the site is ready to deploy, simply use the `hugo` command to generate the latest contents in the `public` submodule, and then commit the submodule as follows:

```bash
$ cd public
$ git add .
$ git commit -am "comment"
$ git push origin master
```

*Thus, every commit to the submodule `master` branch is a deployment.*

After running `hugo` and the changing the submodule, the superproject then identifies the submodule as dirty until it is committed.
To discard these changes just `git clean -f` or `git checkout -- .` as necessary.

## Styling

With the above simple workflow used to deploy this site, I then wanted to make some minor adjustments to the theme.
This can be accomplished by duplicating theme files from, for example, `./themes/hermit/layouts/index.html` to `./layouts/index.html` and then editing as needed to adjust certain elements.
Any such files in the equivalent path within the main project directory will overwrite those from the theme.
This is true as well for assets in `./themes/hermit/assets/scss/_predefined.scss` by `./assets/scss/_predefined.scss` for example.
Additionally, custom CSS can be applied from a file such as `./static/css/style.css` by specifying in the `config.toml` file:

```toml
[params]
  customCSS = ["css/style.css"]
```

With these options, I wanted to be able to modify the theme in the least obtrusive way.
Overwriting layouts seemed like a bad idea - I'd be duplicating a substantial amount of code to make minor modifications, and the modified code (a duplicated SCSS file) may and likely would not even apply should I change the theme later.
Because of this, I opted to import a small and simple CSS file via `config.toml` and only change the few necessary elements to achieve the desired appearance.

# Hyperlinks in Hugo

When writing this post, one of the first things I wanted to do is create links that would open in a new tab.
The seemingly most straightforward way to do this, and something that is often useful in markdown anyway, is to use HTML.
However, when inspecting the generated HTML when doing this, the following comment appeared where my embedded HTML would have been:

```html {linenos=false}&nbsp;
<!-- raw HTML omitted -->
```

After a bit of Googling and reading through the docs, I learned that with the latest version of Hugo v0.60 that was released just days before having installed it, the following addition to `config.toml` was needed.
Per the [v0.60 release notes](https://gohugo.io/news/0.60.0-relnotes/) and also in the [Configure Markup](https://gohugo.io/getting-started/configuration-markup/#goldmark) docs, the following statement is made regarding inline HTML:

> By default, Goldmark does not render raw HTMLs and potentially dangerous links.
If you have lots of inline HTML in your Markdown files, you may have to enable the `unsafe` mode:

```toml
[markup.goldmark.renderer]
  unsafe = true
```

When using another markdown engine, such as Mmark, this was not necessary.
But as long as Goldmark is used this setting is needed to use HTML.

Another difficulty that was realized after adding HTML links such as the one below, was that when generating the static site, Hugo would interpret the text as a link and thus generate a standard `a` tag without the additional `target` attribute.

```html {linenos=false}&nbsp;
<a href="https://gohugo.io/" target="_blank">https://gohugo.io/</a>
```

I considered two solutions to this problem.

## Shortcodes

A relatively easy and simple way around this was using [shortcodes](https://gohugo.io/content-management/shortcodes/).
More information can be found in the Hugo Docs [Create Your Own Shortcodes](https://gohugo.io/templates/shortcode-templates).
The following shortcode file was created in `./layouts/shortcodes/plainlink.html`.

```html {linenos=false}&nbsp;
<a href={{ index .Params 0 | safeHTML }} target="_blank">{{ index .Params 0 | safeHTML }}</a>
```

This provides a shortcode that can be called by name, render the contents using the specified template and the passed parameters.
In this case, the `plainlink` shortcode would be called and passed a url from within a posts markdown file as shown below.

```md {linenos=false}&nbsp;
{{</* plainlink "https://themes.gohugo.io/" */>}}
```

The result, when the site was generated, was a correctly generated `a` tag that preserved the `target` attribute, ensuring the clicked link would be opened in a new window.

## Superfluous Span Tag

A downside to the above method is that the shortcode now contained in the markdown can only be interpreted by Hugo via one of its supported markdown engines.
This means that if any portion of this markdown source containing shortcode is ported anywhere else (e.g. Github wiki) I'll have to live with awkward shortcode in the displayed output, or manually fix the source to not require shortcode.

An alternative method to write HTML links in a way that was more portable was desired.
Fortunately I found the following suggestion in this [Stack Overflow answer](https://stackoverflow.com/a/53462722): simply insert an empty div in the URL text so it is not interpreted as a URL:

```html {linenos=false}&nbsp;
<a href="https://gohugo.io/" target="_blank">htt<span></span>ps://gohugo.io/</a>
```

This solution, while not elegant, does get the job done for the few cases where it's needed.
It's also worth noting that Github does not include the target attribute, so in the event this markdown source is ported to Github, the link will not open in a new window.

## Update v0.62

When this post was first written, I had the most recent version of Hugo at the time: v0.60.
The latest version at the time of this update is v0.74.3.
In v0.62.0 a new feature was added which offers a much more elegant solution to the hyperlink problem desribed above: [Markdown Render Hooks](https://gohugo.io/getting-started/configuration-markup/#markdown-render-hooks).
This is well described in the docs, but in short:

> Render Hooks allow custom templates to override markdown rendering functionality.

Thus, with a simple template `./layouts/_default/_markup/render-link.html` with the following contents, standard [external markdown links](https://gohugo.io/) are made to open in a new tab while the behavior of [internal links](/posts) is unaffected.

```go
<a href="{{ .Destination | safeURL }}"{{ with .Title}} title="{{ . }}"{{ end }}{{ if strings.HasPrefix .Destination "http" }} target="_blank"{{ end }}>{{ .Text }}</a>
```

# Latex

Next I wanted a configuration that would enable Latex to be rendered on the page.
I wanted to use [KaTeX](https://katex.org/) as it is known to be faster and lighter weight than the obvious alternative, [MathJax](https://www.mathjax.org/).
I found some nice information online, such as the blog post [Render LaTeX with KaTex in Hugo Blog](https://eankeen.github.io/blog/posts/render-latex-with-katex-in-hugo-blog/).
However, due to issues with the markdown processors Goldmark and Blackfriday used by Hugo, an alternative markdown processor was needed.
The blog post above uses Mmark, which at the time of writing is listed in the Hugo [List of content formats](https://gohugo.io/content-management/formats/#list-of-content-formats) as being an alternative, with the comment:

> Mmark is deprecated and will be removed in a future release.

In fact, it seemed to have been just deprecated when this post was written per the [v0.60 release notes](https://gohugo.io/news/0.60.0-relnotes/).
I looked at other alternatives, such as [kramdown](https://kramdown.gettalong.org/index.html), described in the blog post [Hugo meets kramdown + KaTeX](https://takuti.me/note/hugo-kramdown-and-katex/).
But the best options seemed to be either use Mmark until it is removed or Goldmark supports KaTex, or use MathJax, as described in the blog post [Setting MathJax with Hugo](https://divadnojnarg.github.io/blog/mathjax/).

While KaTeX does seem to be the preferred Latex interpreter today, MathJax also seems to have been long used and well liked.
Since neither option seemed to be obviously better than the other I decided to do a quick look for any practical differences between these two Latex interpreters, primarily in their ability to render some of the complex equations I've written before and may write again.

Overall both seemed more-or-less equivalent in their ability to render the equations I provided.
I did notice, however, that upon finally trying to use the `uuline` function from the `ulem` package to double underline a character, that MathJax would properly display the equation with the unknown function displayed in red plain text while KaTeX would display the entire Latex expression in plain text.
For reference, here is a list of [additional functions from other LaTeX packages that are emulated by KaTeX](https://github.com/KaTeX/KaTeX/wiki/Package-Emulation).

I decided that while the way MathJax handled the unknown function was preferred, it ultimately made no difference in my using MathJax over KaTeX - I'd still have to find alternative Latex functions/characters that would properly render the entirety of whatever equation I was writing.
In the case of `\uuline{*}` this was as simple as `\underline{\underline{*}}`.

## KaTeX

I tried KaTeX with Mmark first.
The KaTeX integration was accomplished exactly as described in the blog post above, shown below for convenience.
The KaTeX CSS and JavaScript was imported in a footer partial, in this case `extra-foot.html`.
Here is the code from [Render LaTeX with KaTex in Hugo Blog](https://eankeen.github.io/blog/posts/render-latex-with-katex-in-hugo-blog/):

```html
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/katex@0.10.0-rc.1/dist/katex.min.css"
  integrity="sha384-D+9gmBxUQogRLqvARvNLmA9hS2x//eK1FhVb9PiU86gmcrBrJAQT8okdJ4LMp2uv"
  crossorigin="anonymous"
>

<script defer
  src="https://cdn.jsdelivr.net/npm/katex@0.10.0-rc.1/dist/katex.min.js"
  integrity="sha384-483A6DwYfKeDa0Q52fJmxFXkcPCFfnXMoXblOkJ4JcA8zATN6Tm78UNL72AKk+0O"
  crossorigin="anonymous"
></script>

<script defer
  src="https://cdn.jsdelivr.net/npm/katex@0.10.0-rc.1/dist/contrib/auto-render.min.js"
  integrity="sha384-yACMu8JWxKzSp/C1YV86pzGiQ/l1YUfE8oPuahJQxzehAjEt2GiQuy/BIvl9KyeF"
  crossorigin="anonymous"
  onload="renderMathInElement(document.body);"
></script>
```

The following lines were added to the posts YAML metadata header to tell Hugo to use Mmark as the markdown processor instead of the default.

```yaml
---
katex: true
markup: "mmark"
---
```

The following Latex now renders beautifully with Katex.

```tex
$$
% This works as expected in KaTeX
\begin{aligned}
a &= 1 \\
b &= 2
\end{aligned}
$$
```

Overall the KaTeX integration worked great, but I realized I did not want to be forced to use Mmark.
This came up in one particular instance in attempting to change line numbering from the global default as shown below:

````md
```js {linenos=false}
$ var your = "code here"
```
````

Mmark was not able to handle this.
While this may be a small inconvenience now, I do expect to want to utilize such features in the future.
Furthermore, Mmark will be deprecated, so I'd rather not be dependent on that.

## MathJax

As with Katex, for MathJax the JavaScript was imported in a footer partial, in this case `extra-foot.html`.
Here is the code from [Setting MathJax with Hugo](https://divadnojnarg.github.io/blog/mathjax/):

```html
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
  MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\\(','\\)']],
    displayMath: [['$$','$$']],
    processEscapes: true,
    processEnvironments: true,
    skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
    TeX: { equationNumbers: { autoNumber: "AMS" },
         extensions: ["AMSmath.js", "AMSsymbols.js"] }
  }
  });
  MathJax.Hub.Queue(function() {
    // Fix <code> tags after MathJax finishes running. This is a
    // hack to overcome a shortcoming of Markdown. Discussion at
    // https://github.com/mojombo/jekyll/issues/199
    var all = MathJax.Hub.getAllJax(), i;
    for(i = 0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
  });

  MathJax.Hub.Config({
  // Autonumbering by mathjax
  TeX: { equationNumbers: { autoNumber: "AMS" } }
  });
</script>
```

One difference between Katex was the need to use additional backslash characters to create a new line.

```tex
$$
% Extra backslashes used to make newline with MathJax
\begin{aligned}
a &= 1 \\\\
b &= 2
\end{aligned}
$$
```

Alternatively `\newline` could be used, although this cannot be interpreted normally in Latex math mode which would create difficulties when using these portions of this markdown source elsewhere.

```tex
$$
% \newline with MathJax
\begin{aligned}
a &= 1 \newline
b &= 2
\end{aligned}
$$
```

This is a bit annoying, but overall a minor inconvenience.

## Results

Ultimately I decided to stick with Hugo's default markdown processor and use MathJax.
The results of the MathJax integration are shown below.
First is an underbraced integral expression of mass conservation:

<p>
$$
\underbrace{\frac{\partial}{\partial t}\int_{V}\rho dV}_{\text{Rate of change of mass}}
=\underbrace{-\oint_{S}\rho\underline{v}\cdot\underline{n}dS}_{\text{Net inflow of mass}}
$$
</p>

```tex
\underbrace{\frac{\partial}{\partial t}\int_{V}\rho dV}_{\text{Rate of change of mass}}
=\underbrace{-\oint_{S}\rho\underline{v}\cdot\underline{n}dS}_{\text{Net inflow of mass}}
```

And next are the Incompressible [Navier-Stokes equations](https://en.wikipedia.org/wiki/Navier%E2%80%93Stokes_equations) in an `aligned` environment:

$$
\begin{aligned}
\rho\left(\frac{\partial\underline{v}}{\partial t}+\underline{v}\cdot\underline{\nabla}\underline{v}\right)
& =-\underline{\nabla}p+\mu\nabla^{2}\underline{v}+\rho\underline{g} \\\\
\rho\frac{D\underline{v}}{Dt}
& =-\underline{\nabla}p+\underline{\nabla}\cdot\underline{\underline{\sigma}}+\rho\underline{g} \\\\
\rho\frac{D\underline{v}}{Dt}
& =-\underline{\nabla}p+\mu\nabla^{2}\underline{v}+\rho\underline{g}
\end{aligned}
$$

```tex
\begin{aligned}
\rho\left(\frac{\partial\underline{v}}{\partial t}+\underline{v}\cdot\underline{\nabla}\underline{v}\right)
& =-\underline{\nabla}p+\mu\nabla^{2}\underline{v}+\rho\underline{g} \\\\
\rho\frac{D\underline{v}}{Dt}
& =-\underline{\nabla}p+\underline{\nabla}\cdot\underline{\underline{\sigma}}+\rho\underline{g} \\\\
\rho\frac{D\underline{v}}{Dt}
& =-\underline{\nabla}p+\mu\nabla^{2}\underline{v}+\rho\underline{g}
\end{aligned}
```

These cases both rendered quickly and correctly, as desired.

# Conclusion

With that, I've got this site in a satisfactory place to begin using to document and share my learnings.
I've learned my way around Hugo a bit, and tended to have enjoyed it so far.
Hopefully the above has provided a reasonable overview of why I chose Hugo for this site and how I set it up.
Better yet, hopefully it will be of some use to someone else.
