---
title: "Setting up this Site"
date: 2019-12-03T11:27:53-05:00
draft: true
toc: false
images:
tags: 
  - programming
---

<!-- edit this document for tense: past or present -->

# Introduction

Personal websites may have many uses, and at the time of writing, the uses for this site are not yet clear.
**The two primary motivations for creating this site were:**

1. Create a place to document and share my learnings
2. Learn some new tooling to support motivation 1

A tertiary motivation was to provide a place for people to learn more about me than might be possible via the infomration available on social media.

With this in mind, I began to think about the form that the items of motivation 1 would take, whether it be blog posts, standalone documents, etc. and thus which tooling would best support this.
The outcome of this was that the majority of the items I expected to create for motivation 1 would be single page posts, and that the best tooling would be in the form of a static site generator.

The reasons for using a static site generator based on current and anticipated needs are as follows:

1. I want something fast and simple to setup, edit, host, and maintain.
2. I don't need a content management system such as Wordpress to facilitate making changes to the site, nor does anyone else - I will be the sole content creator and manager of the site.
3. I don't need any server-side functionality.
4. I don't need access to the extent of plugins offered by a CMS like Worpdress, such as those to facilitate SEO, marketing automation, or otherwise.

Most tooling was not precluded due to a lack of programming ability as may be the case if I did not have significant development experience.
These reasons heavily pushed me towards a solution which supported markdown, as much of my existing documentation is already written in markdown as it is well suited to the type of writing I do.
Simplicity meant I did not want a solution with an involved build process, that required a database, or to manage and update plugins.
The changes introduced when updating plugins are not always fully known, and are often necessary to provide security to the site.
This all meant that solutions with content management systems, such as Wordpress, or a web application like React would not be suitable for my needs.

The benefits of static site generators, and comparions to CMS are well documented online. To briefly share some of these benefits within the context of my needs,
All the particular SSGs I considered were very fast to setup, edit, and host.
With just a few simple commands I could install a SSG, create and populate a new page with content, apply a theme, and host.
It was a matter of a few minutes to create this site.
I didn't need to provision a database or sign up for some new hosting provider.
This site is currently hosted using Github Pages, but it just as easily could have been dropped into my existing S3 bucket.

# Picking a Static Site Generator

Once I had decided that a static site generator was the best choice for my needs, I had to pick one.
There are quite a few popular options such as Gatsby, Jekyll, Hugo, and Pelican to name a few, and many comparisons between these and others can be found online.
When evaluating these and other options, all of them were at least reasonable choices.
I didn't need the capabilities of React, or want the associated complexity of using it and GraphQL.
Jekyll was an obvious consideration.
I ultimately went with Hugo for two primary reasons: benchmarks indicated it to be significantly faster than most alternatives, very important for me and my 8 year old MacBook, and I wanted something built on Go, as opposed to Ruby, Python, or JavaScript.

# Setting up Hugo

<a href="https://gohugo.io/documentation/" target="_blank">Hugo Docs</a>

This section will provide a brief overview of setting up Hugo, and my process and workflow for creating content and deploying the site.
Themes can be found at: {{< plainlink "https://themes.gohugo.io/" >}}.
For this site I chose the <a href="https://themes.gohugo.io/hermit/" target="_blank">hermit</a> theme.

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

I'll discuss hosting more in a following section, but the choice to host on Github Pages impacted this portion of the setup.
Github Pages automatically generates Jekyll sites, but hosting Hugo sites on Github Pages simply requires only Hugo's output be plumbed to the Github Pages repository, which is easily accomplished using submodules.
In this case, the Github Pages repository is `dpwiese.github.io`.
The site generated above should be commited to a different repository, for example `personal-website`.
Then, the `public` folder, the output of Hugo, should be made a submodule from the `dpwiese.github.io` repository using the following command.

```bash
$ git submodule add -b master git@github.com:dpwiese/dpwiese.github.io.git public
```

This is described more completely in the Hugo docs: <a href="https://gohugo.io/hosting-and-deployment/hosting-on-github/" target="_blank">Host on Github</a>.
Note the following in <a href="https://help.github.com/en/github/working-with-github-pages/about-github-pages#user--organization-pages" target="_blank">About GitHub Pages</a> that says, for a user or organization page, it must be hosted from the `master` branch.

> The default publishing source for user and organization sites is the master branch.
If the repository for your user or organization site has a master branch, your site will publish automatically from that branch.
You cannot choose a different publishing source for user or organization sites.

## Workflow

With the Hugo site generated and now configured appropriately, with a first page of content and theme applied, the following workflow can be adopted.
A new branch can be created within the `personal-website` superproject repository to support whatever site changes are currently being made.
The `hugo server` command can be used to monitor the changes to the site locally as they are being made.
Commits to the superproject repository can be made as needed.
When the site is ready to deploy, simply use the `hugo` command to generate the latest contents in the `public` submodule, and then commit the submodule as follows:

```bash
$ cd public
$ git add .
$ git commit -am "comment"
$ git push origin master
```

Thus, every commit to the submodule `master` branch is a deployment.

## Styling

With the above simple workflow used to deploy this site, I then wanted to make some minor adjustments to the theme.
This can be accomplished by duplicating theme files from, for example, `./themes/hermit/layouts/index.html` to `./layouts/index.html` and then editing as needed to adjust certain elements.
Any such files in the equivalent path within the main project directory will overwrite those from the theme.
This is true as well for assets in `./themes/hermit/assets/scss/_predefined.scss` by `./assets/scss/_predefined.scss`.
Additionally, custom CSS can be applied from `./static/css/style.css`, for example by specifying in the `config.toml` file.

```toml
[params]
  customCSS = ["css/style.css"]
```

With these options, I wanted to be able to modify the theme in the least obtrusive way.
Overwriting layouts seemed like a bad idea - I'd be duplicating a substantial amount of code to make minor modifications, and the modified code (a duplicated SCSS file) may not even apply should I change the them later.
Because of this, I opted to import a couple small CSS files via `config.toml` and only change the few necessary elements.

## Hosting

I decided to use Github Pages for their free hosting, subject to the limitations described under their <a href="https://help.github.com/en/github/working-with-github-pages/about-github-pages#guidelines-for-using-github-pages" target="_blank">Guidelines for using GitHub Pages</a>.
These limitations should not be a limitation for the purposes of this site.
In any case, another natural alternative was S3, to which this site could be easily deployed by appropriate configuration of `config.toml` as described in the <a href="https://gohugo.io/hosting-and-deployment/hugo-deploy/" target="_blank">Hugo Deploy</a> docs.

## Hyperlinks in Hugo

When writing this post, one of the first things I wanted to do is create links that would open in a new tab.
The seemingly most straightforward way to do this, and something that is often useful in markdown, is to simply use html.
I found by inspecting the generated html when doing this, the following where my embedded html would have been:

```html
<!-- raw HTML omitted -->
```

After a bit of Googling and reading through the docs, I learned that with the latest version of Hugo v0.60 that was released just days before having installed it, the following addition to `config.toml` may be needed. Per the <a href="https://gohugo.io/news/0.60.0-relnotes/" target="_blank">v0.60 release notes</a>:

> Also, if you have lots of inline HTML in your Markdown files, you may have to enable the `unsafe` mode:

```toml
[markup.goldmark.renderer]
  unsafe = true
```

Another difficulty that was realized after adding html links such as the one below, was that when generating the static site, Hugo would interpret the text as a link and thus generate a standard `a` tag without the additional `target` attribute.

```html
<a href="https://gohugo.io/" target="_blank">https://gohugo.io/</a>
```

A relatively easy and simple way around this was using <a href="https://gohugo.io/content-management/shortcodes/" target="_blank">Shortcodes</a>.
The following shortcode file was created in `./layouts/shortcodes/plainlink.html`.

```html
<a href={{ index .Params 0 | safeHTML }} target="_blank">{{ index .Params 0 | safeHTML }}</a>
```

This provides a shortcode that can be called by name, render the contents using the specified template and the passed parameters.
In this case, the `plainlink` shortcode would be called and passed a url from within a posts markdown file as shown below.

```md
{{</* plainlink "https://themes.gohugo.io/" */>}}
```

The result, when the site was generated, was a correctly generated `a` tag that preserved the `target` attribute, ensuring the clicked link would be opened in a new window.

## Latex

## Working with Hugo

```bash
$ hugo list drafts
```
