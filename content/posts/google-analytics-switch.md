---
title: "Switching from Google Analytics to Umami"
date: 2024-02-25T09:55:15-05:00
draft: false
toc: false
images: ["img/posts/google-analytics-switch/og-image.png"]
tags:
  - google analytics
  - umami
keywords: [google analytics, umami]
description: "This post describes the switch from Google Analytics to Umami on this site."
---

<img src="/img/posts/google-analytics-switch/og-image.png" width="800" />

# Introduction

This post is meant to be a quick record of the switch from Google Analytics to [Umami](https://umami.is).

This site exists to store various bits of information or notes from small projects I have worked on in the hopes that they will be beneficial to others.
And whilst I hope the information *is* useful to others, I also frequently reference things I have written here myself.
The projects I work on and corresponding notes I write up are influenced in no way by what I think people want to read -- rather, as I work on various side projects that are interesting to me it is minimal extra effort to share these notes with the world.

Google Analytics was set up on this site at some point after it launched for two primary reasons: it was trivial to include out-of-the-box with Hugo, and only out of occasional curiousity as to why people would come to my site, and where they would visit from.
This site gets very little traffic, and I would find myself logging in to Google Analytics a couple times a year.

However, I wanted to find a non-Google and privacy oriented solution to satisfy my small analytics curiousity, and use this site as an excuse to investigate what options might exist.
This post shares the trivially easy switch from Google Analytics.

# Google Analytics Alternatives

The search for alternatives began with some simple web searches and reading a couple threads on Hacker News, e.g. [Ask HN: Best alternatives to Google Analytics in 2021?](https://news.ycombinator.com/item?id=29662859) and [Ask HN: Alternatives to Google Analytics in 2023](https://news.ycombinator.com/item?id=34391374).

Given the use case described above, the requirements I had for a Google Analytics alternative were very open.

* I didn't really care what features the service offered, as I expected the base level features of all to more than satisfy my curiousity.
* I wasn't opposed to paying a few dollars per month -- perhaps around the $4/mo price point of a Github Team account.
* I wanted a hosted solution, but ideally with a self-hosted option.

Ideally whatever solution I selected would be able to import my Google Analytics history, but it seemed that the barrier to this would largely come from hinderances to my ability to *export* the data from Google, rather than a product's ability to import.
After realizing the export would be non-trivial, I was content with abandoning my analytics history in switching to a different platform.
Again, the data serves no real purpose.

In any case, the following were some of the options that seemed well liked and used to satisfy at least somewhat similar needs to my own.
It became quickly clear for my use case that price would be the biggest factor driving me towards a solution.

* **[Plausible](https://plausible.io)**
  * $90/yr
  * [Self-hosted option available](https://plausible.io/self-hosted-web-analytics)
* **[Matomo](https://matomo.org)**
  * $230/yr
  * [Self-hosted option available](https://matomo.org/what-is-on-premise/)
* **[Simple Analytics](https://www.simpleanalytics.com)**
  * $108/yr
* **[Fathom Analytics](https://usefathom.com)**
  * $150/yr
  * [Fathom Lite](https://github.com/usefathom/fathom) is a self-hosted option
* **[GoatCounter](https://www.goatcounter.com)**
  * Free
  * Available as a free donation-supported hosted service or self-hosted app
* **[Umami](https://umami.is)**
  * Free
  * Self-hosted or fully-managed Umami Cloud

In the end I went with [Umami](https://umami.is).
It seemed like a mature enough solution with room to grow should my use case ever change, and offered a "hobby" tier that should be sufficient for my needs.
The process of actually removing Google Analytics and using Umami was a one-line change in a layout file.
