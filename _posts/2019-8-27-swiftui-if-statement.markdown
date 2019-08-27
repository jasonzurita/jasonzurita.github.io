---
title: "SwiftUI - The If Statement's Sharp Edges"
layout: post
date: 2019-8-27
image: /assets/images/posts/swiftui_if_statement/if_swiftui_logo.png
headerImage: true
tag:
- Swift
- SwiftUI
- iOS
star: false 
category: blog
author: jasonzurita 
description: SwiftUI - The If Statement's Sharp Edges
---

---

<img src="/assets/images/posts/swiftui_if_statement/conditional_error.png" alt="Xcode error message: Closure containing control flow statement cannot be used with Function Builder 'ViewBuilder'" width="650"/><br>
_Looking for the tl;dr? Skip down to the [Workarounds](#workarounds) section below._
{: style="color:gray; font-size: 95%; text-align: center;"}

<br>
# Summary
In working with the SwiftUI domain specific language (DSL), you will see some familiar syntax like `forEach` and `if <boolean condition> { }`. In the SwiftUI DSL, Apple is leveraging already understood syntax like this to provide some continuity between day-to-day Swift code and the SwiftUI DSL — This is great until it isn't 😅.

You may notice some odd behavior where your Swift code doesn't quite work the same in SwiftUI. For example, if you are using an _if statement_ with a simple _boolean_ check to optionally show a view, everything works just fine. If you were to take the conditional check one step further to something familiar like evaluating an enum or to use the `if let` syntax to conditionally unwrap an optional, you will be left scratching your head. This is exactly what happened to me, and this post is about exploring the sharp edges around SwiftUI's use of the _if statement_.

_Note: At the time of this writing, SwiftUI is still in beta and changing a...lot... This post is meant to highlight some of the edge cases, workarounds, and to help contribute to the collective understanding about this new and exciting technology._

---

# What's the deal with the _if statement_
## A little background
If you pass SwiftUI an optional view, and it happens to be `nil`, SwiftUI will not render that view. This is useful if you only want a view to be visible _sometimes_.

You can make use of this in the body that produces a SwiftUI view by using a simple _if statement_. What SwiftUI will do is take the _if statement_ and produce optional views that are `nil` or `not nil` depending on the conditional logic.

<p style="text-align:center;"><img src="/assets/images/posts/swiftui_if_statement/conditional_swiftui_views.png" alt="Screenshot showing a different SwiftUI view when tapped" width="475"/></p>

{% highlight swift %}
struct MaybeDuckView: View {
    @State  var showDuck: Bool = true

    var body: some View {
        VStack {
            // SwiftUI if statement that will conditionally show one of these views
            if showDuck {
                Text("🦆")
            } else {
                Text("🐘")
            }
            // Using the same boolean to change the string
            Text(showDuck ? "Quack" : "Quack?")
                .font(.system(size: 24))
        }
        .font(.system(size: 40))
        .gesture(TapGesture().onEnded { _ in
            self.showDuck.toggle()
        })
    }
}
{% endhighlight %}

_Side note: One strength of the if statement is that if SwiftUI is listening to the conditional statement (e.g., via the `@State` property wrapper, a binding, etc.), SwiftUI will reevaluate that if statement to choose the right view to show when the state changes._

## The problem
What if we want to use other forms of the if statement like `if let` or `if case`? Sadly, they don't work 😕.

SwiftUI builds views using the `ViewBuilder` struct, which leverages a new Swift feature called _Function Builders_. 

---

_Aside:_ To learn more about Function Builders:
- [Swift evolution draft proposal](https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md) to add Function Builders to the Swift Language.
- [Swift forums discussion](https://forums.swift.org/t/function-builders/25167) of Function Builders and the draft proposal.
- [Blog post](https://www.swiftbysundell.com/posts/the-swift-51-features-that-power-swiftuis-api) by John Sundell about SwiftUI features that power SwiftUI's API, that includes details about Function Builders (The post notes that these are all Swift 5.1 features, but Function Builders may not end up being part of Swift 5.1).

---

The implementation of `ViewBuilder` includes support for _if statements_, but as seen in the below screenshot there is only support for the basic _if statement_ control flows (i.e., if, if/else, if/else if). 
- If you press `cmd + shift + o` in the Xcode 11 beta and type in `ViewBuilder`, you can see the definition for the `ViewBuilder` struct.

   <p style="text-align:center;"><img src="/assets/images/posts/swiftui_if_statement/buildif.png" alt="Screenshot of Apple's definition for SwiftUI ViewBuilder if statement support" width="700"/></p>

Looking at the statement that reads, _"...that is visible only when the `if` condition evaluates `true`..."_, tells us that the _if statement_ condition needs to be a boolean result, which sadly doesn't include the other _if statement_ variations.

For example, we cannot use the `if case` style or the `if let` styles (switch statements won't work either).

This will not compile ❌:
{% highlight swift %}
enum FeatureState {
    case active
    case disabled
}

struct FeatureView: View {
    @State var featureState: FeatureState

    var body: some View {
        VStack {
            // Will not compile. Error message:
            // Closure containing control flow statement cannot be used with function builder 'ViewBuilder'
            if case .active = featureState {
                Text("Active view!")
            } else {
                Text("Disabled view")
            }
        }
    }
}
{% endhighlight %}
<sub>Note: the `VStack` is used as a container to guarantee to the compiler that a view will be returned.</sub>

In digging more into this, I came across this interesting statement from the [Function Builder draft proposal](https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md) (seen at the bottom of the proposal), which addresses this limitation. Seems like Function Builders will — by design — provide support for a more general `buildOptional`, and leave the specifics, like supporting different flavors of if statements, up to the implementations:
> The function-building methods in this proposal have been deliberately chosen to cover the different situations in which results can be produced rather than to try to closely match the different source structures that can give rise to these situations. For example, there is a buildOptional that works with an optional result, which might be optional for many different reasons, rather than a buildIf that takes a condition and the result of applying the transformation to the controlled block of the if...it is not be possible to come up with a finite set of such rules that could be soundly applied to an arbitrary function in Swift. For example, **there would be no way to to apply that buildIf to an if let condition:** for one, the condition isn't just a boolean expression, but more importantly, the controlled block cannot be evaluated before the condition has been (uniquely and successfully) evaluated to bind the let variable. **We could perhaps add a buildIfLet to handle this, but the same idea could never be extended to allow a buildIfCase or buildReturn.**...

And although these other types of control flow are not currently supported in SwiftUI, we will hopefully see better parity between SwiftUI DSL control flow and the Swift language control flow in the future; [As noted by Apple in Swift forums](https://forums.swift.org/t/function-builders/25167/345), which is promising 💪:
> ...The future SwiftUI DSL will be the current SwiftUI DSL but without as many restrictions, and it will deploy backwards, so remembering the restrictions (and differences like buildOptional vs. buildIf) will be pointless.

{: #workarounds}
## Workarounds
If you can't wait, like me, for SwiftUI control flow to gain additional functionality, we have some options.

- Pull the _boolean_ producing logic into a function or computer property (continuing the example from above):
   ```swift
struct FeatureView: View {
    @State var featureState: FeatureState

    private func shouldShowDetailView(for state: FeatureState) -> Bool {
        switch state {
        case .active: return true
        case .disabled: return false
        }
    }

    var body: some View {
        VStack {
            if shouldShowDetailView(for: featureState) {
                Text("Active view!")
            } else {
                Text("Disabled view")
            }
        }
    }
}
```

- Use a function or computed property to return the view you want to conditionally show:
   ```swift
struct FeatureView: View {
    ...
    private var featureDetailView: some View {
        switch featureState {
        case .active: return Text("Active view!")
        case .disabled: return Text("Disabled view")
        }
    }

    var body: some View {
        featureDetailView
    }
}
```

---

# Thoughts
- One of the more subtle, and tricky, things about SwiftUI, which also is one of the strengths of the SwiftUI DSL, is the continuity in syntax between the Swift language and SwiftUI DSL. The thing to keep in mind, is that even if the syntax is the same, don't just assume that all the behavior is the same.
  + Another example is `forEach`. You can read more about `forEach` in this [Hacking With Swift Post](https://www.hackingwithswift.com/quick-start/swiftui/how-to-create-views-in-a-loop-using-foreach).
- Although SwiftUI has its youthful sharp edges, embracing SwiftUI for what it is today, including the head-scratching moments, can yield a lot of great learning opportunities like this one; all while setting us up to be highly productive as SwiftUI matures.
- Overall, I am excited about the future of SwiftUI!

---

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
