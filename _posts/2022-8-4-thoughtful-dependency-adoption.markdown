---
title: "Thoughtful Dependency Adoption"
layout: post
date: 2022-8-4
image: /assets/images/posts/thoughtful-dependency-adoption.png
headerImage: true
tag:
- Dependencies
star: false 
category: blog
author: jasonzurita 
description: Thoughtful Dependency Adoption
---

# Summary

In a previous [post](https://jasonzurita.com/the-hidden-cost-of-dependencies/), I wrote about the costs of bringing in a 3rd party dependency. One of the main points was to suggest that we avoid bringing in dependencies. However, this isn't always possible. There are valid reasons to bring one in like to take a shortcut now to help deliver value to your company/product sooner (e.g., at a startup). This post picks up where that previous post left off and offers a method for how to approach bringing in a dependency, if you _have_ to.

Note: although examples below are iOS biased, the ideas are platform agnostic.

---

# Still want that dependency?
With the [previous post](https://jasonzurita.com/the-hidden-cost-of-dependencies/) in mind, let's say we still want to bring a 3rd party dependency into our project. In some cases, it makes sense to add a dependency. Perhaps you are a startup trying to get funding and you need a backend quickly, maybe to support integrating the really complex subject of in-app purchases, or maybe you are just trying to prototype something quickly. Whatever the reason, here are some tips that I have found useful when bringing in a 3rd party dependency.

## 1. Justify the need
First thing to question, even for existing dependencies, is if you actually _need_ the dependency at all. Maybe there is already a 1st party library available that you missed? If not, try and write the dependency yourself. This seems obvious, but we (including myself) are eager to skip this step and jump right for a new dependency. Even if it takes a day or two to write the functionality you need, you will better understand that part of the code and be better positioned to support that code in the future. Also, this is a great opportunity to learn something new!

For example, laying out UIKit user interfaces (UIs) can be tricky. To help with this libraries like [SnapKit](https://github.com/SnapKit/SnapKit) have popped up to make this easier. However, these days the native [Auto Layout](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/index.html) APIs have come a long way, which diminishes the _need_ of something like SnapKit, but we sometimes blindly throw a dependency like this in since we've seen it used in the past or because someone, \*cough\* StackOverflow, tells us this is the route to go. Let's say the current Auto Layout APIs aren't as ergonomic to use as you would have liked. Check out this awesome post by Chris Eidhof - [A Micro Auto Layout DSL](https://chris.eidhof.nl/post/micro-autolayout-dsl/). In something like 20 lines of code (LOC), you have a micro DSL that makes Auto Layout a joy. I use this in nearly all of my UIKit projects — no dependency!

If you attempt to write the functionality yourself, and you fall short. Well, you have just better justified the need for the dependency 💪.

## 2. Copy and paste
So, you tried looking for a 1st party library, tried writing it yourself, and that dependency is looking like a good option. You aren't totally out of alternatives just yet. Try going to the 3rd party library and take a look at the functionality that you want to use (assuming it is open source). Chances are that you only want to use a small portion of the library. In that case, or if the dependency is small enough, consider copying and pasting the code into your project!

You may be saving your code base _many_ LOC, and this is important because you are limiting what you have to support and the surface area of bugs. Also, you don't have to bring in the complexity of managing that dependency, you can inspect the code further to better understand what it is doing, and you are more formally stating that you will support this code. You had to support it regardless even if you brought in as a dependency. We often think of dependencies brought in with a dependency management system (e.g., Swift Package Manager) as something external, and we therefore subconsciously discredit the need to support a dependency as if it were code that we wrote. Remember, if something goes wrong in that 3rd party code, your app at the end of the day is the one with an issue.

It is worth nothing that by copying code in from an open source project doesn't mean you are abandoning open source. There is no reason why you can't still contribute back to that code base if you find a bug or make a useful change to that code. This simply means that you are reducing the complexity in managing that dependency and potentially reducing the surface area of things that can go wrong when bringing in foreign code.

## 3. Isolate the dependency
The term _spaghetti code_ comes up often to describe code bases with many systems and ideas intertwined and dependent on each other. Nobody starts out with a new project thinking they are going to write spaghetti code. One way this ends up happening without us knowing it, is by taking a 3rd party dependency (or dependencies) and integrating them into your project in an uncontrolled manner. Dependencies have a way of smearing themselves across a code base. You start by adding a dependency to one particular part of the code base, but then its types, function calls, and architecture slowly creep all over. Before you know it, you are _dependent_ on that code 😅 and have a pile of spaghetti on your hands. A my work, we've had a hell of a time removing Firebase because it was _everywhere_. With nearly a year of chipping away at it, we are almost done.

If you are going to integrate a dependency, do yourself a favor and cage that animal — I like to think about it as _designing code for deletion or replacement_. By properly isolating a dependency when you bring it in, you set yourself up to more easily remove or replace it in the future and get the side benefit of more easily testing with that dependency (we even control calls to 1st party libraries to help with testing and mocking). There are many ways to accomplish this. This isn't a post on how to do that, but to name a few.
- Put the 3rd party dependency behind a protocol (also known as an interface in other languages): This is a pretty common way to control a dependency. Define the API interface that you want to interact with, and have the dependency conform to this. Then only deal with that abstract interface in your code.
- Use the [client pattern](https://www.pointfree.co/collections/dependencies/designing-dependencies). This throws away protocols in favor of simple types (e.g., structs). There is still an API interface, but it is through a concrete type where most of the properties are closures (or lambda functions in some other languages). This pattern is what we use, and is powerful for dev workflow mocking and testing. Also, it is really lightweight. Moreover, you can use this same pattern across multiple platforms. For example, we use this in both our iOS and Android apps (so many benefits from standardized architectural patterns like this).
- Use dependency injection: Another way to isolate your code from a 3rd party dependency is to use dependency injection. Where that dependency is needed, accept a closure/lambda function that provides the output that you need. The caller will need to be aware of the 3rd party dependency in order to fill out that functionality, but you can at least limit how much of your code base _knows_ about the dependency.

There are other ways, but you get the idea.

---

# Summary
There is a time to bring in a dependency, but blindly doing so can have a large impact down the line. For example, Firebase helped get [Driver](https://apps.apple.com/us/app/driver-dash-cam-navigation/id1415557883) in the early days, but we have spent nearly a year working to get rid of it 😅. Integrating a dependency is so [easy](https://www.youtube.com/watch?v=SxdOUGdseq4). Sometimes just a couple of lines of code and you are good to go. However, like most things, there are [tradeoffs](https://jasonzurita.com/the-hidden-cost-of-dependencies/). Because of this, we should be thoughtful about how we bring in our dependencies and take steps to protect ourselves from them.

Also, huge shoutout to [R0ml](https://twitter.com/r0ml). I have learned a lot of these ideas from him.

---

Thanks for reading. Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
