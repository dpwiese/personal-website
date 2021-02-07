---
title: "React Native Charts Package"
date: 2021-02-07T10:42:00-04:00
draft: true
toc: false
images: ["img/posts/react-native-charts-package/og-image.png"]
tags: 
  - programming
  - react native
  - data visualization
  - canvas
keywords: [programming, react native, data visualization, canvas]
description: "This post provides ... ."
---

# Introduction

In the previous post, [Canvas Charts in React Native](/posts/react-native-canvas-charts/), a simple wrapper was made around a `WebView` to make using canvas-based charts in React Native more convenient.
The resulting package is available on npm as [@dpwiese/react-native-canvas-charts](https://www.npmjs.com/package/@dpwiese/react-native-canvas-charts).

# Making the Package

Before starting, the npm Docs are worth a read, in this case [Creating Node.js modules](https://docs.npmjs.com/creating-node-js-modules) and [Creating and publishing scoped public packages](https://docs.npmjs.com/creating-and-publishing-scoped-public-packages).
These steps won't be covered in detail here, but an overview is:
1. Create `package.json` with `npm init --scope=@dpwiese` (my scope name)
2. Configure `package.json` with the packages `main` file and build instructions
3. Build and publish the package

I had written the original code in TypeScript, and the build step was required to generate consumption-ready JavaScript.
I wasn't sure which compiler I wanted to use, but some good information on this was available in the TypeScript docs [Using Babel with TypeScript](https://www.typescriptlang.org/docs/handbook/babel-with-typescript.html).
[Babel](https://babeljs.io) is the default compiler for React Native, as described in the React Native docs [JavaScript Syntax Transformers](https://reactnative.dev/docs/javascript-environment#javascript-syntax-transformers), so I opted to use that over the TypeScript compiler.

The TypeScript compiler is still used for type checking and for generating type definitions.
The TypeScript docs [Creating .d.ts Files from .js files](https://www.typescriptlang.org/docs/handbook/declaration-files/dts-from-js.html) explain very well how to generate the type definition files.
This basically just requires in `tsconfig.json` the `"emitDeclarationOnly": true` compiler flag be set.
However, setting this flag conflicts with the `noEmit` flag, which should be passed when type checking.
So for this reason I opted not to set these flags in the configuration file, but rather pass them when invoking the compiler command.

## Basic Workflow

When building and updating this package, the workflow is as follows.

```sh {linenos=false}
# 1. Make changes to the package source

# 2. Build the package
% npm run build

# 3. Creates a tarball for testing the package locally
% npm pack

# 4. Install the package tarball to the project where it will be included and tested
% npm install ./dpwiese-react-native-canvas-charts-0.0.1.tgz

# 5. Bump the version in package.json and rebuild

# 6. Login and publish the package
% npm login
% npm publish
```

## Build Setup

Describe steps in `package.json` to build with Babel, type check, and generate type definitions.

## Package Structure

Describe how to organize and export code here.
For example, in subdirectories.

# Using the Package

Include basic demo here of how the package is used.

```sh
% npm i @dpwiese/react-native-canvas-charts
```


```jsx
import { ChartJs, DataPoint, SetData } from "@dpwiese/react-native-canvas-charts";

<ChartJs config={chartConfig1} style={styles.chart} ref={setDataRef} />
```

# Appendix

## TODO

* Import dependencies cleanly into the project without copy-pasting.

The last issue from the non-module implementation was to import Chart.js from separate `.js` source file without copy-paste into `.html` file.

Does this change now that I am building it as a module?

Before publishing the module can I just add a build script to fetch the latest JS and insert it?

That kinda sucks, but might be the most elegant way?
And, I shouldn't be committing such code anyway to my module's repo, given it's just a 3rd party dependency.
So seems I should handle it in my module as a dependency and then just stick it into the HTML somehow...

* When using `WebView` (which has native code) as dependency of my `react-native-canvas-charts` dependency, how properly install this native sub-dependency? See: [React-Native autolink a dependency of a dependency](https://stackoverflow.com/questions/60707869/react-native-autolink-a-dependency-of-a-dependency)

* How to organize module structure for convenient importing of pieces associated with each plotting library that is wrapped.

## Implementation

Testing node module before publishing
[Testing npm packages before publishing](https://dev.to/vcarl/testing-npm-packages-before-publishing-h7o)
[npm-link](https://docs.npmjs.com/cli/v6/commands/npm-link)

[TypeScript With Babel: A Beautiful Marriage](https://iamturns.com/typescript-babel/)

[Main property in package.json defines package entry point](https://bytearcher.com/articles/main-property-in-package.json-defines-entry-point/)

THIS WAS FIX FOR JEST CONFIG?
https://github.com/facebook/jest/issues/6229#issuecomment-433632564
Actually I don't think so.
I think tests broke when I didn't install `react-native-webview` as a dependency in the project, instead of it having been installed as a dependency of `react-native-canvas-charts`.

https://medium.com/angular-in-depth/npm-peer-dependencies-f843f3ac4e7f
