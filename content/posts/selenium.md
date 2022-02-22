---
title: "Browser Automation with Selenium and Python"
date: 2022-02-21T09:18:19-05:00
draft: true
toc: false
images:
tags: 
  - programming
  - python
keywords: [programming, python]
description: "This post describes how to automate browser tasks with Selenium and Python."
---

# Introduction

This project started out of the need to automate a tedious browser task of navigating to hundreds of pages and downloading images and text from the page to create an offline set of flashcards.
Writing a bit of JavaScript in a Chrome [Snippet](https://developer.chrome.com/docs/devtools/javascript/snippets/) seemed like a natural place to start, using the [Document](https://developer.mozilla.org/en-US/docs/Web/API/Document) Web API to find the desired elements by their various HTML attributes, and navigating to various pages where the content lives.

# JavaScript

Two main difficulties were encountered with this approach.
The first was around how to automatically trigger the Snippet to run when the <a href="https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event" target="_blank"><code style="color:#1269d3;">load</code></a> event fired, making sure the necessary content was present.
The second was actually downloading the content from the site, which CORS issues made difficult.

When looking for solutions to the former problem, Chrome Extensions seemed to offer a solution, as did the simpler and less elegant solution of using <a href="https://developer.mozilla.org/en-US/docs/Web/API/setInterval" target="_blank"><code style="color:#1269d3;">setInterval</code></a> to wait a fixed amount of time between operations, determined through trial-and-error to be suitable to allow the page and its resources to load.
The latter problem proved a bit more difficult, and after a few minutes of tinkering a suitable solution was found: simply print the desired text and image URLs to the console as JSON from where they could be copied out and pasted into a separate file for local post-processing.

This awkward solution worked well enough, with a few lines of JavaScript quickly spitting to the console JSON with hundreds of URLs and text descriptions.
A local Python script was used to download the images and name them as desired.
However, this approach was very crude and unsatisfying, and soon new requirements made it clear that a new approach was needed.
For example, the images were overlayed with `<div>`s carefully placed on top of the image as numbered annotations that needed to be captured as well.
For this reason new approaches were considered, finally landing on Selenium.

# Selenium

[Selenium](https://www.selenium.dev) is a tool for automating browsers, used primarily for testing web applications but also useful for automating web-based tasks.
Selenium provides several tools, [WebDriver](https://www.selenium.dev/documentation/webdriver/) being the the API used to drive the browser that is needed for the problem at hand.
Selenium should provide a much more elegant solution and not suffer the same drawbacks as the initial approach using a Chrome Snippet.
The Selenium library supports six programming languages: Java, Python, C#, Ruby, JavaScript, and Kotlin.
For this work Python was selected.

There were other alternatives that weren't considered too carefully for the purposes of this project.
Previously, I'd only had very limited exposure to [Cypress](https://www.cypress.io), used for testing a web application.
The [Cypress Trade-offs doc](https://docs.cypress.io/guides/references/trade-offs#Automation-restrictions) states plainly that Cypress is not a general purpose automation tool, and has some nice documentation describing these limitations and how it is different from Selenium.
Other potential options included [Puppeteer](https://pptr.dev), [Playwright](https://playwright.dev), [Protractor](https://www.protractortest.org/#/), [WebDriverIO](https://webdriver.io), and [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/).
This post won't attempt to describe or compare these alternatives with Selenium.

## Selenium with Python

The [PyPI documentation](https://pypi.org/project/selenium/) and [Selenium with Python](https://selenium-python.readthedocs.io) docs are straightforward enough to follow to install the Selenium library.

Challenge: race conditions between when the browser is instructed to do something and the browser completes the instructions: https://www.selenium.dev/documentation/webdriver/waits/

WebDriver vs DevTools

The particular site in question is written in Angular.
