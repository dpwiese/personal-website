---
title: "Matlab to Python with ChatGPT"
date: 2023-12-09T12:38:15-05:00
draft: false
toc: false
images:
tags:
  - matlab
  - python
  - chatgpt
keywords: [matlab, python, chatgpt]
description: "This post shares my results and experience converting some computational aerodynamics code from an undergraduate course from Matlab to Python using ChatGPT."
---

<img src="/img/posts/matlab-python-chatgpt/python-contour-plot.png" width="400" />

# Introduction

**This post shares my results and experience converting some code from UC Davis's EAE-126 Theoretical & Computational Aerodynamics class that I took as an undergraduate from Matlab to Python using ChatGPT.**
The main reasons for wanting to perform this conversion are to remove my dependency on a closed-source, low-quality, commercial software environment to run my own code, and to avoid using programming languages / environments which have limited value in learning outside a narrow domain.
There are many other reasons I have for wanting to not use Matlab, and while it may be an acceptable solution for some, there are others who may also wish to convert their Matlab code to Python and will find this post helpful.
**Both the Matlab and Python codebases can be found below:**

* [github.com/dpwiese/eae-126-matlab](https://github.com/dpwiese/eae-126-matlab)
* [github.com/dpwiese/eae-126-python](https://github.com/dpwiese/eae-126-python)

This post provides little more than a few tips when it comes to how best to use ChatGPT for such conversion.
There isn't much insight into so-called prompt engineering provided here.
Rather it shares a few notes from this experience, provides the results in the repositories linked above, and hopefully serves as a datapoint for others that at least for some codebases, ChatGPT is a potentially viable conversion tool.

# Other Code Conversion Solutions

When investigating how to best convert existing Matlab code to Python, I found some other existing solutions, but surprisingly not as many as I'd expected to see.
Examples included:

* [smop](https://github.com/victorlei/smop) (Small Matlab and Octave to Python compiler)
* [matlab2python](https://github.com/ebranlard/matlab2python)
* [mat2py](https://github.com/mat2py/mat2py) with a web app at [translate.mat2py.org](https://translate.mat2py.org)

Generally, such projects didn't seem to have gotten much traction, were stale, and ultimately didn't work very well.
While projects like these could certainly be improved, I was less interested at the time in contributing to these tools to the point where I could use them, and more focused on completing the conversion.
I had a quick look at [CodeConvert](https://www.codeconvert.ai), an AI tool which didn't seem to offer anything beyond [ChatGPT](http://chat.openai.com), and after initially trying ChatGPT I realized it would be a suitable tool for this particular conversion task.

# Using ChatGPT for Code Conversion

The use of ChatGPT for performing such conversions has many drawbacks and things to consider, with some of the ones relevant to my conversion listed below.

1. **Confidential code** - This project required the conversion of open-source, academic code which I own, and so confidentiality did not provide a barrier.
In other contexts, for example those involving confidential source, using ChatGPT may not be an option.
2. **Non-deterministic conversions** - The behavior of ChatGPT is rather inconsistent and unpredictable -- when provided the same body of code to convert, the results of conversion would vary greatly and often several iterations of prompts were necessary to finally achieve an acceptable output.
Furthermore, the bugs that resulted from this unpredictability in the generated output were often insidious and hard to spot.
3. **Support for all language features and third-party dependencies** - the codebase which I converted was very small -- on the order of a couple dozen independent scripts, each usually no more than a few hundred lines.
It generally consisted of only the most basic constructs and arithmetic operations, populating values of some matrix equations whose solution was a numeric solution to some underlying partial differential equation.
These could be easily expressed in Python with minimal dependencies beyond `matplotlib` and `numpy`.
I expect it would be much more difficult to convert a larger codebase which uses more Matlab functionality to Python using ChatGPT.
4. **Limited incremental conversion** - converting an Android application from Java to Kotlin, or a web app from JavaScript to TypeScript may be accomplished incrementally without too much difficulty.
Given the code I was converting was comprised of individual scripts, the conversion was easily performed one script at a time, without breaking any of the existing ones.
However, I expect that incrementally converting a larger, more complex codebase between Matlab and Python would be much more difficult.

## Additional Comments

The following are a few additional comments around aspects of the conversion that needed to be contended with.

* `0` vs `1` based array indexing, and variability in expression of ranges over which to iterate was the largest source of insidious bugs that arose from the conversion.
For example `for j = 2 : ny - 1` in Matlab has a Python equivalent of `for j in range(1, ny - 1)`.
It was easy in reviewing and running the converted code to miss bugs that arose when ChatGPT made a conversion mistake on a line like this.
* ChatGPT would often refactor the generated code in ways that were unhelpful.
The original code was written in a way that attempts to preserve structure of the mathematical equations and grid used when setting up the problem.
That this coding style, while certainly more verbose than it otherwise code have been, retained a clear mapping to the math made reading and troubleshooting the code much more easy.
Any refactoring ChatGPT attempted that broke this style was ultimately unhelpful.
* Lastly, my inability to run the Matlab source on the same environment on which it was written made checking the integrity of the conversion more difficult.
While the code did not use any features of Matlab that would have been introduced in the decade since this it was last run, I couldn't be sure that running the same code with Octave on my ARM Mac today, for example, would produce identical results to what was produced on my circa 2009 Windows Laptop running Matlab 2010 or earlier.

# Summary

While the conversion took a good part of a weekend and several evenings, the majority of the time spent was cleaning up the Matlab code prior to conversion.
The conversion itself was little more than passing the entire contents of a Matlab script to ChatGPT populating a Python script with the returned output.
The vast majority of my interactions with ChatGPT during this process were simply to tell it to finish the conversion, as it would very often stop part way through with a vague instruction for me to continue the rest.
Once the conversion was complete there were often a couple of small bugs to fix, but otherwise little post-processing was required.
While ChatGPT certainly isn't a perfect tool, nor the right tool for every such conversion job, it would certainly be one of the first places I will look to for future conversion needs.
