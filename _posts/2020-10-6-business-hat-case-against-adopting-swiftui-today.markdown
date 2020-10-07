---
title: "A (Business Hat) Case Against Adopting SwiftUI Today"
layout: post
date: 2020-10-6
image: /assets/images/posts/business-hat-swift.png
headerImage: true
tag:
- SwiftUI
- Xcode
- iOS
star: false 
category: blog
author: jasonzurita 
description: A (business hat) case against adopting SwiftUI Today

---

# Summary

First off, I should note that everything in me wants to write SwiftUI. How could I not? It is a great step forward for UI development and is clearly the future of development for Apple Platforms. In fact, I have two apps in the App Store that are fully SwiftUI. One of which has been using SwiftUI for over a year now.

So why write this post? This post is focused on considerations targeted at established apps. If you are going to start a new project, still think it through since SwiftUI _is_ young, but by all means select SwiftUI in Xcode and party on!

With that said, we need to be careful about chasing new shiny things like this. If the goal of your existing app is to maintain a high velocity of delivering value to your users, then there are some considerations before declaring your whole code base as _tech debt_ 🤪.

---

# SwiftUI is still new
SwiftUI is now over a year old — not a lot of time for a framework to be battle tested and sharp edges worked out. Apple has had some time to improve SwiftUI, and they have done an amazing job at that! But like all new technologies there are still kinks to be worked out. Some things that come to mind:
- We are all still trying to figure out how to best use SwiftUI (architecture, etc.). There's been some good progress to address this (some references below), but Apple has been kind of light recommending architectural patterns. Furthermore, there aren't that many large scale SwiftUI projects to see how the framework and related architectures scale. If you are going to use SwiftUI, be ready to try and figure things out while they are evolving. This means time that could have been used to deliver feature work.
- SwiftUI doesn't support all Apple frameworks just yet. If you are using things like MapKit (< iOS 14) or AVFoundation, you will need to work around this and bridge between that and SwiftUI. Not a huge deal because the interop story is pretty good, but still something else to deal with.
- Documentation from Apple has been a bit weak, so learning how to get things done, even basic things, is a bit painful right now. Yes, there are more articles out there than a year ago, but opinions are moving pretty fast so there is a level of noise to filter through. In other words, when searching for a way to accomplish something, you will likely come across several different ways to get that thing done — and some, if not most, of them will be wrong, flawed, or outdated.
- Since SwiftUI is still evolving fast, the code you write now will likely need some TLC 6+ months from now. For example, if you worked around the lack of collection views last year, you will likely want to update your implementation to make use of the new grid views. Time that could be saved if you waited for things to mature.

# Migration concerns
I read this [blog post on migrations](https://lethain.com/migrations/) a while back, and it has really stuck with me. Migrations are tricky and harmful if half done or done poorly.

Back in the Swift 2 days, I was working on an Objective-C app. We ended up deciding that all new code should be in Swift. This was exciting for me since I wanted to write Swift as much as possible; however, this turned out to be more of a distraction than anything. We were all actively learning Swift while migrating this existing code base over to Swift. As a result, the code that we produced was a mixture of the different evolving opinions around what was _Swifty_ mixed with Swift that looked like Obj-C when we converted code over. Combine that with pushing it to interop with our opinionated Obj-C architecture and the Swift 3+ changes. The result? Our code base was difficult to move in and buggy to say the least. Ultimately, we ended up distracted from the main goal of quickly delivering stable features to our users and improving the company.


This kind of parallels where SwiftUI is today. We are all figuring out what is _SwiftUI-y_, converting our UIKit/AppKit code over is not a direct port, pushing SwiftUI to interop with UIKit/AppKit and existing code, and adapting to framework changes as they come up.

Also, when doing a migration like this, you would hope that you can leverage tests as a safety net (even if they are throw away). The testing story for SwiftUI is not that clear right now. The best I can come up with for now is throwing down a _safety net_ of UI snapshot tests to validate that the refactor to SwiftUI is visually correct.

# But what about the new widgets?
Yeah hands are tied, use SwiftUI! This will be a nice way to start getting used to SwiftUI development :).

---

# Summary
Like I said, SwiftUI is awesome and clearly the future of UI development on Apple's platforms. As such, we should all be keeping an eager eye on SwiftUI and try and use it where it makes sense. Let's just be mindful about it becoming a distraction from the reason why we write software — our end users.

---

p.s. If you are looking for some references, here are some good ones that come to mind:
  + [objc.io](https://www.objc.io)'s book [Thinking in SwiftUI](https://www.objc.io/books/thinking-in-swiftui/)
  + [Dim Sum Thinking](https://dimsumthinking.com)'s book [A SwiftUI Kickstart](https://gumroad.com/l/swiftuikickstart)
  + [Composable Architecture](https://github.com/pointfreeco/swift-composable-architecture) from the folks at [Point-Free](https://www.pointfree.co)
  + [Hacking With Swift](https://www.hackingwithswift.com)'s [100 Days of SwiftUI](https://www.hackingwithswift.com/100/swiftui)

---

Thank you for reading!

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
