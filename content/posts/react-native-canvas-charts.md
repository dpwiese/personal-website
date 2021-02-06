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

# Introduction

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

There were a few implementations such as [clchart](https://github.com/seerline/clchart) and [react-native-chartjs](https://github.com/KittyCookie/react-native-chartjs), both of which were small projects that didn't seem to be widely used or actively maintained.
I decided to use [Chart.js](https://www.chartjs.org) which I'd used before with success.
It's not the most performant of the canvas-based charting libraries, but is decent and easy enough to use.

# Minimal Implementation

The only dependency was [react-native-webview](https://github.com/react-native-webview/react-native-webview) to allow the canvas element to be created in the web view.
Within minutes I was able to generate a chart with thousands of points using the code below.
This implementation could just as easily be pasted and opened in a browser with no modifications.

```jsx
// App.js

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

# Improved Implementation

The first step is to remove the large multiline template literal to a separate file, thus cleaning up the React component and giving something like this:

```jsx
// App.js

import { WebView } from "react-native-webview";
const chartJsHtml = require("./index.html");

return (
  <WebView
    originWhitelist={["*"]}
    source={chartJsHtml}
  />
)
```

In addition, within `index.html` the Chart.js script as fetched from the CDN can be replaced with it's contents:

```html
<!-- index.html -->

<html>
  <head>
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
  </head>
  <body>
  </body>
</html>
```
This ensures the component will work without internet connectivity.

*I've not yet found a way to import Chart.js directly into this HTML.
Given the HTML can be specified as a string by specifying `source={{ html: '<html>' }}`, the HTML string can be specified as a string template, and string interpolation used to insert some JavaScript.
However, this doesn't work when the inserted JavaScript uses backticks.*

In addition, note that the rest of the HTML is pulled out into component code and the config passed in there, thus allowing the chart to be configured as desired without needing to modify `index.html`.
This includes the creating of the `canvas` element that needs to exist in `index.html`.
The `WebView`'s `ref` is then used to inject the JavaScript to create the `canvas` element and a new `Chart` with the chart configuration.
Using this updated `index.html` would look like:

```jsx
// App.js

import { WebView } from "react-native-webview";
const chartJsHtml = require("./index.html");

const config = {
  // chart config here
};

let webref;

const addChart = () => {
  webref?.injectJavaScript(`const canvasEl = document.createElement("canvas");
    document.body.appendChild(canvasEl);
    window.canvasLine = new Chart(canvasEl.getContext('2d'), ${JSON.stringify(config)});`);
};

<WebView
  originWhitelist={["*"]}
  ref={(r) => (webref = r)}
  source={chartJsHtml}
  onLoadEnd={addChart}
/>
```

Because data is written to the `WebView` as a JavaScript string, when passing data or a chart configuration object, we just need to call `JSON.stringify()` on it.
Of course to further keep the component uncluttered, and given the relatively large size of Chart.js configuration, the config can be moved to a separate file and imported.
And of course we can take this a step further by wrapping the `WebView` in our own `ChartJs` component so that the chart config can be conveniently passed in as a prop:

```jsx
// ChartJs.js

import { WebView } from "react-native-webview";
const chartJsHtml = require("./index.html");

export const ChartJs = (props) => {
  let webref;

  const addChart = (config) => {
    webref?.injectJavaScript(`const canvasEl = document.createElement("canvas");
      document.body.appendChild(canvasEl);
      window.canvasLine = new Chart(canvasEl.getContext('2d'), ${JSON.stringify(config)});`);
  };

  return (
    <WebView
      originWhitelist={["*"]}
      ref={(r) => (webref = r)}
      source={chartJsHtml}
      onLoadEnd={() => { addChart(props.config) }}
    />
  );
}
```

This can then be conveniently used as:

```jsx
// App.js

import { ChartJs } from "./ChartJs";
import { config } from "./config";

// Use ChartJs component in another component
export default () => {
  return (
    <ChartJs config={config} />
  );
}
```

This custom component can accommodate style props as well.
This pretty much wraps up a very basic implementation of a wrapper around a React Native `WebView` to use the Chart.js `canvas`-based library.
**We now have a convenient component that we can simply pass the config to and plot the data.**

# Real-time Plotting

The above implementation is quite basic, but gives a simple `ChartJs` component that can be passed the standard configuration object, and the data plotted within the wrapped `WebView`.
However if the chart data needs to be frequently updated, the only way to do that is via this config object, causing re-render each time the plot needs to be updated.
This is not a performant approach to approach updating the plot at 60 FPS.

## Updating Chart data with Ref

Exposing a ref to the parent component can be used to draw to the `canvas` element without requiring a re-render of the child each time.
See the React Docs on [useRef](https://reactjs.org/docs/hooks-reference.html#useref) to learn more.
Because the component wrapping the `WebView` is a function component, reading the React Docs [Refs and Function Components](https://reactjs.org/docs/refs-and-the-dom.html#refs-and-function-components) gives some relevant information:

> If you want to allow people to take a ref to your function component, you can use `forwardRef` (possibly in conjunction with `useImperativeHandle`), or you can convert the component to a class.

See the React Docs on [Forwarding Refs](https://reactjs.org/docs/forwarding-refs.html) and [useImperativeHandle](https://reactjs.org/docs/hooks-reference.html#useimperativehandle) for more information.
This results in the updated `ChartJs` component below, where `setData` is exposed to allow the plotted data to be set as desired from the parent component without re-rendering the child.

```jsx
// ChartJs.js

const chartJsHtml = require("./index.html");

export const ChartJs = forwardRef((props, ref) => {
  let webref;

  const addChart = (config) => {
    webref?.injectJavaScript(`const canvasEl = document.createElement("canvas");
      document.body.appendChild(canvasEl);
      window.canvasLine = new Chart(canvasEl.getContext('2d'), ${JSON.stringify(config)});`);
  };

  const setData = (dataSets) => {
    if (dataSets) {
      dataSets.forEach((_, i) => {
        webref?.injectJavaScript(`window.canvasLine.config.data.datasets[${i}].data = ${JSON.stringify(dataSets[i])};
        window.canvasLine.update();`);
      });
    }
  };

  useImperativeHandle(ref, () => ({
    setData,
  }));

  return (
    <WebView
      originWhitelist={["*"]}
      ref={(r) => (webref = r)}
      source={chartJsHtml}
      onLoadEnd={() => { addChart(props.config) }}
    />
  );
});
```

The `ChartJs` component with this ref can then be used:

```jsx
// App.js


import { ChartJs } from "./ChartJs";
import { config } from "./config";

// Use ChartJs component in another component
export default () => {
  const setDataRef = useRef();

  // Pass setDataRef.current?.setData() a valid Chart.js data.datasets.data array
  //  to update the plotted data without re-rendering the ChartJs component

  return (
    <ChartJs config={config} ref={setDataRef} />
  );
}
```

# Testing and Other Configuration

With the `html` file now in our `src` directory, we'll also break Jest, as it will try to parse the file as JavaScript, giving the following error.

```sh {linenos=false}
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

# Conclusion

TBD


# Appendix

## TODO

* Import Chart.js from separate `.js` source file without copy-paste into `.html` file.

[React native - webview - load js files](https://stackoverflow.com/questions/55471314/react-native-webview-load-js-files/55493539)

[How to append extension in metro.config.js for Metro Bundler?](https://stackoverflow.com/questions/55484740/how-to-append-extension-in-metro-config-js-for-metro-bundler)

With state of react-native-webview, need to either:
1. Get JS from web (What I did at the beginning)
2. Copy-paste JS into single source HTML file (what I'm doing now)
3. Run local webserver on RN?

Can’t add JS extension to Assets:
https://github.com/expo/expo-cli/issues/1507

Basically tried renaming my js files I wanted to import to `.webviewjs` or some other extension which could be specified and bundled with Metro.

But I think importing them was still an issue?

This was after I had tried to bundle the code with Metro as described above, and then get the URI to the bundled code so I could insert it into the html:
[How to obtain a URI for an image asset in React Native (With Expo)](https://dev.to/fdefreitas/how-to-obtain-a-uri-for-an-image-asset-in-react-native-with-expo-7bm)

## More TODO

* Type Chart.js configuration (instead of any)
* Test on other environments (simulator, Android, iPhone)
* Test and quantify performance

Types for Chart.js v2:
https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/chart.js/index.d.ts

## References (Not to include above)

This was the best post for how to call ref from child to parent:
[Call child method from parent](https://stackoverflow.com/questions/37949981/call-child-method-from-parent)

https://stackoverflow.com/questions/55002137/typescript-noemit-use-case

https://cdn.jsdelivr.net/npm/chart.js@3.0.0-beta.8/dist/chart.js

[Import statements in TypeScript: which syntax to use](https://blog.atomist.com/typescript-imports/)

> You can always const thing = require("Anything"); just like in JS, but you won't get typing. You also won't get compile-time checking that the module is available.

[How to Send State of Current Component as a Parameter to Another External Method Using React](https://www.pluralsight.com/guides/how-to-send-state-of-current-component-as-a-parameter-to-another-external-method-using-react)

## Random

```sh {linenos=false}
# NOTE: THIS CREATES FONT FILES?! IS THAT GOOD?
npx react-native link
```
