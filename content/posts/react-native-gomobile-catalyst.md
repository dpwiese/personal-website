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
An early discussion in [React Native Community](https://github.com/react-native-community) is
[Support Project Catalyst (running iPad apps on macOS)](https://github.com/react-native-community/discussions-and-proposals/issues/131) which provided some good resources and examples before starting.

# Targeting Catalyst from Bare React Native App

Do note at minimum to to target Catalyst you must:

1. Xcode: `File` > `Workspace Settings` > `Enable new build system`

<img src="/img/posts/react-native-gomobile-catalyst/new-build-system.png" width="500" style="border-style:solid;border-width:1px"/>

2. Enable Mac as target under mobile-client target

<img src="/img/posts/react-native-gomobile-catalyst/target-mac.png" width="700" style="border-style:solid;border-width:1px"/>

With `npx react-native init` used to create a new app, at the time in React Native 0.62.0, and performing the two steps above, an attempt to deploy to Catalyst gave the following error:

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
<!--
TODO: FIX SYNTAX HIGHLIGHTING FOR SWIFT
-->
```swift {linenos=false}&nbsp;
 #if !targetEnvironment(macCatalyst)
 // Code to exclude from Mac.
 #endif
```

The pull request with these couple of changes to enable `react-native-ble-plx` to work when targeting Catalyst is [here](https://github.com/Polidea/MultiPlatformBleAdapter/pull/54).

Finally, with these changes, and after setting the entitlments by checking Bluetooth in App Sandbox under Signing & Capabilities, the React Native application was running on my Mac via Catalyst and communicating via BLE.

<img src="/img/posts/react-native-gomobile-catalyst/bluetooth-sandbox.png" width="600" style="border-style:solid;border-width:1px"/>

The next and final step was to include the frameworks I'd previously built using [Go Mobile](https://github.com/golang/mobile) and ensure they worked with Catalyst.

# Gomobile for Catalyst

## Workflow

Symlink helpful.
This allows the source code to be placed as desired, and the relative path specified when running `gomboile bind` to be left alone.

```sh
% ln -s ~/Code/dpwiese/gomobile-catalyst/src/sample ~/go/src/github.com/dpwiese/sample
```

Recall the following command which builds, in this case with the `-taret=ios` a framework for use in Xcode, supporting the architectures defined in [cmd/gomobile/env.go](https://github.com/golang/mobile/blob/master/cmd/gomobile/env.go), and working on both iOS and the simulator.

```sh
% gomobile bind -x -v -target=ios github.com/dpwiese/sample
```

The `gomboile` command is in `~/Code/dpwiese/mobile/cmd/gomobile`.
To edit the source and build a new version of `gomobile`, navigate to the above directory and run `go build`.
This generates a new `gomobile` executable which should be moved to `~/go/bin`.

## Initial Attempt

Having already generated a framework using the above command, a first attempt at building my previous sample application with React Native and Go Mobile [react-native-gomobile-demo](https://github.com/dpwiese/react-native-gomobile-demo) without modification gave the following error.

<!--
TODO: ALLOW SCROLLING WHEN LINE NUMBERS ARE DISABLED
-->
```txt {linenos=false}&nbsp;
error: Building for Mac Catalyst, but the linked framework 'Sample.framework' was built for iOS + iOS Simulator.
You may need to restrict the platforms for which this framework should be linked in the target editor,
or replace it with an XCFramework that supports both platforms.
```

This was not unexpected.
I quickly found the issue [golang/go/issues/36856](https://github.com/golang/go/issues/36856) which proposed setting the compiler flag with `-target x86_64-apple-ios13.0-macabi` to generate a Catalyst compatible framework.
I was not very familiar with the format of this flag, so checking [the LLVM source](https://github.com/llvm/llvm-project) seemed like a good place to start to undestand it a bit better.
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

ARCHITECTURE = ArchType = x86_64
VENDOR = VendorType = apple
OPERATING_SYSTEM = OSType = ios
ENVIRONMENT = EnvironmentType = macabi

Now understanding where this flag had come from, I naively implemented the fix proposed in [golang/go/issues/36856](https://github.com/golang/go/issues/36856) in [golang/mobile/pull/45](https://github.com/golang/mobile/pull/45) to add the flag an build a new framework.
Nominally it worked, but this hasty fix simply overwrote the output for the `x86_64` architecture which is needed for the iOS simulator.
So with nothing more than these few lines I was able to get the framework working for Catalyst, but at the expense of giving up iOS simulator support, which was not acceptable.
I then had to figure out how to provide support for the `x86_64` architecture for both Catalyst _and_ the iOS simulator.

## Combining Static Archives

Looking at the way Go Mobile works, each of the different architectures specified in [cmd/gomobile/env.go](https://github.com/golang/mobile/blob/master/cmd/gomobile/env.go) is used to build a corresponding static library which is ultimately combined with `lipo` and placed in the `.framework` output, as below:

<!--
TODO: FIX SYNTAX HIGHLIGHTING FOR SH
-->

```sh {linenos=false}&nbsp;
% lipo -create \
-arch armv7 sample-arm.a \
-arch arm64 sample-arm64.a \
-arch i386 sample-386.a \
-arch x86_64 sample-amd64.a \
-arch x86_64 sample-catalyst.a \
-o Sample.framework/Versions/A/Sample
```

It is simple enough to create a separate `amd64` static archive with the flags needed for catalyst, but multiple archives of the same architecture cannot be combined with `lipo`.
Given that `catalyst` is not a seperate architecture, this presents some obvious difficulties:

```txt {linenos=false}&nbsp;
fatal error: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/lipo:
/var/folders/5f/26gs6d3n11jfxldns9vqhxn40000gp/T/gomobile-work-105957311/sample-amd64.a
and
/var/folders/5f/26gs6d3n11jfxldns9vqhxn40000gp/T/gomobile-work-105957311/sample-catalyst.a
have the same architectures (x86_64) and can't be in the same fat output file
```

So I started looking at creating two separate `amd64` static archives, each with the flags needed for the iOS simulator and catalyst, and then combining them with [libtool](https://www.gnu.org/software/libtool/) or [ar](https://linux.die.net/man/1/ar).






My first attempt resulted in the following error when attempting to use the generated `.framework`:

```txt {linenos=false}&nbsp;
In ios/Sample.framework/Sample(000006.o), building for Mac Catalyst, but linking in object file built for iOS Simulator, file 'ios/Sample.framework/Sample' for architecture x86_64
```

I think this approach should work, and I probably just need to spend some time learning `libtool`/`ar` better, as I may not be invoking them correctly to combine the static archives. I hope to look back at this again soon.


## Enter XCFramework

As the error above `replace it with an XCFramework that supports both platforms.` suggested...

[This comment](https://github.com/golang/mobile/pull/45#issuecomment-651183803) suggested using XCFramework.

XCFrameworks introduced WWDC2019 and in Xcode 11...

https://github.com/dpwiese/mobile/pull/1

```sh {linenos=false}&nbsp;
% xcodebuild -create-xcframework \
-framework arm/Sample.framework \
-framework arm64/Sample.framework \
-framework 386/Sample.framework \
-framework amd64/Sample.framework \
-framework catalyst/Sample.framework \
-output Sample.xcframework
```

A first attempt at using `xcodebuild -create-xcframework` gave:

```txt {linenos=false}&nbsp;
error: unable to find any architecture information in the binary at '~/Desktop/arm64/Sample.framework/Sample'
```

This seems to be solved by thinning?

```sh {linenos=false}&nbsp;
% lipo arm64/Sample.framework/Versions/A/Sample -thin arm64 -output arm64/Sample.framework/Versions/A/Sample
```

After thinning tried to combine again.


this post

[firebase-ios-sdk/pull/4737](https://github.com/firebase/firebase-ios-sdk/pull/4737/files#diff-ae87db694a268f93c6029c8b91a3b9edR333)

says

> xcframework doesn't support legacy architectures: armv7, i386.
> It will throw a "Both ios-arm64 and ios-armv7 represent two equivalent library definitions" error.

<img src="/img/posts/react-native-gomobile-catalyst/firebase-ios-sdk-discussion.png" width="900" style="border-style:solid;border-width:1px"/>

but still not able to figure out if that is true or not, or how to do it...

I reached out to [steipete](https://github.com/steipete) who very quickly and kindly replied, but did not provide any information or hints that armv7, i386 were in fact supported in XCFramework.

What are implications of not being able to use these older architectures in XCFramework?
Which devices does that mean cannot be supported?
And which devices _are_ supported by using only architectures `arm64` and `amd64`?

## Steps Overview

A nominal approach working, at least for `arm64` and `amd64` (including catalyst).
Steps are:

1. Use updated `gomobile` to generate separate `.Frameworks` for each architecture/variant
2. Thin `arm64` with lipo
3. Combine the (three currently working) frameworks with `xcodebuild -create-xcframework`
4. Import `.xcframework` into Xcode

### Step 1: Modify Gomobile

The goal here is to have the output of `gomobile bind` give a separate `.framework` for each architecture.
The `mobile` repository is forked, and relevant files are in `/Users/dpwiese/Code/dpwiese/mobile/cmd/gomobile` and are: `bind_iosapp.go` and `env.go`.
The modifications are quite straighforward and the command is the same:

```sh {linenos=false}&nbsp;
% gomobile bind -x -v -target=ios github.com/dpwiese/sample
```

The various frameworks are all output to the current working directory, with each framework being of the same name but separated into folders by architecture, e.g.:

```txt {linenos=false}&nbsp;
arm64/Sample.framework
amd64/Sample.framework
catalyst/Sample.framework
```

### Step 2: Thin the arm64 Framework with Lipo

This converts the archive to non-Fat with the following command:

```sh {linenos=false}&nbsp;
% lipo arm64/Sample.framework/Versions/A/Sample -thin arm64 -output arm64/Sample.framework/Versions/A/Sample
```

### Step 3: Combine the Frameworks

```sh {linenos=false}&nbsp;
% xcodebuild -create-xcframework \
-framework arm64/Sample.framework \
-framework amd64/Sample.framework \
-framework catalyst/Sample.framework \
-output Sample.xcframework
```

### Step 4: import into Xcode

This is very easy.




# Prerequisites

* Static versus dynamic linking
* FAT and thinning
* Framework versus library

