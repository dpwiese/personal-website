---
title: "Generating Aviation Checklists with Make"
date: 2023-01-22T08:13:22-05:00
draft: true
toc: false
images: ["img/posts/makefile-checklists/og-image.jpg"]
tags: 
  - programming
  - pandoc
  - aviation
keywords: [programming, pandoc, aviation]
description: "This post describes how to use Make and Pandoc to generate PDF output from nested structure of source HTML files."
---

# Introduction

The safe operation of aircraft is enabled in large part by following checklists across all phases of flight.
While manufacturers provide the basic checklist items for most things in the aircraft flight manual, supplemental checklists, often in single card or small booklet, provide a more convenient format to use during aircraft operations.
Such checklists, often created by maintained by flight schools, individuals, or sold by third parties, allow for additional items to be included beyond those specified by the manufacturer.
This is particularly useful if aircraft has been modified from its original configuration, or when used for flight training, where additional information and context within the checklist is beneficial to the learner.

*Disclaimer: the checklist image below is a representative output using the method described in this post and should not be used for the operation of your aircraft. Consult your aircraft's pilot operating handbook / approved flight manual to ensure safe and proper operation.*

<img src="/img/posts/makefile-checklists/checklist.jpg" width="600" style="border-style:solid;border-width:1px"/>

My experience in general aviation has led me to develop and maintain my own checklists.
Originally, these were in the form `.docx` documents that were created in Microsoft Word.

While I've not attempted to understand the `.docx` specification, I encountered enough difficulty when attempting to open and edit old checklists in other applications like Apple's Pages and Google Docs which support the `.docx` format.
I decided to recreate my checklists in a more open format.

I wanted to create simple source documents in a human-readable file format that was widely accessible by open-source software, that could be converted to PDF also using open-source software, and that was easily styled to achieve the desired appearance.
**HTML and CSS were the obvious choice to generate the desired document layout, and [Pandoc](https://pandoc.org/) quickly became a natural choice to convert these source files into a PDF.**

I'd used Pandoc quite a bit before, although for a slightly different use case, described below.

# Pandoc and Markdown Background

My existing approach to documentation, originally described in [Documentation](/posts/documentation/), was to write content in Markdown and convert to styled PDF with Pandoc.

When calling Pandoc to generate an output from some input, many command-line options can be passed to affect the generated output.
To simplify the passing of many command-line options each time Pandoc is run, they can be stored in a default YAML file and passed to Pandoc with the `--defaults` option as follows.

```sh
pandoc --defaults defaults.yaml -o out.pdf in.md
```

The `defaults.yaml` file contains fields that correspond to command-line option settings with the mapping between the two described in the [Pandoc User's Guide](https://pandoc.org/MANUAL.html) [Defaults files](https://pandoc.org/MANUAL.html#defaults-files).
The settings in `defaults.yaml` are overridden by subsequent command-line options.
Some of the more important options specified in `defaults.yaml` are the document template, Lua filters, and LaTeX headers.

Additional options can be passed to Pandoc via the `--metadata-file` option, which, like all other command line options can be included in `defaults.yaml`.
So `defaults.yaml` could include `metadata.yaml` which contains additional document configuration.
Metadata can also be included in the document with a [yaml_metadata_block](https://pandoc.org/MANUAL.html#extension-yaml_metadata_block), where metadata values specified inside the document overwrite values specified with `metadata.yaml`.

With reasonable defaults specified for a class of documents and individual documents customized with the YAML metadata block, this allows for highly customizable document outputs to be generated with the same simple arguments when calling Pandoc.

# Updated Approach

It was easy enough to throw together some simple HTML and CSS and generate an acceptable output.
Due to the relatively large size of the HTML document, each single-sided page of the checklist was saved as a single file, and several files constituted a complete checklist, usually just a few pages.
I also maintained checklists for more about ten aircraft, resulting in about 50 different HTML files.
After the checklists were created, the nature of any future updates were expected to be small - the addition or deletion of a single item, rewording of an item, or the swapping of position of a pair of items after which an update checklist would be generated.

I wrote a small shell script that contained the Pandoc command that I would run when I wanted to update the checklists.
However, with each checklist taking a few seconds to generate the output, I quickly found myself looking for a simple way to only generate new outputs for those checklists whose underlying source files had changed.
*This seemed like a perfect use case for [Make](https://www.gnu.org/software/make/)*.

# Make

I hadn't really worked with Make before, except in projects which already incorporated it, and where any of my modifications to the `makefile` were trivial.
[makefiletutorial.com](https://makefiletutorial.com) was a great place to start, and I stumbled across Github user [rueycheng](https://gist.github.com/rueycheng)'s' [GNU Make Cheatsheet](https://gist.github.com/rueycheng/42e355d1480fd7a33ee81c866c7fdf78) to be very helpful as well.
Below are my notes on how I used Make to satisfy the above use case.
From my limited use with Make on this project, it seems very powerful but also took a little bit to start understanding its capabilities.

First check what version of Make is installed.

```sh
make --version

# GNU Make 3.81
# Copyright (C) 2006  Free Software Foundation, Inc.
# This is free software; see the source for copying conditions.
# There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
# PARTICULAR PURPOSE.

# This program built for i386-apple-darwin11.3.0
```

## Requirements

As I put together a simple `makefile`, I started to formulate some additional constraints that I wanted to respect.
First, I wanted to keep my existing project directory and naming structure as shown below, and make Make work with that structure, rather than adopt a directory structure that might be better suited to how Make might normally be used.
I wanted the addition of new aircraft checklists to be as straightforward as possible, and simply be able to run `make all` to generate the new checklist, without any changes to the `makefile`.

<img src="/img/posts/makefile-checklists/directories.png" width="260" style="border-style:solid;border-width:1px"/>

# Makefile

First some basic directories were defined.
This was pretty straightforward, with each element of `TARGETS` corresponding to a directory in `src`.

```sh
# Project directories
CWD := $(abspath $(shell pwd))
SRC := $(CWD)/src
OUT := $(CWD)/out
SRC_DIRS := $(wildcard $(SRC)/*)

# Define one output file name for each directory in src
TARGETS := $(addsuffix .pdf,$(subst src,out,$(SRC_DIRS)))

# External directories
PANDOC=/opt/homebrew/bin/pandoc
PANDOC_OPTIONS=--defaults ./pandoc/defaults.yaml
RM=/bin/rm -f

# SRC_DIRS
# /my/absolute/path/checklists/src/plane1
# /my/absolute/path/checklists/src/plane2

# TARGETS
# /my/absolute/path/checklists/out/plane1.pdf
# /my/absolute/path/checklists/out/plane2.pdf
```

With each target defined as above, the following rule could be created to trigger the making of all the checklists.
That is, all of the output PDFs, or `TARGET`s are prerequesites for `all`.

```sh
all: $(TARGETS)
```

Next, rules needed to be created to generate each of the output PDFs from source.
This was fairly straightforward using rules like the following.

```sh
# Rule to generate /my/absolute/path/checklists/out/plane1.pdf
$(OUT)/plane1.pdf: $(CWD)/css/checklist.css $(wildcard $(SRC)/plane1/*.html)
  @$(PANDOC) $(PANDOC_OPTIONS) $(filter-out $<,$^) -o $@

# Rule to generate /my/absolute/path/checklists/out/plane2.pdf
$(OUT)/plane2.pdf: $(CWD)/css/checklist.css $(wildcard $(SRC)/plane2/*.html)
  @$(PANDOC) $(PANDOC_OPTIONS) $(filter-out $<,$^) -o $@

# Repeat this rule for each plane
# ...
```

These rules indicate the following.
Consider the target `/my/absolute/path/checklists/out/plane1.pdf`.
The dependencies are `/my/absolute/path/checklists/css/checklist.css` and all HTML files in the directory `/my/absolute/path/checklists/src/plane1`.

The command calls Pandoc, specifies the `--defaults` file, and the `\$(filter-out \$<,\$^)` command uses [automatic variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables) to pass all the HTML files: `$^` gives the names of all prerequisites from which the first prerequisite, specified by `$<`, is filtered out, leaving only the HTML files being passed to Pandoc.
The CSS is used from within the HTML file and need not be passed to Pandoc.
`$@` indicates that the output is the name of the target.
Finally, the leading `@` on the command suppresses the output from being printed.
The resulting command is below.

```sh
/opt/homebrew/bin/pandoc --defaults ./pandoc/defaults.yaml \
/my/absolute/path/checklists/src/plane1/plane1-page1.html \
/my/absolute/path/checklists/src/plane1/plane1-page2.html \
/my/absolute/path/checklists/src/plane1/plane1-page3.html \
/my/absolute/path/checklists/src/plane1/plane1-page4.html \
-o /my/absolute/path/checklists/out/plane1.pdf
```

A command like this would be run for each `plane` directory I had as long as the required rule existed in the makefile.
With minimal knowledge of Make, and a couple dozen line `makefile`, the project was nearly complete.

**One unsatisfying part of this solution was the need for a new rule to be added each time a new checklist source directory was created.**
While not really a huge deal, it seemed easy enough solve to write a single rule to specify generically each target's prerequesites and the command to generate the target.

## A More Flexible Solution

As I learned Make, a solution may have been possible using `VPATH` to [Search Path for All Prerequisites](https://www.gnu.org/software/make/manual/make.html#General-Search), but seemed like a departure from what I expected the obvious solution to look like, and felt more complicated than necessary.

I then considered [pattern rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules) to simply substite the occurences of `planeN` from each rule with a stem that would match each desired target.
This was the "obvious" solution I was looking for.
Unfortunately it didn't work.

```sh
# Doesn't work as $(wildcard $(SRC)/%/*.html) is empty
$(OUT)/%.pdf: $(CWD)/css/checklist.css $(wildcard $(SRC)/%/*.html)
  @$(PANDOC) $(PANDOC_OPTIONS) $(filter-out $<,$^) -o $@
```

When this rule is evaluated, `\$(wildcard \$(SRC)/%/*.html)` looks for HTML files in the directory `/my/absolute/path/checklists/src/%/`, which is not unexpected.
After some quick searching, I was introduced to [secondary expansion](https://www.gnu.org/software/make/manual/make.html#Secondary-Expansion) by way of an [answer](https://unix.stackexchange.com/a/566135) to a Stackexchange question [Using wildcard in GNU Make pattern rule](https://unix.stackexchange.com/questions/297288/using-wildcard-in-gnu-make-pattern-rule).
Secondary expansion can be used to substitute the stem *and then* apply the `wildcard` function.
As the docs state:

> As make searches for an implicit rule, it substitutes the stem and then performs secondary expansion for every rule with a matching target pattern.

A quick modification to the initial solution gave the following, which worked as expected.

```sh
# Solution 1 using secondary expansion
.SECONDEXPANSION:
$(OUT)/%.pdf: $(CWD)/css/checklist.css $$(wildcard $(SRC)/%/*.html)
  @$(PANDOC) $(PANDOC_OPTIONS) $(filter-out $<,$^) -o $@
```

Knowing how to use secondary expansion, another solution inspired by the second example in the docs was the following.
Here, on the first expansion `PREREQS` won't be fully expanded.
It is then assigned a value using `$@` which is available during the second expansion, giving `PREREQS` the desired value.

```sh
# Solution 2 using secondary expansion
.SECONDEXPANSION:
$(OUT)/%.pdf: $(CWD)/css/checklist.css $$(PREREQS)
  @$(PANDOC) $(PANDOC_OPTIONS) $(filter-out $<,$^) -o $@
PREREQS = $(wildcard $(SRC)/$(subst .pdf,,$(notdir $@))/*.html)
```

# Checklist Source

The checklist source is exceedingly simple and not valuable enough to share here.
It's a 3-column flexbox layout with minimal CSS.

# Summary

This post provided a simple solution using Make to call Pandoc to generate PDF outputs from several source HTML and CSS files, while maintaining the desired project directory structure.
While other solutions were surely possible, the use of secondary expansion allowed the obvious but non-working solution attempt to work by adding the special `.SECONDEXPANSION` target and adding a single character.
