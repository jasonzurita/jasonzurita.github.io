---
title: "Testing for watchOS Apps (Apple Watch)"
layout: post
date: 2021-2-12
image: /assets/images/posts/watchos_testing/logo.png
headerImage: true
tag:
- Swift
- Swift Package Manager
- Testing
- watchOS
- Xcode
star: false 
category: blog
author: jasonzurita 
description: Automated testing for watchOS Apps
---

# Summary

When [Apple Watch only apps](https://developer.apple.com/documentation/watchkit/creating_independent_watchos_apps/) were announced, I was excited and immediately jumped to create a standalone Apple Watch app! Even though I enjoy working on my watch apps, the development and developer experience lacks — almost as if support for independent watch apps was a rushed thought. Here are just some of the things that stood out to me (some of which I [filed a radar](https://twitter.com/jasonalexzurita/status/1306617636185427968?s=20) for):
- watchOS only apps in the App Store have a rating just like any other app, but sadly there is no 1st class support for [requesting a review](https://developer.apple.com/documentation/storekit/skstorereviewcontroller) from users like other platforms.
- Independent watchOS app don't have their own platform section in iTunes Connect. If you have a watchOS app, it is still labeled iOS. Confusing, I know, but I guess you get used to it.
- The experience of searching and discovering Apple Watch only apps is a bit awkward. The iOS App Store kind of shows that an app is watch only, and the usability of the App Store app on the watch is weakened by the restricted watch screen size.
- Lack of support for some APIs/Frameworks (e.g., lack of MetricKit support).
- Sometimes painful process to build and run on an actual test device.
- **Lack of automated testing support.**

Addressing _Lack of automated testing support_ is the subject of this post. I find a lot of value in automated testing, so I was real bummed that Apple allowed independent Apple Watch apps without automated testing support (XCTest). In hindsight, I probably should have expected this since testing doesn't seem to be a major focus from the Apple dev tools side. After some soul searching and digging, I found a way to add tests to independent watchOS apps so I figured I would share!

---

# Automated Testing for watchOS Apps
<sub>Note: If Xcode stops working as expected (like not being able to build or errors aren't showing up correctly), restart Xcode. I have had to do this more times than I care to admit 😅.</sub>

### Hello Swift Package Manager
The first thing to point out is that this is a work around for the lack of testing support for watchOS apps. As mentioned above, there is no _out of the box_ support for testing watchOS code. So we need another place to put our code that both allows testing and can be imported into the watchOS app. This is where Swift Package Manager (SPM) comes in! The high level idea is to add a local swift package using SPM that includes targets for both _watchOS_ **and** another platform that supports testing like _iOS_.

### Initial setup
- Open your Xcode project that has a watchOS app (this will work for both independent and companion watch apps).
  + Note: if you don't already have a project with a watchOS target, you can create a new independent watchOS app by:<br>`Open Xcode > File > New Project > watchOS > Watch App > Name and settings are up to you`.
- Add a local swift package.
  + `File > New > Swift Package`
  + Name the new Swift Package.
  + Select a location. I usually opt to put the swift package in the root of the Xcode project directory.
  + At the bottom, change _Add to:_ to your project or workspace.
  + Also at the bottom, change _Group:_ to your project or workspace.
    <p style="text-align:center;"><img src="/assets/images/posts/watchos_testing/1.png" alt="Screenshot showing the options to pick for the local swift package" width="375"/></p>
  + Click _Create_.
  + You should now see your swift package under your project or workspace directory like the screenshot below.
    <p style="text-align:center;"><img src="/assets/images/posts/watchos_testing/2.png" alt="Screenshot showing the location of the newly added local swift package" width="375"/></p>
- Make use of the local swift package.
  + We have to link the local swift package to the main app target (the watchOS app in this case).
  + Click on the project or workspace.
  + Find the target you want to use the swift package in.
  + Click on the `+` under _Frameworks, Libraries, and Embedded Content_.
  + Find the swift package.
  + Click add.
    <p style="text-align:center;"><img src="/assets/images/posts/watchos_testing/3.png" alt="Screenshot showing the steps to link a local swift package to the main app" width="500"/></p>
  + Now the swift package is available for use! Simply `import ExampleSwiftPackage` when you want to access your local swift package code.
    + Replace _ExampleSwiftPackage_ with the name of your swift package.
    + Also, keep in mind that you will need to mark code in your swift package as public to make use of it.

### Testing setup
At this point, we have a local swift package integrated and available for use in the watchOS app 🎉. That is great, but we haven't actually tested anything. We will, but first I should note some things about the swift package for clarity (this isn't a post on SPM, so this list doesn't aim to be comprehensive).
- The out-of-the-box swift package we added contains one module, but you can have as many as you like.
  + From a high level, you would update your _Package.swift_ to add the new module(s) and then make the respective directory/file changes.
- As mentioned, there is one module available for use in the main app from the stock setup. If you look in the _Package.swift_, that is reflected as the _.library(_.
  + A library is composed of _targets_, which can also be seen under _targets:_.
  + For a given target, like in our case, you typically have a _.target_ (your source code) and a _.testTarget_ the tests for that source code. These targets implicitly map to the directory structure for the swift package. You can override the path if you want to structure things differently.

Now that we know a little more about the way things fit together for a swift package, let's do some testing!
- Switch over to the swift package scheme.
    <p style="text-align:center;"><img src="/assets/images/posts/watchos_testing/4.png" alt="Screenshot showing the selection of the swift package scheme" width="500"/></p>
- Select a simulator that can run tests like like an iPhone and `cmd + b` to make sure the package is building.
- Click on the _Package.swift_.
- We will add the supported platforms to our swift package (We don't strictly need to do this, but it is better to be explicit).
   ```swift
let package = Package(
    name: "ExampleSwiftPackage",
    // Add the _platforms_ field
    platforms: [
        .watchOS("7"), .iOS(.v13),
    ],
    . . .
```
- Open up the test file.
  + In this case, `ExampleSwiftPackage > Tests > ExampleSwiftPackageTests > ExampleSwiftPackageTests.swift`.
- Run the stock tests, `cmd + u`.
  + These tests will run against the iOS simulator, but the code will be usable in your watchOS app!
- We are now testing 🥳!

### Running tests from the watchOS apps scheme
Lets take this one step further and add the ability to run these tests from the watchOS app scheme for ease of running. This isn't hard to do, and also has the side benefit of working well in a CI/CD system.
- At the top of Xcode, select the watchOS app scheme then select _Edit Scheme..._.
- Click on _Test_ from the left options.
- Click on the `+` at the bottom to add tests.
- Find and add the swift package tests!
  + Do this for each SPM module that you want to include when running tests from the watchOS scheme.
  <p style="text-align:center;"><img src="/assets/images/posts/watchos_testing/5.png" alt="Screenshot showing how to add SPM tests to the watchOS app scheme" width="500"/></p>
- Now when you press `cmd + u` in the watchOS scheme, you will run all the tests!!
- That's it 💪!

---

# Additional thoughts
- This post focuses on testing for watchOS apps, but this workaround will also work for adding tests to other extensions (widget, notification, etc.)!
- Although adding a local swift package adds support for testing, you also get other benefits from structuring your code this way. For starters, this is good practice for [code separation](https://en.wikipedia.org/wiki/Separation_of_concerns) (decoupling your code). Organize your SPM modules as isolated units of work, and you will make your code easier to change and reuse such as using that module in other contexts like in an iOS app or tvOS app!

---

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
