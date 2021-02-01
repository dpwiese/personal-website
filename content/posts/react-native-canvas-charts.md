---
title: "React Native Canvas Charts"
date: 2021-01-31T21:00:00-04:00
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

# React Native ChartJS Webview

## Introduction

Charting options in React Native aren't well suited for plotting lots of data, or streaming data at a high frequency.
The common existing charting libraries such as [Victory Native](https://formidable.com/open-source/victory/docs/native/), [react-native-svg-charts](https://github.com/JesperLekland/react-native-svg-charts), and [react-native-pathjs-charts](https://github.com/capitalone/react-native-pathjs-charts) are mostly SVG based, and performance is very bad for anything other than displaying a static chart of a hundred or so points.
[React Native Charts Wrapper](https://github.com/wuxudong/react-native-charts-wrapper) is a wrapper around the native charting libraries [MPAndroidChart](https://github.com/PhilJay/MPAndroidChart) for Android and [Charts](https://github.com/danielgindi/Charts) for iOS, the latter of which I'd enjoyed using in a native iOS project some years ago.
I expect performance is better, but at the expense of additional installation steps due to the use of native code.
In addition, matching styling of these charts on other platforms (e.g. web) would add effort if consistentcy was desired.
I really wanted to find a JavaScript only solution that didn't require any native code, could share styling and configuration with a web-based implementation, and was performant enough to plot thousands of data points simultaneously, and plot data streamed at 60 Hz.

Given the above constraints, it seemed to leave two main options.

The first is charting using SVG Elements Directly.
See the following resources.
[Building SVG Line Charts in React](https://www.headway.io/blog/building-a-svg-line-chart-in-react) is a detailed blog post that describes how to do this.
The approach in the above post can be extended to React Native with [react-native-svg](https://github.com/react-native-svg/react-native-svg).
It seemed possible to get performance gains over existing SVG based libraries by creating a custom solution, although it wouldn't be particularly easy.
One user in the Reddit post [Is it possible for react native, to render a line chart every 10-15ms to stream real time data (e.g. Sensor Data)?](https://www.reddit.com/r/reactnative/comments/j55w1b/is_it_possible_for_react_native_to_render_a_line/) seemed to have good success.
However, it wasn't obvious that this approach would meet the performance requirements that I was looking for.

The second approach is to use one of the many canvas-based charting libraries.
I've used several of these before on the web and found the performance to be very good, supporting plotting well into the thousands of points.
Being able to adapt one of these libraries to React Native would make the generation of consistent plots across web and mobile easier.
And it seemed there were existing libraries that brought canvas support to React Native.
**Using canvas charts in React Native was the approach I decided to take.**

There were a few implementations such as [clchart](https://github.com/seerline/clchart) and [react-native-chartjs](https://github.com/KittyCookie/react-native-chartjs), both of which were small projects that hadn't been updated in a long time.
I decided to use [Chart.js](https://www.chartjs.org) which I'd used before with success.
It's not the most performant of the canvas-based charting libraries, but is decent and easy enough to use.

## Implementation

The only dependency was [react-native-webview](https://github.com/react-native-webview/react-native-webview) to allow the canvas element to be created in the web view.
Within minutes I was able to generate a chart with thousands of points using the code below.
This implementation could just as easily be pasted and opened in a browser with no modifications.

```jsx
import { WebView } from "react-native-webview";

return (
  <WebView
    originWhitelist={["*"]}
    source={{
      html: `
      <html>
        <head>
          <script src="https://cdn.jsdelivr.net/npm/chart.js@3.0.0-beta.8"></script>
        </head>
        <body>
          <canvas id="canvasId" height="200"></canvas>
          <script>
            window.onload = function() {
              const chartConfig = {
                // chart config here
              };
              const canvasEl = document.getElementById('canvasId');
              window.canvasLine = new Chart(canvasEl.getContext('2d'), chartConfig);
            }
          </script>
        </body>
      </html>
      `,
    }}
  />
);
```

However, this minimal implementation has many drawbacks.
We're relying on the CDN to fetch Chart.js, we have a massive multiline string of HTML cluttering up our component, and we aren't able to pass data through the component props to chart it.
As a first step, we can remove the multiline string to another file, at least tidying up the component.

```jsx
const chartJsHtml = require("./index.html");

return (
  <WebView
    originWhitelist={["*"]}
    source={chartJsHtml}
  />
)
```

In addition, within `index.html` the Chart.js script fetched from the CDN can be replaced with it's contents.
*I've not yet found a way to import Chart.js directly into this HTML.

```html
<script>
  /*!
   * Chart.js v3.0.0-beta.8
   * https://www.chartjs.org
   * (c) 2021 Chart.js Contributors
   * Released under the MIT License
   */
  !function(t,e){"object"==typeof exports&&"undefined"!=typeof module?module.exports=e(): // ...
  /*!
   * @kurkle/color v0.1.9
   * https://github.com/kurkle/color#readme
   * (c) 2020 Jukka Kurkela
   * Released under the MIT License
   */
  function Me(t){var e=function(t){return ve(t.r)&&ve(t.g)&&ve(t.b)&&ve(t.a)}(t)?_e:ye; // ...
</script>
```

The rest of the HTML can be pulled out into component code, and config passed in.
We can also create the canvas element that needs to exist in `index.html`, reducing requirements for what needs to be in that file.

```jsx
const chartJsHtml = require("./index.html");

const config = {
  // chart config here
};

const addChart = () => {
  webref?.injectJavaScript(`const canvasEl = document.createElement("canvas");
    document.body.appendChild(canvasEl);
    window.canvasLine = new Chart(canvasEl.getContext('2d'), ${JSON.stringify(config)});`);
};

<WebView
  originWhitelist={["*"]}
  source={chartJsHtml}
  onLoadEnd={addChart}
/>
```

Of course to further keep the component uncluttered, and given the relatively large size of Chart.js configuration, the config can be moved to a separate file and imported.

```js
import { config } from "./config";
```

However now I had to break this HTML file apart, to pull chart configuration and Chart.js source code out, and be able to pass data in from React Native.

Data is written to the webview as a JavaScript string, so when passing data or a chart configuration object, we just need to call `JSON.stringify()` on it.

### Updating Chart data with Ref

Data needed to be able to updated in the canvas chart.

Does this cause rerender?


## Testing and Other Configuration

With the `html` file now in our `src` directory, we'll also break Jest, as it will try to parse the file as JavaScript, giving the following error.

```jsx
Jest encountered an unexpected token

This usually means that you are trying to import a file which Jest cannot parse, e.g. it's not plain JavaScript.
```


`<rootDir>/__mocks__/fileMock.js` with the following contents:

```js
module.exports = {};
```

and in `jest.config.js` add:

```js
module.exports = {
  moduleNameMapper: {
    "\\.(html)$": "<rootDir>/__mocks__/fileMock.js",
  }
}
```


# TODO

```js
// TODO@dpwiese
//  - Import Chart.js from separate .js source file without copy-paste
//  - Type Chart.js configuration (instead of any)
//  - Test on other environments (simulator, Android, iPhone)
//  - Test and quantify performance
```

* Where/when to check types given Babel is used to compile?
https://stackoverflow.com/questions/55002137/typescript-noemit-use-case

* How/why to generate `.d.ts` files?
https://www.typescriptlang.org/docs/handbook/babel-with-typescript.html


https://www.typescriptlang.org/docs/handbook/declaration-files/dts-from-js.html


[React-Native autolink a dependency of a dependency](https://stackoverflow.com/questions/60707869/react-native-autolink-a-dependency-of-a-dependency)


****
THIS CREATES FONT FILES?! IS THAT GOOD?

npx react-native link 

****



THIS WAS FIX FOR JEST CONFIG:

https://github.com/facebook/jest/issues/6229#issuecomment-433632564

# Resources

Testing node module before publishing
[Testing npm packages before publishing](https://dev.to/vcarl/testing-npm-packages-before-publishing-h7o)
[npm-link](https://docs.npmjs.com/cli/v6/commands/npm-link)

Basically:

```sh
% npm pack
% npm install ./dpwiese-react-native-canvas-charts-0.0.0.tgz
```

```sh
% npm login
% npm publish
```

[TypeScript With Babel: A Beautiful Marriage](https://iamturns.com/typescript-babel/)

[Main property in package.json defines package entry point](https://bytearcher.com/articles/main-property-in-package.json-defines-entry-point/)

See this file:
https://github.com/KittyCookie/react-native-chartjs/blob/master/index.js


This was the best post for how to call ref from child to parent:
[Call child method from parent](https://stackoverflow.com/questions/37949981/call-child-method-from-parent)


https://cdn.jsdelivr.net/npm/chart.js@3.0.0-beta.8/dist/chart.js

https://stackoverflow.com/questions/55471314/react-native-webview-load-js-files/55493539

https://stackoverflow.com/questions/55484740/how-to-append-extension-in-metro-config-js-for-metro-bundler

Can’t add JS extension to Assets:
https://github.com/expo/expo-cli/issues/1507


A solution for including HTML files (I don’t have this issue currently, but maybe it’s a better solution):
https://github.com/react-native-webview/react-native-webview/issues/428#issuecomment-642458997


Example passing config with message. Look back at this when cleaning up how to pass Chartjs config:
https://snack.expo.io/@samithaf/react-native-web-view


https://stackoverflow.com/questions/59171876/react-native-webview-with-local-javascript
https://github.com/react-native-webview/react-native-webview/blob/master/docs/Guide.md#loading-local-html-files


With state of react-native-webview, need to either:
1. Copy-paste JS into single source HTML file
2. Get JS from web (either CDN or own hosting)
3. Run local webserver on RN?


Types for Chart.js v2:
https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/chart.js/index.d.ts


https://dev.to/fdefreitas/how-to-obtain-a-uri-for-an-image-asset-in-react-native-with-expo-7bm

https://www.pluralsight.com/guides/how-to-send-state-of-current-component-as-a-parameter-to-another-external-method-using-react

On typing import of html file:
https://blog.atomist.com/typescript-imports/
