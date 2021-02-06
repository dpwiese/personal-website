---
title: "React Native Charts Module"
date: 2021-02-06T15:31:00-04:00
draft: true
toc: false
images: ["img/posts/TODO/og-image.png"]
tags: 
  - programming
  - react native
  - data visualization
  - canvas
keywords: [programming, react native, data visualization, canvas]
description: "This post provides ... ."
---

# Introduction

## Importing Into my Module

The last issue from the non-module implementation was to import Chart.js from separate `.js` source file without copy-paste into `.html` file.

Does this change now that I am building it as a module?

Before publishing the module can I just add a build script to fetch the latest JS and insert it?

That kinda sucks, but might be the most elegant way?
And, I shouldn't be committing such code anyway to my module's repo, given it's just a 3rd party dependency.
So seems I should handle it in my module as a dependency and then just stick it into the HTML somehow...

## TODO

* When using `WebView` (which has native code) as dependency of my `react-native-canvas-charts` dependency, how properly install this native sub-dependency? See: [React-Native autolink a dependency of a dependency](https://stackoverflow.com/questions/60707869/react-native-autolink-a-dependency-of-a-dependency)

## Implementation

Testing node module before publishing
[Testing npm packages before publishing](https://dev.to/vcarl/testing-npm-packages-before-publishing-h7o)
[npm-link](https://docs.npmjs.com/cli/v6/commands/npm-link)

Basically:

```sh {linenos=false}
% npm pack
% npm install ./dpwiese-react-native-canvas-charts-0.0.0.tgz
```

And then

```sh {linenos=false}
% npm login
% npm publish
```

[TypeScript With Babel: A Beautiful Marriage](https://iamturns.com/typescript-babel/)

[Main property in package.json defines package entry point](https://bytearcher.com/articles/main-property-in-package.json-defines-entry-point/)

Since Babel is used for compilation, the TypeScript compiler is only used for typechecking on command or in CI, and for generating the type definition files when building.

Some good information on this is from the TypeScript docs [Using Babel with TypeScript](https://www.typescriptlang.org/docs/handbook/babel-with-typescript.html).

Why did I use Babel to build? Mostly because RN does by default?

[Creating .d.ts Files from .js files](https://www.typescriptlang.org/docs/handbook/declaration-files/dts-from-js.html)

Basically, in `tsconfig.json` need to add

```json {linenos=false}
"emitDeclarationOnly": true
```

THIS WAS FIX FOR JEST CONFIG?
https://github.com/facebook/jest/issues/6229#issuecomment-433632564
Actually I don't think so.
I think tests broke when I didn't install `react-native-webview` as a dependency in the project, instead of it having been installed as a dependency of `react-native-canvas-charts`.


