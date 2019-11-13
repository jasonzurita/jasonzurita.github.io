---
title: "Bare Minimum for Apple Watch Communication"
layout: post
date: 2019-11-13
image: /assets/images/posts/bare-minimum-apple-watch-communication-logo.png
headerImage: true
tag:
- Apple Watch
- watchOS
- iOS
star: false 
category: blog
author: jasonzurita 
description: Bare Minimum for Apple Watch Communication

---

# Summary
When recently starting to add an Apple Watch companion app to an existing iOS app, I wanted to get communication working between the phone and watch as fast as possible since the rest of the Apple Watch app's functionality depended on the inter-app communication.

Like other critical path features, this key part of the implementation needed to be assessed for strengths and weaknesses at the beginning, or the rest of the Watch app would be at risk of being fundamentally flawed — potentially lots of wasted time.

There is a lot of great material out there about getting started with Apple Watch development / communication to other devices, but I wanted to help supplement that with the bare minimum implementation to help other people, like me, get up and running as quickly as possible!

A few things to note:
- This post is not about the strengths and weaknesses of Apple Watch inter-app communication.
- The inter-app communication implementation below is quick and dirty, but there is [value in working software](https://agilemanifesto.org/principles.html).
  + Use the learnings from this minimum viable feature (MVF) to iterate!
- The below implementation is based on [this commit](https://github.com/jasonzurita/BabyPatterns/commit/0d2b465163fcdd25369a82af0b5631fc627ea383), which was part of adding Apple Watch support to [Baby Patterns](https://apps.apple.com/us/app/baby-patterns/id1404068130?mt=8).
  + This app is [open sourced](https://github.com/jasonzurita/BabyPatterns) and the [Watch implementation uses SwiftUI](https://twitter.com/jasonalexzurita/status/1192500817489747968?s=20)!

---

# The Bare Minimum Implementation
_Throw the communication logic directly where it will be used!_

Initially I wanted to send some information between Apple Watch app and iPhone app so the iPhone app could perform an action. There are many ways to overthink the implementation of this feature, but forget all that and put the communication logic _right_ where you need it! Then, after things are working, clean this up (like separate out the communication logic). Or, if you aren't looking for anything more sophisticated, you are done 😜.

### Main app's implementation (iPhone app in my case)

{% highlight swift %}
// 1
import WatchConnectivity

// `ReceivingEntity` is the class where you want to receive communication
final class ReceivingEntity: NSObject {
    override init() {
        super.init()
        . . .
        // 2
        if WCSession.isSupported() {
            let session = WCSession.default
            session.delegate = self
            session.activate()
        }
    }

// 3
extension ReceivingEntity: WCSessionDelegate {
    public func session(_: WCSession, activationDidCompleteWith _: WCSessionActivationState, error: Error?) {
        if let e = error {
            print("Completed activation with error: \(e.localizedDescription)")
        } else {
            print("Completed activation!")
        }
    }

    public func sessionDidBecomeInactive(_: WCSession) { print("session did become inactive") }
    public func sessionDidDeactivate(_: WCSession) { print("session did deactivate") }

    // 4
    public func session(_: WCSession,
                        didReceiveMessage message: [String: Any],
                        replyHandler: @escaping ([String: Any]) -> Void) {
        print("message received! - \(message)")

        // 5
        guard let m = message as? [String: String] else {
            // 6
            replyHandler([
                "response": "poorly formed message",
                "originalMessage": message,
            ])
            return
        }

        // 7
        replyHandler([
            "response": "properly formed message!",
            "originalMessage": m,
        ])

        // 8
        if m["someKey"] == "someValue" {
            // do stuff
        }
        . . .
    }
}
{% endhighlight %}

1. Make sure to import the watch connectivity framework.
1. You should check to see if watch communication is supported.
  + If supported, you need to set yourself as the delegate to receive communication.
  + Don't forget to activate the session. **Without this, no communication will work!**
1. Conform to the `WCSessionDelegate` to receive communication from the Watch app and to receive information about the session activation.
1. Use this delegate method to receive messages from the Watch app.
  + Although there are other delegate methods to receive messages in slightly different ways, I initially chose this one due to the flexibility of dictionaries and because the reply handler helps when debugging.
1. Expect the incoming message to be `[String: String]`.
1. If not `[String: String]`, send a helpful message back in the reply handler.
1. If we have `[String: String]`, reply to the Watch app so it knows the message was formed well.
1. Pull values out of the message and do what you want based off of that!

### The Apple Watch's implementation

Similar to the above iPhone session setup, update your Apple Watch's Watch Extension's _ExtensionDelegate.swift_ to include:
{% highlight swift %}
import WatchConnectivity
import WatchKit

// 1
final class ExtensionDelegate: NSObject, WKExtensionDelegate {
    func applicationDidFinishLaunching() {
        if WCSession.isSupported() {
            let session = WCSession.default
            session.delegate = self
            session.activate()
        }
    }
}

extension ExtensionDelegate: WCSessionDelegate {
    func session(_: WCSession, activationDidCompleteWith _: WCSessionActivationState, error: Error?) {
        if let e = error {
            print("Completed activation with error: \(e.localizedDescription)")
        } else {
            print("Completed activation!")
        }
    }
}
{% endhighlight %}

1. I chose to put the communication initialization setup work here since it should be early in the lifecycle of the Watch app in order to receive and support sending communication to the main app (iPhone app in my case). Later on, you will likely want to move this out of the _ExtensionDelegate_, but fine for an initial pass to get thing working.

## And wherever you want to send a message to the main app
{% highlight swift %}
// 1
import WatchConnectivity

func sendMessage() {
    // 2
    guard WCSession.default.isReachable else { return }

    // 3
    WCSession.default.sendMessage(
        ["messageKey": "messageValue"],
        replyHandler: { reply in print(reply) },
        errorHandler: { e in
            print("Error sending the message: \(e.localizedDescription)")
    })
}
{% endhighlight %}

1. Make sure to import the connectivity framework.
1. Check to see if the iPhone is reachable before trying to send a message.
1. Sending a message is as simple as calling the `sendMessage` function.

Note: To see this being used, check out [these lines](https://github.com/jasonzurita/BabyPatterns/commit/0d2b465163fcdd25369a82af0b5631fc627ea383#diff-6802035004502da2cc9b8250980b7156R89-R93) from Baby Patterns.

---

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
