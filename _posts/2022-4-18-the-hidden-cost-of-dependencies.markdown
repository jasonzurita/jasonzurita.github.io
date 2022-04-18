---
title: "The Hidden Costs of Your Dependencies"
layout: post
date: 2022-4-18
image: /assets/images/posts/hidden-cost-of-dependencies/logo.png
headerImage: true
tag:
- Dependencies
- Swift Package Manager
- Xcode
star: false 
category: blog
author: jasonzurita 
description: The Hidden Costs of Your Dependencies
---

# Summary

Dependencies in our projects have hidden, possibly unthought-of or ignored, costs when added. Dependency management tools like Swift Package Manager (SPM) and NPM make adding a 3rd party library almost too easy. With a few clicks you can import thousands of lines of code into your project and ship that to your users. Great, you got the task done, but at what cost? Did you think of the maintainability of that code, impact to your career growth, security implications, the complexity of what you just brought in, or how about the transitive dependencies?

Using a dependency isn't always _bad_, but we should be intentional about how and when we use them and what the trade-offs are.

<sub>Note: I am not talking about 1st class dependencies like importing UIKit from Apple. Yes, there can be some similar risks as mentioned below, but those risks are much lower because they are coming from the platform vendor.</sub>

---

# The hidden costs

## Complexity

First up are the added complexities brought in by each dependency, and some of them are not obvious. Multiply this by however many 3rd party dependencies are being used, and you have the potential for a lot of headache and distraction from more important work. The below items are in no particular order.
- **Managing versions**: If a dependency is maintained (hopefully it is — e.g., bug fixes, security improvements, API changes, new features), then you will have to update that dependency over time. There is more said below about the code migration impact of updating a dependency, but simply managing bumping versions and updating them takes time. This gets worse when you bring on dependencies that end up having the same transitive dependency. If those transitive dependency versions don't match, you have to resolve the conflict or may end up even having to ship two versions of the same library.

  In addition, you should be on top of each version update that comes out. Maybe there is an important security fix to get in or some other weakness (e.g., performance) to benefit from. Making sure you keep on top of your dependency updates is important, tedious, and sometimes a once in a while thing. Moreover, if you wait too long to update your dependency versions, you might have a hard time with all the changes waiting to come in.

- **Ripple effect of updating a dependency**: This item assumes that you are looking at the release notes for a dependency update to see what is in there (I am guilty of not doing this every time). When a dependency version is updated, you are then left with updating your code accordingly. Hopefully everything compiles and behaves as it did before. There is a real chance that something changed that now requires further action. Did an API change that is now preventing your code from compiling? Did you put a work-a-round for something weird in that dependency that is now fixed, and now your fix is a bug 😅? This of course is eased if the dependency follows [semver](https://semver.org/), so you know what to expect with each update. Let's face it though, 3rd party libraries are hit or miss when it comes to following semver. Managing the ripple effect of updating dependencies takes time. The more dependencies you have, the worse this gets. Oh and by the way, what do you do if a dependency is no longer maintained?

- **Build times**: Don't forget that each line of code added needs to be compiled (assuming your language is a compiled language). A single line alone has a mostly negligible impact on your code. Bring in a large dependency with 100s to 1000s of lines of code, and you just signed yourself up, and your team, to losing potentially lots of time throughout the year.

- **That new code is yours to support**: You bring in a library, and it solves a problem you have. Great. What happens if it doesn't work or crashes at some point? You have to resolve the issue. The bugs or crashes introduced are in your code base now, and that means digging in and working around them, fixing them, or finding a replacement for the dependency. Many of us think of dependencies as _black boxes_ with _free_ functionality, but the reality is that those black boxes are ours to support just like our own code — little is free in life. If you aren't ready to support that code, don't bring in that dependency.

  Artsy's principle of [owning your dependencies](https://github.com/artsy/README/blob/main/culture/engineering-principles.md#own-your-dependencies) is a good example. When they made use of React Native (RN), they didn't treat it like a black box and forget about it. Instead, they treated that code like a first class citizen and even got involved with the community that works on RN. 

- **Adding to the spaghetti**: When a dependency is brought in, it has a tendency to intertwine itself with your code. I am working on a project that uses [Realm](https://realm.io/). Coming to the realization that we don't actually _need_ Realm, we began making moves to remove it. The issue is that our use of Realm has smeared itself across the code base (e.g., accessing objects on particular threads, use of returned Realm objects, etc.). This is what tends to happen. Dependencies end up _bleeding_ their implementation details across your project making it difficult to work with over time let alone remove. Furthermore, if you have several opinionated dependencies architecturally, then you end up having a mix of many architectural decisions that you didn't make yourself, but now have to deal with.

## The lines of code problem
This may sound silly, but considering each line of code (LOC) added to your project is important. One reason, as mentioned, is that you have to support each line of code that is added, and a given developer can only support so many LOC. Alone, each line added isn't that big of a deal. If you are considering adding a function with 10 lines versus 9 lines then it doesn't make _that_ big of a difference in the short term. Multiply this over the course of the year and based on how many people are contributing to a code base and that number becomes a lot bigger. I like to think of it like dropping grains of sand at some location. At some point, you will have a beach, and that beach will have all sorts of garbage mixed in (tech debt, bugs, etc.). It is our job as developers to clean and maintain that beach.   

<p style="text-align:center;"><img src="/assets/images/posts/hidden-cost-of-dependencies/messy-beach.jpg" alt="Screenshot showing the options to pick for the local swift package" width="750"/></p>

Add a 3rd party library, and you have just _dumped_ some unknown truckload of sand (and garbage?) on your beach to maintain. If that 3rd party dependency has transitive dependencies, where that dependency has dependencies, then there is even more being brought into your project.

As mentioned, there is only so much sand that we can maintain (aka LOC). It shouldn't be that surprising to know that there is a limit. Enter [R0ml](https://twitter.com/r0ml)'s law: Each developer can write about 1000 LOC, debugged, each month. That same developer can support about 25k LOC a year. So at a point (~2 years), you would have stalled most new feature development of a project for that developer in about 2 years<sup>[1](https://github.com/jasonzurita/talks/blob/master/Code%20Golfing%20in%20Swift/Code%20Golfing%20in%20Swift.pdf)</sup>. Bringing in a dependency also counts towards the total LOC (even if you discount those lines of code because they are "battle tested"). If that dependency has transitive dependencies...🤯. Like mentioned above, you still have to maintain that code.

Time to hire. 

## Security
This is one of those subjects, like accessibility, that everyone knows they should be spending more time prioritizing. For dependencies, this means understanding what that code will be _doing_ when integrated into your project. If you don't audit your dependencies when bringing them in, I know I don't as much as I should, then you can be bringing in _anything_. In theory, a dependency, say *cough* *cough* Facebook's sdk<sup>**</sup>, can be sending up location or any other private user data to their backend for processing. Dependencies can even _listen_ to user entered input or [record all interaction](https://www.bugsee.com/) in your app. For Xcode projects, don't forget that any code including a dependency you bring in will have the same permissions that users grant your app. Meaning that if you request, and receive permission, to always access location, then the ability to get the location at all times is readily available to your source code in addition to any 3rd party dependency 🕵️.

Furthermore, if there is a dependency with a security weakness, then you are also vulnerable to that weakness. If a dependencies servers are hacked and they have your user data on them, that is your problem too. From a security perspective, dependencies increase the surface areas for something to go wrong. Remember, your users won't be blaming your app's dependencies for doing something shady or leaking data.

<sup>**</sup>_I am not actually sure if their libraries track your location, but I am skeptical about integrating anything made by Facebook/Meta._

## Broken windows
The idea of broken windows in software is that others are more likely to follow the established conventions and habits in a code base. If there are a lot of dependencies in a project or the community is dependency oriented (e.g., npm), then that tends to lead to adding more dependencies with less thought as to whether the dependency was actually needed. This can bring in a lot of complexity fast to your code base (e.g., managing versions of those dependencies, build time increases, bugs, etc.).

## Impacts to your team and career
In addition to all the above considerations, there are some that aren't as directly tangible.
- **Onboarding**: Depending on what dependencies you have, this can become more challenging. New hires are getting up to speed with the domain that your app is in and learning all that context while also balancing learning the tech stack. If at every turn, they have to learn a totally new dependency that they aren't already familiar with, then they are buried with homework. Hopefully the documentation is good for those dependencies.

- **Personal career growth**: Said bluntly, if all you are doing is bringing in dependencies to solve your problems, you are mainly learning how to be a dependent developer. This is a great way to start learning as a developer, but at some time you should drop a layer deeper and _understand_ what you are doing at a more fundamental level. Bringing in a dependency is a short cut that subtly sides steps you having to learn more about that space. Do you have to do multipart uploading? Why not check out the relevant RFCs ([1](https://www.rfc-editor.org/rfc/rfc1867.html), [2](https://www.ietf.org/rfc/rfc2388.txt)) and learn more about how the internet works. You will be surprised how much deeper learning like this can help your career.

---

# Additional thoughts
If you don't limit the number of your dependencies, you may end up getting consumed by them and all of the costs associated with bringing them in. I'll admit, keeping all these _costs_ in mind for each dependency is a lot. Instead, try and be critical about what you bring into your project, and avoid dependencies if you can.

After all of this, you may be saying to yourself, "Okay, but I still _need_ this dependency. Now what?". In an upcoming post, I'll have some thoughts to share for the times where you do bring on a dependency. Spoiler, think it through and be intentional about the process of bringing on a dependency.

Also, here are a couple other resources on dependencies that may be of interest:
 - [I will pay you cash to delete your npm module](https://drewdevault.com/2021/11/16/Cash-for-leftpad.html)
 - [Dependencies: Why we got rid of most dependencies](https://chris.eidhof.nl/post/fewer-dependencies/)

---

Thanks for reading. Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
