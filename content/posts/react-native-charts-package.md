---
title: "Making React Native Canvas Charts Package"
date: 2021-02-14T10:42:00-04:00
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

<img src="/img/posts/react-native-charts-package/npmjs.png" width="800" style="border-style:solid;border-width:1px"/>

# Introduction

In the previous post, [Canvas Charts in React Native](/posts/react-native-canvas-charts/), a simple wrapper was made around a `WebView` to make using canvas-based charts in React Native more convenient.
This post contains a few notes describing the process of taking that simple wrapper and publishing it as a package, available on npm as [@dpwiese/react-native-canvas-charts](https://www.npmjs.com/package/@dpwiese/react-native-canvas-charts).

# Making the Package

Before starting, the npm Docs are worth a read, in this case [Creating Node.js modules](https://docs.npmjs.com/creating-node-js-modules) and [Creating and publishing scoped public packages](https://docs.npmjs.com/creating-and-publishing-scoped-public-packages).
The steps to make a (scoped) package won't be covered in detail here, but an overview is:
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
See [package.json](https://github.com/dpwiese/react-native-canvas-charts/blob/master/package.json) for how the build commands are defined.

## Basic Workflow

When building and updating this package, the workflow is as follows.

```sh {linenos=false}
# 1. Make changes to the package source

# 2. Build the package
% npm run build

# 3. Create a tarball for testing the package locally
% npm pack

# 4. Install the package tarball to the project where it will be included and tested
% npm install ./dpwiese-react-native-canvas-charts-0.0.0.tgz

# 5. Bump the version in package.json and rebuild

# 6. Login and publish the package
% npm login
% npm publish
```

This workflow was very easy to follow while building and updating the package.

## Build Setup

Given this package is just a simple wrapper around a `WebView` to making plotting with canvas-based libraries more convenient, I had anticipated adding support for several different libraries.
As such, I wanted to be able to import various components as types specific to each particular charting library:

```jsx {linenos=false}
import { Chart, SetData } from "@dpwiese/react-native-canvas-charts/ChartJs";
import { SetData, UPlot } from "@dpwiese/react-native-canvas-charts/UPlot";
```

I figured it would be trivial to configure the package this way, and perhaps it is, although it wasn't as obvious I expected it would be.
It seemed [babel-plugin-module-resolver](https://github.com/tleunen/babel-plugin-module-resolver) could be used for what I was trying to achieve, but it wasn't immediately obvious how to do this, especially given my seemingly very simple use case.
I looked at [subpath exports](https://nodejs.org/api/packages.html#packages_subpath_exports) as well, but again it still didn't seem to be an obvious and convenient solution.

I found the post [Publishing flat npm packages for easier import paths & smaller consumer bundle sizes](https://davidwells.io/blog/publishing-flat-npm-packages-for-easier-import-paths-smaller-consumer-bundle-sizes) that seemed to offer a reasonable solution which I opted to follow instead.
I won't discuss the pros and cons of this approach in this post - it was suitable for now.
It was very easy to do, and within minutes I was able to use the package as I'd intended.

This approach basically required `package.json` be copied to `dist` when building, and packing and publishing be done from `dist` as well.
To ensure I didn't accidentally publish the package from the root directory, the `prepublishOnly` pre-commit [script](https://docs.npmjs.com/cli/v6/using-npm/scripts) was used to prevent this, and a separate `publish` script added to handle publishing from `dist`.

```json {linenos=false}
"scripts": {
  "build": "rm -rf dist && babel src --out-dir dist --extensions '.ts,.tsx' --copy-files && tsc --project tsconfig.json --emitDeclarationOnly && cp -rf package.json dist && cp -rf README.md dist",
  "pack": "cd dist && npm pack && cd ..",
  "lint": "eslint src --ext .ts,.tsx --fix",
  "tc": "tsc --project tsconfig.json --noEmit",
  "publish": "cd dist && npm publish --ignore-scripts && cd ..",
  "prepublishOnly": "echo \"Error: Don't run 'npm publish' in root. Use 'npm run publish' instead.\" && exit 1"
},
```

I'm sure there are many drawbacks to what I've done, and a lot I don't understand about ES modules, Babel configuration and compilation, and more.
But for now it's an acceptable result.

Finally my basic workflow was updated as follows:

```sh {linenos=false}
# 1. Make changes to the package source

# 2. Typecheck and lint
% npm run tc
% npm run lint

# 3. Build the package
% npm run build

# 4. Create a tarball for testing the package locally with custom script
% npm run pack

# 5. Install the package tarball to the project where it will be included and tested
% npm install ./dpwiese-react-native-canvas-charts-0.0.0.tgz

# 6. Bump the version in package.json and rebuild

# 7. Login and publish the package with custom script
% npm login
% npm run publish
```

# Using the Package

Using the package in my React Native demo app was as easy as running the following install commands:

```sh {linenos=false}
% npm i --save react-native-webview @dpwiese/react-native-canvas-charts
% cd ios && pod install
```

Note the need to install `react-native-webview` as a dependency.
While this is a dependency of `@dpwiese/react-native-canvas-charts`, I haven't yet figured out how to make autolinking work for dependencies which themselves have native dependencies, as is the case here.
The workaround for now is to require `react-native-webview` be installed along with `@dpwiese/react-native-canvas-charts` which isn't so bad.

This seemed like a problem many others would have already encountered and thus have a straightforward solution.
I only found resources like the Stack Overflow post [React-Native autolink a dependency of a dependency](https://stackoverflow.com/questions/60707869/react-native-autolink-a-dependency-of-a-dependency) and [How do I add a react-native library containing native code as a dependency in my library?](https://github.com/callstack/react-native-builder-bob#how-do-i-add-a-react-native-library-containing-native-code-as-a-dependency-in-my-library) which weren't much help.

After installing the package, it can be used as shown below.
See the package's` [README.md](https://github.com/dpwiese/react-native-canvas-charts/blob/master/README.md) for more information.

```jsx
import { Chart, SetData } from "@dpwiese/react-native-canvas-charts/ChartJs";
import { useRef } from "react";
import { chartConfig } from "./chartConfig";

export default () => {
  const setDataRef = useRef<SetData>();

  // Update the charted data with newData
  setDataRef.current.setData(newData);

  return (<Chart config={chartConfig} ref={setDataRef}/>);
}
```
