---
title: "Setting up This Site"
date: 2019-12-03T11:27:53-05:00
draft: true
toc: false
images:
tags: 
  - untagged
---

# Introduction

Personal websites may have many uses, and at the time of writing, the uses for this site are not yet clear.
**The two primary motivations for creating this site were:**

1. Create a place to document and share my learnings
2. Learn some new tooling to support 1

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

# Hugo

This section will provide a brief overview of setting up Hugo, and my process and workflow for creating content and deploying the site.
[Hugo Docs](https://gohugo.io/documentation/)

```bash
# Install Hugo
$ brew install hugo

# Create a new site
$ hugo new site my-website

# Install a theme
$ cd my-website
$ git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke

# Edit config
$ vi config.toml

# Create my first post
$ hugo new posts/new-post.md
```

I wanted to host on Github Pages.
Github Pages automatically generates Jekyll sites, but hosting Hugo sites on Github Pages simply requires only Hugo's output be plumbed to the Github Pages repository, which is easily accomplished using submodules.
In this case, the Github Pages repository is `dpwiese.github.io`.
The site generated above should be commited to a different repository, for example `my-site`.
Then, the `public` folder, the output of Hugo, should be made a submodule from the `dpwiese.github.io` repository using the following command.

```bash
$ git submodule add -b master git@github.com:dpwiese/dpwiese.github.io.git public
```

This is described more completely in the Hugo docs: [Host on Github](https://gohugo.io/hosting-and-deployment/hosting-on-github/).

## Workflow

With everything now configured appropriately, the following workflow can be adopted.
A new branch can be created within the `my-website` superproject repository to support whatever site changes are currently being made.
The `hugo server` command can be used to monitor the changes to the site locally, as they are being made.
Commits to the superproject repository can be made as needed.
When the site is ready to deploy, simply use the `hugo` command to generate the latest contents in the `public` submodule, and then commit as follows:

```bash
$ cd public
$ git add .
$ git commit -am "comment"
$ git push origin master
```

Thus, every commit to the submodule `master` branch is a deployment.

<!-- GitHub Pages Limitations#
GitHub Pages makes it easy for someone to generate a site for their project for free and with minimal effort. However, the following limitations of GitHub Pages might force you to use an alternative solution for hosting your web content:

Max. file size of 100MB
Repositories below 1GB
Risk of account deactivation if traffic is above average usage.
Expiry is set to 10 minutes, which is bad performance-wise.
Although GitHub Pages’ main advantage is its price point (free), the above limitations may not be acceptable for certain projects. For example, blogs that use many images or other large resources will quickly meet the 1GB GitHub size limitation. Therefore using one of the alternatives mentioned below for your GitHub CDN integration may prove to be a more acceptable solution.

GitHub Pages Alternative -->
