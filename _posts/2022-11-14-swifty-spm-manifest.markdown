---
title: "Make Your Swift Package Manager (SPM) Manifest Swifty"
layout: post
date: 2022-11-14
image: /assets/images/posts/swifty_spm_manifest/logo.png
headerImage: true
tag:
- Swift Package Manager
- Swift
- Modularization
star: false 
category: blog
author: jasonzurita 
description: Make Your Swift Package Manager (SPM) Manifest Swifty
---

# Summary

The [Swift Package Manager (SPM)](https://www.swift.org/package-manager/) has been a great 1st party addition to Swift development and by extension Apple platform development (e.g., iOS and iPadOS). Not only are we able to more easily import and use a dependency, but we can more easily _modularize_ our code<sup>1</sup>. In the current project that [I work on](https://drivertechnologies.com), we had a really large SPM manifest file (aka Package.swift), which defined over 100 internal modules! Adding a new module into the mix was a bit cumbersome (especially since most of a module's definition is done using strings). This can be better. Why not make some tweaks to the manifest file to make it more _Swifty_<sup>2</sup> — it is written in Swift after all (as a DSL), so shouldn't we be able to use some Swift features to add type safety and let the compiler help in adding new modules 🤔.

<sub>[1]: The subject of modularizing your code is for a future post, but the main idea is breaking down your code into smaller isolated parts that can be tested and used in isolation. Composition of reusable components is quite useful, powerful, and helps simplify development. If you want to learn more, this is a [great free video](https://www.pointfree.co/episodes/ep142-a-tour-of-isowords-part-1) showing off a project that is well modularized.</sub><br><br>
<sub>[2]: Believe it or not, "Swifty" is a legitimate word. It means _idiomatic Swift_. Kind of like _Pythonic_ for the Python language. Interestingly enough, some languages don't seem to have a word like this. For example, JavaScript 🤷‍♂️.</sub>

---

👀 **Looking for some reference code?** [Here is a SPM template]().

---

# A Swifty SPM Manifest
<sub>Note: If you want an overview of the anatomy of the SPM manifest and how to create a SPM Package, see the [official overview of SPM](https://www.swift.org/package-manager/). You can also checkout the documentation of the [SPM project on GitHub](https://github.com/apple/swift-package-manager) (it's open source).</sub>


With the changes suggested in here, adding a new module can be made much simpler. Since we are using an enum to define the SPM modules, we can lean on _exhaustive switch statements_ to let the compiler effectively guide us in adding a new module. This is probably the biggest benefit (and coolest part 😎). Just check out the gif below!<br>

<img src="/assets/images/posts/swifty_spm_manifest/adding_a_new_module.gif" alt="Adding a new module to SPM" width="700"/><br>

Let's break all this down.


## The new "Swifty" Manifest
Going a little backwards here, this is what the SPM package definition ends up looking like! Not much to it because the _products_ and _targets_ are defined elsewhere (more on this below). With the changes suggested here, there isn't much to do here anymore 😌. 

```swift
// MARK: - SPM Definition
let package = Package(
    name: "Modules",
    defaultLocalization: "en",
    platforms: [
        .iOS(.v15),
    ],
    products: Modules.allProducts,
    dependencies: [
        .package(url: "https://github.com/pointfreeco/swift-snapshot-testing.git", from: "1.9.0"),
    ],
    targets: Modules.allTargets
)
```

## How to Define the Products and Targets?
This is a real SPM manifest file from a test app I work on, so what you see below is the real deal and not a toy example. This might look like a lot up front, but it is just an enum. To help add some commentary, I added numbered comments (e.g., _// 1_). Below you will find further explanation for each number.

### The breakdown
1. This section is pretty straight forward. These are all the modules. To add a new one, simply add a new case — that's it!
  + What is really powerful here, is that the compiler will walk you through adding all the necessary information like dependencies. There is a gif later showing what this looks like.
1. Second part

```swift
// MARK: - Custom SPM Configuration
// To add a new module, add a case below and follow the compiler errors 💪.
enum Modules: String, CaseIterable {
// 1
    case world
    case locationClient
    case summaryFeature
    case permissionsFeature
    case motionActivityClient
    case automaticDriveDetectionFeature
    case app
    case language
    case motionClient

    // MARK: - Public API

    /// All the targets, including test targets. To be used in `targets:` section.
    static var allTargets: [PackageDescription.Target] { targets + testTargets }

    /// All the products that are to be externally accessed. To be used in the `products:` section.
    static var allProducts: [PackageDescription.Product] {
        allCases.map {
            .library(name: $0.moduleName, targets: [$0.moduleName])
        }
    }

    // MARK: - Private API

    // Prefix for the module names. This is useful for things like IDE auto-complete.
    private var prefix: String {
        switch self {
            case .world, .locationClient, .summaryFeature,
                .permissionsFeature, .motionActivityClient,
                .automaticDriveDetectionFeature, .app, .language,
                .motionClient:
        return "MT"
        }
    }

    // Used anywhere where we want to know the string name for the module.
    private var moduleName: String {
        return prefix + String(rawValue.prefix(1).uppercased() + rawValue.dropFirst())
    }

    // Directory path as an array to where the source files live
    private var path: [String] {
        switch self {
        case .summaryFeature, .automaticDriveDetectionFeature, .permissionsFeature: return ["Features"]
        case .app, .world, .motionClient, .locationClient, .language, .motionActivityClient: return []
        }
    }

    // MARK: - Targets

    private static var targets: [PackageDescription.Target] {
        [Modules.world, Modules.language, Modules.locationClient, Modules.motionClient,
         Modules.app, Modules.motionActivityClient, Modules.summaryFeature, Modules.automaticDriveDetectionFeature,
         Modules.permissionsFeature].map {
            return .target(
                name: $0.moduleName,
                dependencies: $0.targetDeps.map { Target.Dependency.byName(name: $0.moduleName) },
                path: "Modules/\($0.path.joined(separator: "/"))/\($0.rawValue)/src",
                resources: $0.resources
            )
        }
    }

    private var targetDeps: [Modules] {
        switch self {
        case .world: return [.language, .locationClient, .motionClient, .motionActivityClient]
        case .summaryFeature: return [.language, .world, .motionActivityClient, .automaticDriveDetectionFeature]
        case .permissionsFeature: return [.language, .world]
        case .automaticDriveDetectionFeature: return [.motionActivityClient, .locationClient]
        case .app: return [.summaryFeature, .permissionsFeature, .locationClient, .motionClient]
        case .language, .locationClient, .motionClient, .motionActivityClient: return []
        }
    }

    private var resources: [PackageDescription.Resource]? {
        switch self {
        case .app, .language, .motionClient, .locationClient, .world, .automaticDriveDetectionFeature:
            return nil
        case .summaryFeature, .motionActivityClient, .permissionsFeature:
            return [.process("Resources")]
        }
    }

    // MARK: - Test Targets

    private static var testTargets: [PackageDescription.Target] {
        allCases
            .filter { $0.shouldIncludeTestTarget }
            .map {
                PackageDescription.Target.testTarget(
                    name: $0.moduleName + "Tests",
                    dependencies: $0.testTargetDeps,
                    path: "Modules/\($0.path.joined(separator: "/"))/\($0.rawValue)/Tests",
                    exclude: $0.exclude
                )
            }
    }

    private var shouldIncludeTestTarget: Bool {
        switch self {
        case .world: return false
        case .locationClient: return false
        case .summaryFeature: return true
        case .permissionsFeature: return true
        case .motionActivityClient: return false
        case .automaticDriveDetectionFeature: return true
        case .app: return true
        case .language: return false
        case .motionClient: return false
        }
    }

    private var testTargetDeps: [Target.Dependency] {
        switch self {
        case .world: return []
        case .summaryFeature:
            return [
                Modules.summaryFeature
            ].map { Target.Dependency.byName(name: $0.moduleName) } +
            [.product(name: "SnapshotTesting", package: "swift-snapshot-testing")]
        case .permissionsFeature:
            return [
                Modules.permissionsFeature
            ].map { Target.Dependency.byName(name: $0.moduleName) } +
            [.product(name: "SnapshotTesting", package: "swift-snapshot-testing")]
        case .automaticDriveDetectionFeature:
            return [
                Modules.automaticDriveDetectionFeature
            ].map { Target.Dependency.byName(name: $0.moduleName) }
        case .app:
            return [
                Modules.app, Modules.locationClient
            ].map { Target.Dependency.byName(name: $0.moduleName) } +
            [.product(name: "SnapshotTesting", package: "swift-snapshot-testing")]
        case .language, .locationClient, .motionClient, .motionActivityClient: return []
        }
    }

    private var exclude: [String] {
        switch self {
        case .world: return []
        case .locationClient: return []
        case .motionActivityClient: return []
        case .automaticDriveDetectionFeature: return []
        case .app, .summaryFeature, .permissionsFeature: return ["__Snapshots__"]
        case .language: return []
        case .motionClient: return []
        }

    }
}
```

---

Now you get compiler help

With this, gone are the days where you forget to add a letter in the path or uppercase something that you shouldn't have. Hunting those mistakes down are painful sometimes. Especially with the tendency to receive cryptic error messages from the compiler.
- Auto complete

# Summary
String APIs aren't great in that they can easily break. This combined with the possibility of the SPM manifest growing very large makes for a potentially cumbersome file to work with. If you are using Swift, then you have a superpower — the type system. If used with intention, we can have the compiler _help_ us write code. This is the case with the above refactor. When you go to add a new internal dependency, the Swift compiler will show you all the areas that need to be updated!

---

Thanks for reading. Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
