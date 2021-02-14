---
title: "Canvas Charts in React Native"
date: 2021-02-06T15:30:00-04:00
draft: false
toc: false
images: ["img/posts/react-native-canvas-charts/og-image.png"]
tags: 
  - programming
  - react native
  - data visualization
  - canvas
keywords: [programming, react native, data visualization, canvas]
description: "This post describes a simple implementation of wrapper around a WebView to use the canvas-based plotting library Chart.js in React Native."
---

<img src="/img/posts/react-native-canvas-charts/iphones.png" width="800" />

# Introduction

Charting libraries in React Native aren't well suited for plotting lots of data or streaming data at a high frequency.
Common libraries such as [Victory Native](https://formidable.com/open-source/victory/docs/native/), [react-native-svg-charts](https://github.com/JesperLekland/react-native-svg-charts), and [react-native-pathjs-charts](https://github.com/capitalone/react-native-pathjs-charts) are mostly SVG based, and performance is very bad for anything other than displaying a static chart of a hundred or so points.

[React Native Charts Wrapper](https://github.com/wuxudong/react-native-charts-wrapper) is a wrapper around the native charting libraries [MPAndroidChart](https://github.com/PhilJay/MPAndroidChart) for Android and [Charts](https://github.com/danielgindi/Charts) for iOS, the latter of which I'd enjoyed using in a native iOS project some years ago.
I expect performance is significantly better than the SVG options, but at the expense of additional installation steps due to the use of native code.
In addition, matching styling of these charts on other platforms (e.g. web) would add effort if consistency was desired.
I really wanted to find a solution that minimized the use of native code, could share styling and configuration with a web-based implementation, and was performant enough to plot thousands of data points simultaneously, and plot data streamed at 60 FPS.

Given the above constraints, it seemed to leave two main options.

The first is charting using SVG Elements Directly.
[Building SVG Line Charts in React](https://www.headway.io/blog/building-a-svg-line-chart-in-react) is a detailed blog post that describes how to do this.
This approach can be extended to React Native with [react-native-svg](https://github.com/react-native-svg/react-native-svg).
It seemed possible to get performance gains over existing SVG based libraries by creating a custom solution, although it wouldn't be particularly easy and would still be subject to the same limitations as the existing SVG-based plotting libraries.
One user in the Reddit post [Is it possible for react native, to render a line chart every 10-15ms to stream real time data (e.g. Sensor Data)?](https://www.reddit.com/r/reactnative/comments/j55w1b/is_it_possible_for_react_native_to_render_a_line/) seemed to have good success.
However, it wasn't obvious that this approach would meet the performance requirements that I was looking for.

The second approach is to use one of the many canvas-based charting libraries.
I've used several of these before on the web and found the performance to be very good, supporting plotting well into the thousands of points.
Being able to adapt one of these libraries to React Native would make the generation of consistent plots across web and mobile easier.
**Using canvas charts in React Native was the approach I decided to take.**

There were a few implementations such as [clchart](https://github.com/seerline/clchart) and [react-native-chartjs](https://github.com/KittyCookie/react-native-chartjs), both of which were small projects that didn't seem to be widely used or actively maintained.
I decided to use [Chart.js](https://www.chartjs.org) which I'd used before with success and implement it inside of a React Native `WebView`.
It's not the most performant of the canvas-based charting libraries, but is decent and easy enough to use.
The implementation below should be easily extensible to support other canvas-based plotting libraries in React Native.
*Note that the sample code below uses Chart.js 3, which is still in Beta at the time of this writing.*

# Minimal Implementation

The only dependency was [react-native-webview](https://github.com/react-native-webview/react-native-webview) to allow the `canvas` element to be created in the web view.
This did require native code, but the installation process was very easy.
Within moments I was able to copy-and-paste a simple sample chart from the web and generate a chart with thousands of points in a React Native app using the code below.

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
          <meta name="viewport" content="width=device-width, initial-scale=1.0" />
          <script src="https://cdn.jsdelivr.net/npm/chart.js@3.0.0-beta.10"></script>
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

However, this initial minimal implementation has many drawbacks.
The CDN (and thus internet connectivity) is relied to fetch Chart.js, there is a massive multiline template literal of HTML cluttering up the component, and data is not easily passed to the component and plotted.

# Improved Implementation

As a first step, the multiline template literal of HTML can be moved to another file, at least tidying up the component:

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

Within `index.html` the Chart.js script as fetched from the CDN can be replaced with its contents, ensuring the component will work without internet connectivity:

```html
<!-- index.html -->

<html>
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
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

**I've not yet found a way to import Chart.js directly into this HTML.**
That the HTML can be specified as a string in the `WebView` props as `source={{ html: '<html>' }}` offered one possible solution.
In this case, the HTML string can be specified as a string template, and string interpolation used to insert some JavaScript.
However, this presents difficulties when the inserted JavaScript uses backticks, and still requires the JavaScript to be inserted live somewhere as a string.
Ideally, in the same way that the HTML can be `require`d and passed to the `WebView`'s `source` prop, additional `require`d JavaScript could be passed as additional props.
Other than that the Chart.js dependency has been copy-pasted into the otherwise empty `index.html` file, being committed to the projects source and causing a minor inconvenience when updating the library version, the solution overall isn't too terrible.

The rest of the HTML is also pulled out into component code and the config passed in there, thus allowing the chart to be configured as desired without needing to modify `index.html`.
This includes the creating of the `canvas` element that needs to exist in `index.html`.
The `WebView`'s `ref` is then used to inject the JavaScript to create the `canvas` element and a new `Chart` with the chart configuration.
The updated `index.html` shown above can then be included and used:

```jsx
// App.js

import { WebView } from "react-native-webview";
const chartJsHtml = require("./index.html");

const config = {
  // chart config here
};

let webref;

const addChart = () => {
  webref.injectJavaScript(`const canvasEl = document.createElement("canvas");
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

Because data is written to the `WebView` as a JavaScript string, when passing data or a chart configuration object, `JSON.stringify()` needs to be called on it.
Now the implementation is getting more manageable.
`index.html` is basically an empty file with the Chart.js library pasted in, and the chart configuration is conveniently defined as an object in the component where the chart is.

To further keep the component uncluttered, and given the relatively large size of the Chart.js configuration object, the config can be moved to a separate file and imported.
This can be taken a step further by wrapping the `WebView` in a custom `ChartJs` component so that the chart config can be conveniently passed in as a prop:

```jsx
// ChartJs.js

import { WebView } from "react-native-webview";
const chartJsHtml = require("./index.html");

export const ChartJs = (props) => {
  let webref;

  const addChart = (config) => {
    webref.injectJavaScript(`const canvasEl = document.createElement("canvas");
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
This provides very basic implementation of a wrapper around a React Native `WebView` to use the Chart.js canvas-based library.
**The result is a convenient component that can simply receive the chart config as a prop and plot the contained data.**

# Real-time Plotting

However, if the chart data needs to be frequently updated, the only way to do that is via this `config` prop, causing re-render each time the plot needs to be updated.
This is not a performant approach to update the plot at 60 FPS.

## Updating Chart data with Ref

Exposing a ref to the parent component can be used to draw to the `canvas` element without requiring a re-render of the child each time.
Because the component wrapping the `WebView` is a function component, reading the React Docs [Refs and Function Components](https://reactjs.org/docs/refs-and-the-dom.html#refs-and-function-components) gives some relevant information:

> If you want to allow people to take a ref to your function component, you can use `forwardRef` (possibly in conjunction with `useImperativeHandle`), or you can convert the component to a class.

See the React Docs on [useRef](https://reactjs.org/docs/hooks-reference.html#useref), [Forwarding Refs](https://reactjs.org/docs/forwarding-refs.html) and [useImperativeHandle](https://reactjs.org/docs/hooks-reference.html#useimperativehandle) for more information.
The use of `useImperativeHandle` allows `setData` to be called by the parent component that renders `ChartJs` to update the plotted data without re-rendering `ChartJs`.

```jsx
// ChartJs.js

const chartJsHtml = require("./index.html");

export const ChartJs = forwardRef((props, ref) => {
  let webref;

  const addChart = (config) => {
    webref.injectJavaScript(`const canvasEl = document.createElement("canvas");
      document.body.appendChild(canvasEl);
      window.canvasLine = new Chart(canvasEl.getContext('2d'), ${JSON.stringify(config)});`);
  };

  const setData = (dataSets) => {
    if (dataSets) {
      dataSets.forEach((_, i) => {
        webref.injectJavaScript(`window.canvasLine.config.data.datasets[${i}].data = ${JSON.stringify(dataSets[i])};
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

The `ChartJs` component above with this ref can then be used to plot new data without needing to pass new `config` prop and triggering a re-render:

```jsx
// App.js

import { ChartJs } from "./ChartJs";
import { config } from "./config";

// Use ChartJs component in another component
export default () => {
  const setDataRef = useRef();

  // Pass setDataRef.current.setData() a valid Chart.js data.datasets.data array
  //  to update the plotted data without re-rendering the ChartJs component
  setDataRef.current.setData(newData);

  return (
    <ChartJs config={config} ref={setDataRef} />
  );
}
```

# Fixing Jest

While the above implementation was working well, Jest will break as it will try to parse the HTML file (which lives within `src`) as JavaScript, giving the following error.

```sh {linenos=false}
Jest encountered an unexpected token

This usually means that you are trying to import a file which Jest cannot parse, e.g. it's not plain JavaScript.
```

This was a quick fix to mock the HTML file as an empty object.
Create the file `<rootDir>/__mocks__/fileMock.js` with the following contents:

```js
module.exports = {};
```

and in `jest.config.js` use this mock:

```js
module.exports = {
  moduleNameMapper: {
    "\\.(html)$": "<rootDir>/__mocks__/fileMock.js",
  }
}
```

This at least offers a solution to fix failing tests simply due to the presence of the HTML file.
It may be necessary to find a better solution in the future if tests are actually added to the `ChartJs` component.

# Conclusion

This post provided a minimal implementation of the canvas-based plotting library Chart.js in React Native.
I hope to find a more elegant solution for including JavaScript dependencies that are needed within the `WebView`, but for now I am reasonably satisfied with this solution.
