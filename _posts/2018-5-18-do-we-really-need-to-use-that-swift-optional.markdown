---
title: "Do We Really Need To Use That Swift Optional?"
layout: post
date: 2018-5-18
image: /assets/images/profile2.jpg
headerImage: false
tag:
- iOS
- Swift
- Optionals
star: false 
category: blog
author: jasonzurita 
description: Reduce the use of optionals
---

# Summary
I love optionals as much as the next person, but optionals are overused. One common reason is that they are easy to throw at problems where an entity (class, struct, enum) may not exist (i.e., `nil`). Then over time, we conflate the meaning of the `nil` case: is the optional `nil` due to json processing?, user input?, view properties that need to get set after initialization?, and so on. Often we don't even think twice about why we are using an optional; perhaps we are a bit blinded by Swift being _type-safe_. The type-safety in Swift is great and optionals help us write _safer_ code, but we shouldn't use that to ignore what it takes to write higher quality code. My hope is for us to better evaluate the need of using an optional to produce clearer and more maintainable code!

---
## Optionals are great
A great thing about optionals is that they force us to deal with the existence of an entity (class, struct, enum) that may not exist. After addressing that entity's existence (unwrapping, using a higher order function like map, optional binding, etc.), then that entity is for the most part guaranteed at compile time to be safe to use, unless you like to use implicitly unwrapped optionals (IUOs) 😦. For a simple example, we are able to safely access values in a dictionary using optionals. If a key exists, we are able to use the returned value with confidence:

{% highlight swift %}
typealias MenuItem = String // typealiases like this make code easier to read :)
let bill: [MenuItem: Int] = [:]
let greedyMarkup = 0.5 // 50%

// can we safely get soda from the bill?
if let sodaWholesaleCost = bill["soda"] {
    let sodaCost = Double(sodaWholesaleCost) * (1.0 + greedyMarkup)
    . . .
}
{% endhighlight %}

---
## Optionals are overused

#### Example 1:
Although optionals are great, they are often overused/relied on. A common example is the use of an optional bool (`Bool?`):

{% highlight swift %}
func nextCheckoutStep(_ shouldShowPopup: Bool?) {
    if let shouldShow = shouldShowPopup {
        if shouldShow {
            showCompletionPopup()
        } else {
            moveToNextStep()
        }
    } else {
        showErrorPopup()
    }
} 
{% endhighlight %}

The `bool?` would be much better as an enum in this case!:

{% highlight swift %}
enum CheckoutState {
    case complete
    case inProgress
    case error // we could provide an assoicated value to know the failure reason
}

func nextCheckoutStep(_ checkoutState: CheckoutState) {
    switch checkoutState {
        case .complete:
            showCompletionPopup()
        case .inProgress:
            moveToNextStep()
        case .error:
            showErrorPopup()
    }
} 
{% endhighlight %}

Using an enum instead of an optional bool lets us more clearly define our intention, and there is no more ambiguous: _what does a bool that is nil mean really mean?_. We also get the benefit of the compiler telling us if we change the enum to include more or less cases due the requirement for exhaustive switch statements! Not only is this safer going forward, but our code becomes easier to reason about and modify later.

#### Example 2:
Should that passed in function parameter be an optional?

{% highlight swift %}
struct UserProfile {
    name: String
    email: String
    phone: String
}

final class CustomView: UIView {
    // internal properties omitted
    init(profile: UserProfile?) {
        nameLabel.text = profile?.name ?? "john smith"
        emailLabel.text = profile?.email ?? "john@example.com"
        phoneLabel.text = profile?.phone ?? "555.555.5555"
    }
}
{% endhighlight %}

The problem with this is more subtle than the `Bool?` situation. Here, the `CustomView` is making an assumption as to the state of this view if the user profile doesn't exist; the assumption made here is to set specific default strings. By making an assumption like this, future developers (included you) may not know how to interpret what a nil profile means, which makes working with this code harder and more prone to bugs.

The optional may originally have been used to signify the initial loading of this view when no user is logged in, but what if we then go to use this view as an avatar's metadata in a social feed? Then a nil profile could be expanded to mean that there is an error with the data that came from the server. The above code could easily turn into:

{% highlight swift %}
struct UserProfile { . . . } 

final class CustomView: UIView {
    init(profile: UserProfile?) {
        if let p = profile {
            if p.name.isEmpty || p.email.isEmpty || p.phone.isEmpty {
                nameLabel.text = profile?.name ?? "n/a"
                emailLabel.text = profile?.email ?? "n/a"
                phoneLabel.text = profile?.phone ?? "n/a"

                // or maybe make this `init` a failable initializer and return nil here

            } else {
                nameLabel.text = p.name
                emailLabel.text = p.email
                phoneLabel.text = p.phone
            }
        } else {
            nameLabel.text = profile?.name ?? "john smith"
            emailLabel.text = profile?.email ?? "john@example.com"
            phoneLabel.text = profile?.phone ?? "555.555.5555"
        }
    }
}
{% endhighlight %}

Ugly, right? A clearer and more robust way to handle the different states of this view should be to eliminate the optional and force the decision, more appropriately, up the chain (to the view controller, coordinator, etc.); we should leave the view dumb and simple:

{% highlight swift %}
struct UserProfile { . . . }

final class CustomView: UIView {
    init(profile: UserProfile) {
        nameLabel.text = profile.name
        emailLabel.text = profile.email
        phoneLabel.text = profile.phone
    }
}
{% endhighlight %}

**Other options include:**
- Defining an additional `init()` that generically sets the text labels, but setting the labels to a default other than an empty string is arguably having the view make too much of a decision about what is going to be displayed.
- Exposing the internal labels to let the consumer set the labels directly. Although, I usually try to keep implementation details like this hidden and immutable (the labels), so I can reduce the potential for future unexpected use...🐛s!

**Note:** this doesn't necessarily mean to push the optional up the chain too. In this case, we can probably eliminate the optional all together as the up stream view display decision makers should be different for the login vs social screens. If not, we likely have some other problems on our hands to deal with.

#### Example 3:
Next, when we feel the need to make a property optional, we should think if it is due to need or convenience:

{% highlight swift %}
final class SettingsViewController: UIViewController {
    var currentSettings: AppConfiguration?
    . . .
}

final class HomeViewController: UIViewController {
    . . .
    @objc func openSettings() {
        let settingsVc = // get the settings view controller from a xib or storyboard
        // segue to the settings view controller
    }

    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        // if transitioning to the settings view controller
        settingsVc.currentSettings = settings 
        // `settings` is our app configuration like push notificaiton preferences, account info, etc.
    }
}
{% endhighlight %}

This use of optionals makes working with this property more difficult and makes this part of our app more vulnerable to future bugs. For starters, we should to check to see if `currentSettings` exists every time we use it. Also, `currentSettings` is a var and exposed externally to this class, so someone (including you) can easily change the current settings that the `SettingsViewController` uses. Apple does kind of push us down this route (which is a shame), especially if you are using storyboards + segues...

This would be a more robust implementation:
{% highlight swift %}
final class SettingsViewController: UIViewController {
    private let _currentSettings: AppConfiguration
    
    init(currentSettings: AppConfiguration) {
        // we can't change current settings and we are guaranteed to have one!
        _currentSettings = currentSettings 
        super.init(nibName: nil, bundle: nil)
    }
    . . .
}

final class HomeViewController: UIViewController {
    @objc func openSettings() {
        let settingsVc = SettingsViewController(currentSettings: settings)
        navigationController.pushViewController(settingsVc, animated: true)
    }
}
{% endhighlight %}

Simpler and more straight forward to get the `SettingsViewController` the data that it needs! This may make creating your views a bit more difficult (no segues and you need to manually initialize), but this implementation is less vulnerable to future changes and is easier to reason about what is going on.

---

# Additional Thoughts 
To be clear, I am not advocating for a zero optional policy. I think optionals are great and definitely have their uses when your data may or may not exist, such as in bridging to other languages, networking, user input, and so on.

As seen in the examples, reaching for nested `if statement`s is usually a tip (aka code smell) that you should rethink what you are trying to accomplish.

Also, there are more examples than this (like using implicitly unwrapped optionals or force unwrapping), but the general idea that we should think through why we are reaching for an optional. Taking the easy way out by making something an optional when it doesn't really need to be can lead to a lot of problems down the line. Swift is great in that it forces us to address the existence of an optional entity, but let's not use that as a crutch to write brittle code.

A good additional related read on [Enums And Optionals](http://khanlou.com/2018/04/enums-and-optionals/)


Feel free to reach out, I would appreciate any feedback and additional thoughts!
