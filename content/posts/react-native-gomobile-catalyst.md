---
title: "React Native with Go Mobile on Mac Catalyst"
date: 2020-09-07T23:10:00-04:00
draft: false
toc: false
images: ["img/posts/react-native-gomobile/og-image.png"]
tags:
  - react native
  - programming
  - golang
  - go mobile
  - catalyst
keywords: [reactnative, programming, go, mac catalyst, gomobile, react native gomobile, golang react native, gomobile example, bluetooth]
description: "This post describes how to update your Xcode frameworks built with gomobile to support targeting mac catalyst."
---

<!--
TODO: Fix some long code lines do not wrap
TODO: Improve syntax highlighting style for shell and swift
-->

# Introduction

In my previous post [Using Go in React Native](/posts/react-native-gomobile/), I gave an overview of building iOS frameworks and Android archives using Go, and using them in a React Native application.
During WWDC 2019 [Mac Catalyst](https://developer.apple.com/mac-catalyst/) was announced allowing iOS apps to be built for and run on Mac.
I found this quite exciting, but as the majority of my mobile application development was in React Native and, frameworks built with [Go Mobile](https://github.com/golang/mobile), and used Bluetooth, I needed to make sure each of these three elements would work when targeting Catalyst.
An early discussion [Support Project Catalyst (running iPad apps on macOS)](https://github.com/react-native-community/discussions-and-proposals/issues/131) in [React Native Community](https://github.com/react-native-community) provided some good resources and examples before starting.
**In this post I describe the steps needed to build a React Native app with Go Mobile and Bluetooth for Catalyst.**

# Targeting Catalyst from Bare React Native App

Starting with `npx react-native init` to create a new app, at the time in React Native 0.62.0, to build for Catalyst at minimum you must:

1. Enable the new build system in Xcode: `File` > `Workspace Settings` > `Enable new build system`.

<img src="/img/posts/react-native-gomobile-catalyst/new-build-system.png" width="500" style="border-style:solid;border-width:1px"/>

2. Enable Mac as target.

<img src="/img/posts/react-native-gomobile-catalyst/target-mac.png" width="700" style="border-style:solid;border-width:1px"/>

After performing only the two above steps an attempt to build for Catalyst gave the following error:

<div class="wrap">

```txt {linenos=false}&nbsp;
/ios/Pods/Headers/Private/Flipper-Folly/folly/portability/Time.h:51:17: Typedef redefinition with different types ('uint8_t' (aka 'unsigned char') vs 'enum clockid_t')
```

</div>

This issue is described in [flipper/issues/834](https://github.com/facebook/flipper/issues/834) and [react-native/issues/27845](https://github.com/facebook/react-native/issues/27845), with the obvious solution to simply remove Flipper from the project, or patching [folly/portability/Time.h](https://github.com/facebook/folly/blob/e3ed6d7c878e02157af67b3e037a7248398492aa/folly/portability/Time.h) as described [here](https://github.com/facebook/flipper/issues/834#issuecomment-605448785).
Nevertheless, the inclusion of Flipper when targeting catalyst was not important to me, so I accepted these solutions and moved forward.
I've yet to revisit this for a better solution.

# Using Bluetooth

Using Bluetooth in React Native has been made very easy with the [react-native-ble-plx](https://github.com/polidea/react-native-ble-plx) Bluetooth Low Energy library.
I've used it for years and have had very positive and pleasant interactions with the team at [Polidea](https://www.polidea.com) who have built it.
I expected Bluetooth support to transition well to Catalyst, as the [Core Bluetooth](https://developer.apple.com/documentation/corebluetooth) framework around which `react-native-ble-plx` is built is available for both iOS and Catalyst.
After installing, an attempt to target Catalyst produced the following error:

<div class="wrap">

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

</div>

This error came from a few calls to the `#available(iOS <ios_version>, *)` during compilation that was used to, for example, set the maximum MTU size compatible with the particular iOS version.
This _availability condition_ is used to to determine the availability of APIs at runtime, and is very well described in the post [Swift API Availability](https://nshipster.com/available/#available-1) from [NSHipster](https://nshipster.com).
The Apple Developer docs, [Marking API Availability in Objective-C](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/marking_api_availability_in_objective-c#overview), also provide an overview, stating:

> In Swift, you use the @available attribute to control whether a declaration is available to use when building an app for a particular target platform. Similarly, you use the availability condition #available to execute code conditionally based on required platform and version conditions.

I'm not sure why when targeting Catalyst this symbol was not defined as some posts such as [this](https://samwize.com/2020/06/01/issues-with-macos-catalyst/) do seem to indicate it _can_ be used for availability checks on Catalyst, so presumably it should compile.
The post [Forward-compatibility of @available?](https://forums.swift.org/t/forward-compatibility-of-available/27955) on the Swift forums was a bit unrelated, but an interesting read.
Perhaps some compiler settings could be changed to solve the issue, but I wasn't able to find any clues as to why it wouldn't work in the first place.
In the article [Creating a Mac Version of Your iPad App](https://developer.apple.com/documentation/xcode/creating_a_mac_version_of_your_ipad_app) it states:

> If you have source code referencing APIs unavailable to the Mac version of your app, enclose the code in a compilation conditional block that uses the `targetEnvironment():` platform condition.

without mentioning the potential use of the run time `#available` check when targeting Catalyst.
In any case, it was simple enough to wrap the few occurences of offending `#available` as suggested:

```swift {linenos=false}&nbsp;
 #if !targetEnvironment(macCatalyst)
 // Code to exclude from Mac.
 #endif
```

The pull request with these couple of changes to enable `react-native-ble-plx` to work when targeting Catalyst is [here](https://github.com/Polidea/MultiPlatformBleAdapter/pull/54).

Finally, with these changes, and after setting the entitlements by checking Bluetooth in App Sandbox under Signing & Capabilities, the React Native application was running on my Mac via Catalyst and communicating via BLE.

<img src="/img/posts/react-native-gomobile-catalyst/bluetooth-sandbox.png" width="600" style="border-style:solid;border-width:1px"/>

The next and final step was to include the frameworks I'd previously built using [Go Mobile](https://github.com/golang/mobile) and ensure they worked with Catalyst.

# Go Mobile for Catalyst

Previously, when using Go Mobile with iOS apps, the output was a `.framework` which could be included in an Xcode project.
The exported methods could be easily accessed in React Native via a bridge.
Getting a Go package built with Go Mobile to build for Catalyst ended up being a bit more effort than I'd expected.

## Workflow

I first created a simple symlink between my default git directory and the default go directory.
This allows the source code to be placed as desired, without a relative path specified when running `gomboile bind`.

```sh {linenos=false}&nbsp;
% ln -s ~/Code/dpwiese/gomobile-catalyst/src/sample ~/go/src/github.com/dpwiese/sample
```

Recall the following command which builds, in this case with the `-taret=ios`, a framework for use in Xcode, supporting each of the architectures defined in [cmd/gomobile/env.go](https://github.com/golang/mobile/blob/master/cmd/gomobile/env.go), and working on both iOS and the simulator.

```sh {linenos=false}&nbsp;
% gomobile bind -x -v -target=ios github.com/dpwiese/sample
```

As changes to `gomobile` were necessary to support Catalyst, after editing the source a new version of `gomobile` needed to be built by navigating to the `gomobile` source directory and running `go build`.
This generates a new `gomobile` executable which can be placed in `~/go/bin` from where it can be easily run.

## Initial Attempt

Having previously generated a framework using the above command, a first attempt at building my previous sample application with React Native and Go Mobile [react-native-gomobile-demo](https://github.com/dpwiese/react-native-gomobile-demo) without modification gave the following error.

<div class="wrap">

```txt {linenos=false}&nbsp;
error: Building for Mac Catalyst, but the linked framework 'Sample.framework' was built for iOS + iOS Simulator. You may need to restrict the platforms for which this framework should be linked in the target editor, or replace it with an XCFramework that supports both platforms.
```

</div>

This was not unexpected.
I quickly found the issue [golang/go/issues/36856](https://github.com/golang/go/issues/36856) which proposed setting the compiler flag with `-target x86_64-apple-ios13.0-macabi` to generate a Catalyst compatible framework.
I was not very familiar with the format of this flag, so checking [the LLVM source](https://github.com/llvm/llvm-project) seemed like a good place to start to understand it a bit better.
The `VersionTuple` class in [llvm/ADT/Triple.h](http://llvm.org/doxygen/Triple_8h_source.html) describes the format:

```cpp {linenos=false}&nbsp;
/// Triple - Helper class for working with autoconf configuration names. For
/// historical reasons, we also call these 'triples' (they used to contain
/// exactly three fields).
///
/// Configuration names are strings in the canonical form:
///   ARCHITECTURE-VENDOR-OPERATING_SYSTEM
/// or
///   ARCHITECTURE-VENDOR-OPERATING_SYSTEM-ENVIRONMENT
```

Just a few lines below gave me the answer I was looking for:

```cpp {linenos=false}&nbsp;
enum EnvironmentType {
  ...
  MacABI, // Mac Catalyst variant of Apple's iOS deployment target.
  ...
};
```

The triple format is also described in the LLVM docs [Cross-compilation using Clang](https://clang.llvm.org/docs/CrossCompilation.html#target-triple).
In the case of the "triple" `x86_64-apple-ios13.0-macabi`, the architecture is `x86_64`, vendor is `apple`, operating system `ios13.0`, and environment `macabi`.

Now understanding where this flag had come from, I naively implemented a fix proposed in [golang/go/issues/36856](https://github.com/golang/go/issues/36856) in [golang/mobile/pull/45](https://github.com/golang/mobile/pull/45) to add the flag an build a new framework.
Nominally it worked, but this hasty fix simply overwrote the output for the `x86_64` architecture which is needed for the iOS simulator for that needed for Catalyst.
So with nothing more than these few lines I was able to get the framework working for Catalyst, but at the expense of giving up iOS simulator support, which was not acceptable.
I then had to figure out how to provide support for the `x86_64` architecture for both Catalyst _and_ the iOS simulator.

## Combining Static Archives

Looking under the hood of Go Mobile, each of the different architectures specified in [cmd/gomobile/env.go](https://github.com/golang/mobile/blob/master/cmd/gomobile/env.go) are used to build a corresponding static library which is ultimately combined with [lipo](https://ss64.com/osx/lipo.html) and placed in the `.framework` output, as below:

```sh {linenos=false}&nbsp;
% lipo -create \
-arch armv7 sample-arm.a \
-arch arm64 sample-arm64.a \
-arch i386 sample-386.a \
-arch x86_64 sample-amd64.a \
-arch x86_64 sample-catalyst.a \
-o Sample.framework/Versions/A/Sample
```

It is simple enough to create an additional and separate `amd64` static archive with the flags needed for catalyst, but multiple archives of the same architecture cannot be combined with `lipo`.
Given that `catalyst` is not a separate architecture, this presents some obvious difficulties that I did not have experience with and was not yet sure how to solve.
Attempting to combine two static archives of the same architecture with `lipo` gives:

<div class="wrap">

```txt {linenos=false}&nbsp;
fatal error: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/lipo: /var/folders/5f/26gs6d3n11jfxldns9vqhxn40000gp/T/gomobile-work-105957311/sample-amd64.a and /var/folders/5f/26gs6d3n11jfxldns9vqhxn40000gp/T/gomobile-work-105957311/sample-catalyst.a have the same architectures (x86_64) and can't be in the same fat output file
```

</div>

I started looking into how to combine two static archives of the same architecture, each with different flags needed for the iOS simulator and catalyst.
Two tools that seemed like they may be helpful to solve this problem were [libtool](https://www.gnu.org/software/libtool/) and [ar](https://linux.die.net/man/1/ar).
I hadn't used them before, nor do I have strong knowledge of compilers, or building and modifying archives and libraries.
My first attempts resulted in terrors such as the one below when attempting to use the generated `.framework`:

<div class="wrap">

```txt {linenos=false}&nbsp;
In ios/Sample.framework/Sample(000006.o), building for Mac Catalyst, but linking in object file built for iOS Simulator, file 'ios/Sample.framework/Sample' for architecture x86_64
```

</div>

In my short time working with these tools I think this approach should work - I don't expect there is a fundamental limitation to including multiple static archives of the same architecture but built with different flags in a `.framework`.
To solve this problem I probably just need to spend some time learning `libtool`/`ar` better, as it's probably a case of user error to correctly combine the static archives.
I do hope to revisit this at some point, but for now it turned out there was perhaps a more modern alternative to what I had been attempting to do with `lipo`.

## Enter XCFramework

The error from my initial attempt [above](http://localhost:1313/posts/react-native-gomobile-catalyst/#initial-attempt) suggested replacing the framework `with an XCFramework that supports both platforms.`
[This comment](https://github.com/golang/mobile/pull/45#issuecomment-651183803) suggested the same.
XCFrameworks were introduced WWDC2019 and in Xcode 11, and I also hadn't previously used them.
There didn't seem to be as many resources on XCFrameworks as I'd expected, but the WWDC 2019 video [Binary Frameworks in Swift](https://developer.apple.com/videos/play/wwdc2019/416/), the Apple Developer article [Distributing Binary Frameworks as Swift Packages](https://developer.apple.com/documentation/swift_packages/distributing_binary_frameworks_as_swift_packages), and the Xcode Help article [Create an XCFramework](https://help.apple.com/xcode/mac/11.4/#/dev544efab96) were useful.
I made a quick modification to Go Mobile to output individual `.framework`s in a draft PR [dpwiese/mobile/pull/1](https://github.com/dpwiese/mobile/pull/1) enabling the use of `xcodebuild -create-xcframework` from these individual frameworks, instead of `lipo -create` from the individual static archives:

```sh {linenos=false}&nbsp;
% xcodebuild -create-xcframework \
-framework arm/Sample.framework \
-framework arm64/Sample.framework \
-framework 386/Sample.framework \
-framework amd64/Sample.framework \
-framework catalyst/Sample.framework \
-output Sample.xcframework
```

A first attempt at using this command gave:

```txt {linenos=false}&nbsp;
error: unable to find any architecture information in the binary at '~/Desktop/arm64/Sample.framework/Sample'
```

This was solved by thinning the `arm64` framework with `lipo`:

```sh {linenos=false}&nbsp;
% lipo arm64/Sample.framework/Versions/A/Sample -thin arm64 -output arm64/Sample.framework/Versions/A/Sample
```

After thinning a second attempt with `xcodebuild -create-xcframework` gave another error:

```txt {linenos=false}&nbsp;
Both ios-armv7 and ios-arm64 represent two equivalent library definitions.
```

I wasn't able to find much information as to why these two frameworks of different architectures were being reported as equivalent, but a pull request in [firebase-ios-sdk/pull/4737](https://github.com/firebase/firebase-ios-sdk/pull/4737/files#diff-ae87db694a268f93c6029c8b91a3b9edR333) gave some information.
A comment said:

> xcframework doesn't support legacy architectures: armv7, i386.
> It will throw a "Both ios-arm64 and ios-armv7 represent two equivalent library definitions" error.

with some following discussion indicating that while tricky, 32 bit support should be possible:

<img src="/img/posts/react-native-gomobile-catalyst/firebase-ios-sdk-discussion.png" width="900" style="border-style:solid;border-width:1px"/>

The comment about this being woefully under-documented still seems to be true.
As I was unable to find much more useful information about this issue, I reached out to [steipete](https://github.com/steipete) who very quickly and kindly replied, but did not provide any information or hints that the architectures `armv7` and `i386` were in fact supported in XCFramework.
I decided for now that _not_ being able to support these 32 bit architectures was acceptable, although I do hope to revisit this later on.
**For now I had nominally gotten a solution that would support iOS and iOS simulator as well as Catalyst for 64 bit architectures**, which provides support for the last seven or so years worth of iOS devices.
This extends the power of cross-platform development to include Catalyst, while supporting Bluetooth and Go powered frameworks to be used.
The basic steps are outlined below.

## Steps Overview

1. Use the updated `gomobile` command from [dpwiese/mobile/pull/1](https://github.com/dpwiese/mobile/pull/1) to generate separate `.Frameworks` for each architecture/variant:
    ```sh {linenos=false}&nbsp;
    % gomobile bind -x -v -target=ios github.com/dpwiese/sample
    ```
2. Thin the `arm64` framework with `lipo -thin`:
    ```sh {linenos=false}&nbsp;
    % lipo arm64/Sample.framework/Versions/A/Sample -thin arm64 -output arm64/Sample.framework/Versions/A/Sample
    ```
3. Combine the frameworks with `xcodebuild -create-xcframework`:
    ```sh {linenos=false}&nbsp;
    % xcodebuild -create-xcframework \
    -framework arm64/Sample.framework \
    -framework amd64/Sample.framework \
    -framework catalyst/Sample.framework \
    -output Sample.xcframework
    ```
4. Import `.xcframework` into Xcode and build!
