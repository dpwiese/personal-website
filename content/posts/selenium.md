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

[Selenium](https://www.selenium.dev) is a framework for automating browsers, used primarily for testing web applications but also useful for automating web-based tasks.
Selenium provides several tools, [WebDriver](https://www.selenium.dev/documentation/webdriver/) being the the API used to drive the browser that is needed for the problem at hand.
The [W3C WebDriver](https://w3c.github.io/webdriver/) specification has standardized this interface, and it is implemented through a browser-specific driver that controls the browser of choice performing the desired commands.
For example, [ChromeDriver](https://chromedriver.chromium.org/getting-started) is used to drive Chrome, which is the browser driver used for this project.

## WebDriver and DevTools

There are alternatives to WebDriver to control the browser, for example the [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) or [Microsoft Edge DevTools](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide-chromium/) (that are based off of the Chromium tools).
Alternative frameworks to Selenium/WebDriver exist, some of which use WebDriver, others DevTools, and some that offer support for both.
Each of these frameworks offers their own advantages and features, but under the hood the mechanism by which they control the browser will come with its own benefits and limitations.
For example [Puppeteer](https://pptr.dev) uses the DevTools protocol, [Playwright](https://playwright.dev) uses the Chrome DevTools for Chromium-based browsers, and similar but custom protocols for Firefox and WebKit, and [WebDriverIO](https://webdriver.io) that supports both WebDriver and DevTools, as well as [Protractor](https://www.protractortest.org/#/), [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/), and [Cypress](https://www.cypress.io).
Previously I'd only had very limited exposure to Cypress used for testing a React web application.
The [Cypress Trade-offs](https://docs.cypress.io/guides/references/trade-offs#Automation-restrictions) doc states plainly that Cypress is not a general purpose automation tool, and the [Cypress Key Differences](https://docs.cypress.io/guides/overview/key-differences) and [Why Cypress](https://docs.cypress.io/guides/overview/why-cypress) docs describe some of its limitations and how it is different from Selenium.
This post won't attempt any further to describe or compare these alternatives with Selenium, nor was deep consideration given to these alternatives for the purposes of this project.


Selenium should provide a much more elegant solution and not suffer the same drawbacks as the initial approach using a Chrome Snippet.
The Selenium library supports six programming languages: Java, Python, C#, Ruby, JavaScript, and Kotlin.
For this project Python was selected.

## Selenium with Python

The [PyPI documentation](https://pypi.org/project/selenium/) and [Selenium with Python](https://selenium-python.readthedocs.io) docs are straightforward enough to follow to install the Selenium library.
Given how specific the code is to the website being scraped--leveraging specific element class names and IDs, for example--the full code will not be included here.
Rather important or interesting bits will genericized and shared.
For what it's worth, the particular site that was being scraped is written in Angular, but the Selenium implementation is agnostic to what framework was used to create the web application.

Typical dependencies for a simple project like this are shown below.
The first step is to load the driver (in this case the Chrome Driver) and navigate to the page.
This will launch an instance of the Chrome browswer locally where the commands will be executed and can be observed.

```python
# Dependencies
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.support.ui import WebDriverWait
from selenium.common.exceptions import ElementNotInteractableException
from webdriver_manager.chrome import ChromeDriverManager

# Load Chrome driver
s = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=s)

# Navigate to page
driver.get('https://danielwiese.com')
```

Should the particular site require a login to access the content that needs scraping, logging in is very straightforward.
It will involve finding the login form, entering credentions, and clicking the login button.

```python
# Find input fields and sign-in button
email_input = driver.find_element(By.ID, "email_input")
password_input = driver.find_element(By.ID, "password_input")
button_sign_in = driver.find_element(By.ID, "sign_in_button")

# Input email and password
email_input.send_keys("email@danielwiese.com")
password_input.send_keys("mypassword")

# Click sign-in button
button_sign_in.click()
```

After having signed in, the page may present all the information that needs to be scraped, but likely navigation to many pages will be required to retrieve the content.
In this case, the content was located on pages, each accessible by clicking a button on the main page.
So each of these button elements must be retrieved so they can be iterated over, clicking into each one.
This is the first small issue encountered.

```python
# Find all the buttons by class name
elems = driver.find_elements(By.CLASS_NAME, 'button-class-name')

# Get the IDs of all the buttons we need to click on
for el in elems:
    el.click()
    # Do stuff
    # Navigate back
```

The list of elems will look something like the below:

```sh {linenos=false}
# Elements 0 - 2:
<selenium.webdriver.remote.webelement.WebElement (session="44790f5065a4e72fb48be21f6e3ad487", element="7a47df73-ed1f-4d2f-9a5e-d9da385027ff")>
<selenium.webdriver.remote.webelement.WebElement (session="44790f5065a4e72fb48be21f6e3ad487", element="a66c9945-b626-4986-8355-d1c7876edc58")>
<selenium.webdriver.remote.webelement.WebElement (session="44790f5065a4e72fb48be21f6e3ad487", element="e07a41e2-dc34-42a6-939d-20edaeb1b40b")>
```

The above loop will work for the first iteration if the click action navigates to a new page and updates the DOM.
Then upon navigating back to click on the next element, the following error will be received:

```sh {linenos=false}
selenium.common.exceptions.StaleElementReferenceException: Message: stale element reference: element is not attached to the page document
```

Again finding a list of all elements by class name, the same element will have a different element ID.
_Note that the element ID is different than the element's HTML ID attribute._

```sh {linenos=false}
# Element 2:
<selenium.webdriver.remote.webelement.WebElement (session="44790f5065a4e72fb48be21f6e3ad487", element="a04f89e5-9e13-4f96-8a16-7caacaed4c8b")>
```

So, to iterate over all the buttons with class `button-class-name` each time clicking into the page, on navigating back the elements could all be retrieved again.
The index of the element could be stored and use to march through each element, even as the each element reference becomes stale upon navigating back to the page.

However, I'm not sure that there is any guarantee that the list of elements that is returned by Selenium maintains it's order?

In any case, each element had a unique ID attribute that could be saved and used to iterate over each element only once.

```python
# Find all the buttons by class name
elems = driver.find_elements(By.CLASS_NAME, 'button-class-name')

# Get the IDs of all the buttons we need to click on
elems_id_arr = []
for el in elems:
    elems_id_arr.append(c.get_attribute("id"))

# Go through each button ID attribute
for el_id in elems_id_arr:
    # Find the button to click on by its ID attribute
    button = driver.find_element(By.ID, el_id)
    button.click()
    # Do stuff
    # Navigate back
```


Challenge: race conditions between when the browser is instructed to do something and the browser completes the instructions: https://www.selenium.dev/documentation/webdriver/waits/


TODO
* Anatomy of a website (e.g. XPATH)
* Type hints, how they changes in 3.8 to 3.9+, and `mypy` for type checking
* Discuss the use of `time.sleep(2)` versus waits: https://www.selenium.dev/documentation/webdriver/waits/
* Some elements aren't clickable?
