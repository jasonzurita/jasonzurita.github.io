---
title: "Add iOS App to Previously Independent watchOS App"
layout: post
date: 2021-5-21
image: /assets/images/posts/add-ios-app-to-previously-independent-watchos-app.png
headerImage: true
tag:
- Xcode
- iOS
- watchOS
- App Store
star: false 
category: blog
author: jasonzurita 
description: Add iOS App to Previously Independent watchOS App
---

# Summary

I have an [independent watchOS app](https://apps.apple.com/us/app/spec-golf-swing-analyzer/id1479731740?mt=8) (currently only runs on the watch without any iOS component). In a kind of backwards series of events, I wanted to add an iOS companion app 🙃. I say _backwards_ because most of the time people want to add a watch counterpart app. You might guess that this is a straight forward thing to do. Well I have some very real scar tissue to show as proof that it is not. Since adding an iOS app to an existing independent watchOS app is a bit unusual, solving this was more or less un-Googleable. Hopefully this post helps some other poor souls like me.

---

# The Plan Forward

## A Little More Background
As most things that touch code signing and app configuration related (e.g., Info.plist, targets, build config, etc.), I spent a crazy amount of time banging my head against the wall trying to figure this out. I even called _Apple Setup and Distribution_ [support](https://developer.apple.com/contact/topic/select). Although very nice and eager to help, this issue seemed a little too in the weeds for them. Their suggestion was to create a new app in the App Store with a note in the description that there is an Apple Watch companion app. This didn't sit well for a number of reasons:
- There would be two listings in the App Store, which would be confusing to potential customers.
- Due to the two listings, I would have to maintain the metadata separately (e.g., description), which would likely end up getting out of sync.
- I wouldn't be able to share in-app purchase subscriptions between apps (iOS & watchOS apps). For example, if there is a "pro" feature in my watchOS app, having the "pro" level unlocked in the iOS app would be difficult. And, in-app purchases are difficult enough as is.
- No automatic downloading of the watchOS app if the iOS app is downloaded, which is the expected behavior of users downloading applications.
- When viewing the iOS app in the App Store, the listing wouldn't automatically include that a watchOS app is available like other listings with both apps. This is further confusing and possibly leads to a less appealing application to download (i.e., only showing support for iOS even though a watchOS app exists).
Help with this would be greatly helpful as we want to provide our users with the best possible experience!

## The Solution

- The way independent watchOS apps are configured, they include a kind of "shell" carrier target. This could be wrong, but my hunch is that since independent watchOS apps live in App Store Connect under iOS apps they still need something to _fake_ the iOS target. The solution from a high level is:
- to replace this _shell_ target with a regular iOS app
- embed the watchOS app in the newly created iOS app
- Mess with Info.plists until the Xcode is satisfied


---

# Summary


---

Thanks for reading, and feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!

---


