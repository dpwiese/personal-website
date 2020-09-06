---
title: "React Native with Gomobile on Mac Catalyst"
date: 2020-09-05
draft: true
toc: false
images: ["img/posts/react-native-gomobile/og-image.png"]
tags:
  - reactnative
  - programming
  - go
  - mac catalyst
keywords: [reactnative, programming, go, mac catalyst]
description: "This post describes how to update your Xcode frameworks built with gomobile to support targeting mac catalyst."
---

# Introduction

In my post [Using Go in React Native](/posts/react-native-gomobile/), originally given at a talk in February 2019, I give an overview of building frameworks for iOS apps using Go, and using them in a React Native application.
Shortly after, at WWDC 2019, [Mac Catalyst](https://developer.apple.com/mac-catalyst/) was announced, allowing iOS apps to be built for and run on Mac.
I found this quite exciting, but as the majority of my mobile application development was in React Native, and used frameworks built with [Go Mobile](https://github.com/golang/mobile), and Bluetooth, I needed to make sure each of these three elements would not cause any issues when targeting Catalyst.

# Targeting Catalyst from Bare React Native App

Accomplishing this was straighforward enough, with only one dependency causing trouble.
With `npx react-native init` used to create a new app, at the time in React Native 0.62.0, an attempt to deploy to Catalyst gave the following error:

```txt {linenos=false}&nbsp;
/ios/Pods/Headers/Private/Flipper-Folly/folly/portability/Time.h:51:17:
    Typedef redefinition with different types ('uint8_t' (aka 'unsigned char') vs 'enum clockid_t')
```

This issue is described in [flipper/issues/834](https://github.com/facebook/flipper/issues/834) and [react-native/issues/27845](https://github.com/facebook/react-native/issues/27845), with the obvious solution to simply remove Flipper from the project, or patching [folly/portability/Time.h](https://github.com/facebook/folly/blob/e3ed6d7c878e02157af67b3e037a7248398492aa/folly/portability/Time.h) as described [here](https://github.com/facebook/flipper/issues/834#issuecomment-605448785).
Nevertheless, the inclusion of Flipper when targeting catalyst was not important to me, so I accepted these solutions and moved forward.

# Using Bluetooth

Using Bluetooth in React Native has been made very easy with the [react-native-ble-plx](https://github.com/polidea/react-native-ble-plx) Bluetooth Low Energy library.
I've used it for years, and think very highly of the team at [Polidea](https://www.polidea.com) who have built it.
I expected Bluetooth support to transition well to Catalyst, as the [Core Bluetooth](https://developer.apple.com/documentation/corebluetooth) framework around which `react-native-ble-plx` is built is available for both iOS and Catalyst.
After installing, an attempt to target Catalyst produced the following error:

<!--
TODO: ALLOW SCROLLING WHEN LINE NUMBERS ARE DISABLED
-->
```txt {linenos=false}&nbsp;
Undefined symbol: Swift._stdlib_isVariantOSVersionAtLeast(Builtin.Word, Builtin.Word, Builtin.Word) -> Builtin.Int1

Undefined symbols for architecture x86_64:
  "Swift._stdlib_isVariantOSVersionAtLeast(Builtin.Word, Builtin.Word, Builtin.Word) -> Builtin.Int1", referenced from:
      react_native_ble_plx_swift.Peripheral.mtu.getter : Swift.Int in libreact-native-ble-plx-swift.a(BleExtensions.o)
      react_native_ble_plx_swift.RxCBPeripheral.canSendWriteWithoutResponse.getter : Swift.Bool in libreact-native-ble-plx-swift.a(RxCBPeripheral.o)
      react_native_ble_plx_swift.RxCBPeripheral.(InternalPeripheralDelegate in _DDA244B8664EBFFBFFA88AFB2D25DF21).peripheralIsReady(toSendWriteWithoutResponse: __C.CBPeripheral) -> () in libreact-native-ble-plx-swift.a(RxCBPeripheral.o)
ld: symbol(s) not found for architecture x86_64

clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

This error came from a few calls to the `#available(iOS <ios_version>, *)` during compilation that would, for example, set the maximum MTU size compatible with the particular iOS version.
This _availability condition_ is used to to determine the availability of APIs at runtime, and is very well described in the post [Swift API Availability](https://nshipster.com/available/#available-1) from [NSHipster](https://nshipster.com).
The Apple Developer docs, [Marking API Availability in Objective-C](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/marking_api_availability_in_objective-c#overview), also provides an overview, stating:

> In Swift, you use the @available attribute to control whether a declaration is available to use when building an app for a particular target platform. Similarly, you use the availability condition #available to execute code conditionally based on required platform and version conditions.

I'm not sure why when targeting Catalyst this symbol was not defined as some posts such as [this](https://samwize.com/2020/06/01/issues-with-macos-catalyst/) do seem to indicate it _can_ be used for availability checks on Catalyst, so presumably it should compile.
The post [Forward-compatibility of @available?](https://forums.swift.org/t/forward-compatibility-of-available/27955) on the Swift forums was a bit unrelated, but an interesting read.
Perhaps some compiler settings could be changed to solve the issue, but I wasn't able to find any clues as to why it wouldn't work in the first place.
In the article [Creating a Mac Version of Your iPad App](https://developer.apple.com/documentation/xcode/creating_a_mac_version_of_your_ipad_app) it states

> If you have source code referencing APIs unavailable to the Mac version of your app, enclose the code in a compilation conditional block that uses the `targetEnvironment():` platform condition.

without mentioning the potential use of the run time `#available` check when targeting Catalyst.
In any case, it was simple enough to wrap the few occurances of offending `#available` as suggested:

```swift {linenos=false}&nbsp;
 #if !targetEnvironment(macCatalyst)
 // Code to exclude from Mac.
 #endif
```

The pull request with these couple of changes to enable `react-native-ble-plx` to work when targeting Catalyst is [here](https://github.com/Polidea/MultiPlatformBleAdapter/pull/54/files).

Finally, with these changes, and after setting the entitlments by checking Bluetooth in App Sandbox under Signing & Capabilities, the React Native application was running on my Mac via Catalyst and communicating via BLE.

<img src="/img/posts/react-native-gomobile-catalyst/bluetooth-sandbox.png" width="600" style="border-style:solid;border-width:1px"/>

The next and final step was to include the frameworks I'd previously built using [Go Mobile](https://github.com/golang/mobile) and ensure they worked with Catalyst.
