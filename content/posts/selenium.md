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

## JavaScript Chrome Snippet Attempt

Two main difficulties were encountered with this approach.
The first was around how to automatically trigger the Snippet to run when the <code style="color:#1269d3;">[load](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event)</code> event fired, making sure the necessary content was present.
The second was actually downloading the content from the site, which CORS issues made difficult.

When looking for solutions to the former problem, Chrome Extensions seemed to offer a solution, as did the simpler and less elegant solution of using <code style="color:#1269d3;">[setInterval](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)</code> to wait a fixed amount of time between operations, determined through trial-and-error to be suitable to allow the page and its resources to load.
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

# Selenium with Python

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
sign_in_button = driver.find_element(By.ID, "sign_in_button")

# Input email and password
email_input.send_keys("email@danielwiese.com")
password_input.send_keys("mypassword")

# Click sign-in button
sign_in_button.click()
```

## Ordering of Elements

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

The list of elems will look something like the below, where each element indicates its [session ID](https://w3c.github.io/webdriver/webdriver-spec.html#dfn-session-id) and [web element reference](https://w3c.github.io/webdriver/webdriver-spec.html#dfn-web-element-reference), a UUID that that uniquely identifies the element across all browsing contexts.
_Note that the web element reference is different than the element's HTML ID attribute._

```sh {linenos=false}
# Elements 0 - 2:
<selenium.webdriver.remote.webelement.WebElement (session="44790f5065a4e72fb48be21f6e3ad487", element="7a47df73-ed1f-4d2f-9a5e-d9da385027ff")>
<selenium.webdriver.remote.webelement.WebElement (session="44790f5065a4e72fb48be21f6e3ad487", element="a66c9945-b626-4986-8355-d1c7876edc58")>
<selenium.webdriver.remote.webelement.WebElement (session="44790f5065a4e72fb48be21f6e3ad487", element="e07a41e2-dc34-42a6-939d-20edaeb1b40b")>
```

The above loop will work for the first iteration if the click action navigates to a new page and updates the [browsing context](https://html.spec.whatwg.org/#browsing-context).
Then upon navigating back to click on the next element, the following error will be received:

```sh {linenos=false}
selenium.common.exceptions.StaleElementReferenceException: Message: stale element reference: element is not attached to the page document
```

Again getting the list of all elements by class name, it can be verified that with the browser context change, the elements have been assigned a new web element reference.

```sh {linenos=false}
# Element 2:
<selenium.webdriver.remote.webelement.WebElement (session="44790f5065a4e72fb48be21f6e3ad487", element="a04f89e5-9e13-4f96-8a16-7caacaed4c8b")>
```

To iterate over all the buttons with class `button-class-name`, each time clicking into the page thus updating the browsing context, knowledge of which elements have and have not been traversed needs to be retained independent of the browsing context.
One solution was to leverage that each button element had an `ID` attribute set.

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

However, it may not be the case that a list of elements have an `ID` attribute.
In this caes, when the browsing context updates, `find_elements()` would need to be called again.
The [Locator Strategies](https://w3c.github.io/webdriver/webdriver-spec.html#locator-strategies) W3C doc indicates that <code style="color:#1269d3;">[querySelectorAll()](https://dom.spec.whatwg.org/#dom-parentnode-queryselectorall)</code> is called to return the elements as a [collection](https://dom.spec.whatwg.org/#concept-collection-static), which will be returned in [tree order](https://dom.spec.whatwg.org/#concept-tree-order):

> The collection then represents a view of the subtree rooted at the collection’s root, containing only nodes that match the given filter. The view is linear. In the absence of specific requirements to the contrary, the nodes within the collection must be sorted in tree order.

Another strategy used to locate elements returns the result of calling <code style="color:#1269d3;">[evaluate](https://www.w3.org/TR/DOM-Level-3-XPath/xpath.html#XPathEvaluator-evaluate)</code> which itself returns an [XPathResult](https://www.w3.org/TR/DOM-Level-3-XPath/xpath.html#XPathResult) that _can_ return document-ordered nodes, although it seems this may not be the default case.
However, it didn't seem to state if the WebDriver's `evaluate` call did set the `type` to return ordered nodes.
The details here are not really clear, but there is plenty of documentation to read through to better understand exactly how this works.
It's also worth mentioning in [Element Location Strategies](https://www.w3.org/TR/2013/WD-webdriver-20130312/#element-location-strategies) from the March 2013 W3C specification the ordering of elements is more clearly stated:

> All element location strategies must return elements in the order in which they appear in the current document.

## Finding Elements by XPath

Above, `find_elements` was used to return an (ordered) list of elements matching the specified class name, or ID.
Selenium offers [other ways to find elements](https://selenium-python.readthedocs.io/locating-elements.html), some of which are fairly obvious and easy to use, others less so.
Particularly by XPath, which I was not very familiar with but found very powerful.

The first useful tool I discovered was Chrome DevTools [XPath Function](https://developer.chrome.com/docs/devtools/console/utilities/#xpath-function) `$x(path, [startNode])`.
This was really helpful when learning XPath selector syntax, as it can be based in the Chrome (and other browsers that support it) console to immediately view and experiment with the results.
For example `$x("//div")` will return all `div`s.


[devhints.io Xpath cheatsheet](https://devhints.io/xpath)


```python
# Get only one level deep versus infinite. The latter is nice to find nested elements but when
# iterating over returned elements it can get text twice For example if we have 
# <div>hello <i>world</i></div> then "world" will be retrieved twice. So the first example only
# goes one level deep.
nested_elem = elem.find_elements(By.XPATH, "./*")
nested_elem = elem.find_elements(By.XPATH, ".//*")

# Finding by XPATH based on partial match against element ID
links = driver.find_elements(By.XPATH, "//div[contains(@id, 'my-div-id')]")
chapters = driver.find_elements(By.XPATH, "//a[starts-with(@id, 'my-anchor-id-prefix-')]")

# Find the first child element by XPATH
first_element = elem.find_elements(By.XPATH, "*")[0]
```

## Race Conditions

Challenge: race conditions between when the browser is instructed to do something and the browser completes the instructions: https://www.selenium.dev/documentation/webdriver/waits/

Discuss the use of `time.sleep(2)` versus waits: https://www.selenium.dev/documentation/webdriver/waits/

# Summary
