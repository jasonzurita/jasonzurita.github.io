---
title: "Linting vs Formatting: A Swift Guide (Part 1)"
layout: post
date: 2021-6-25
image: /assets/images/posts/linting-vs-formatting-logo.png
headerImage: true
tag:
- Swift
- Linting
- Formatting
- Code style
star: false 
category: blog
author: jasonzurita 
description: Linting vs Formatting - A Swift Guide (Part 1)
---

<sub> Aside: I ran a workshop for [try! Swift World](https://www.tryswift.co/world/) on this subject. If interested, [the resources are freely available](https://github.com/jasonzurita/talks/tree/master/Swift%20Linting%20and%20Formatting%20Workshop) including a guide that you can follow!</sub>

# Summary

#### Note: This is the first part of two posts on the subject of linting and formatting. After being surprised by how much was floating around in my head, I decided _Part 1_ (this post) should be focused on the "why", and _Part 2_ will be on the "what and how" (i.e., more practical including tools and recommended setup).

From a high level, linting and formatting are meta-processes that you apply to your code to improve code quality, consistency, style, and to reduce [bikeshedding](https://en.wiktionary.org/wiki/bikeshedding) (even with yourself 😅).  The subject of _linting_ and _formatting_ code is an interesting one. These terms tend to be conflated together to mean the same thing, but the distinction is useful to make and understand. Why? As programmers, we are constantly translating our knowledge of the world and the way things work into code and processes. Without knowing differences between things (even if subtle), how can we use them effectively?

_Note: Part 2 will include a recommended iOS/Swift linting and formatting setup including Xcode and CI/CD system._

---

# Linting vs Formatting
Linting and formatting are useful to enforce coding style and conventions, so that you and your team can better focus on the more meaningful parts of your code. Below are some Swift style guides and code conventions that linting and formatting tools attempt to codify.
- [Swift.org](https://swift.org/documentation/api-design-guidelines/)
- [Github](https://google.github.io/swift/)
- [Swift Style: An Opinionated Guide to an Opinionated Language](https://www.amazon.com/Swift-Style-Opinionated-Guide-Language/dp/1680502352/ref=sr_1_1?dchild=1&keywords=Swift+Style%3A+An+Opinionated+Guide&qid=1615490806&sr=8-1)
- [Ray Wenderlich](https://github.com/raywenderlich/swift-style-guide)
- [Airbnb](https://github.com/airbnb/swift)
- [Linkedin](https://github.com/linkedin/swift-style-guide)

## Why is the difference between the two important?
As mentioned, we as programmers translate our understanding of things into code and processes. In order to make this translation, our definitions (and understanding) need to be the same between us and other sources of information such as other developers, stakeholders, and documentation — we need to _come to terms_. If our definitions are not the same, then there's potential that expectations are not aligned and the outcome of some work isn't what everyone expected (an impedance mismatch).<sup>1</sup>

A small but interestingly subtle example is if you are asked to send up a _user id_ to your backend for whatever reason. A typical assumption is to think that this refers to _your business'_ _user id_, but is that really the case? Maybe this means the _user id_ for the analytics provider or another service? Once you get this definition correct, you go and create a function that takes in a _string user id_. Down the line, when this definition is forgotten (which happens faster than you might think), you or someone else goes and passes a different _string user id_ to that function...a weird and difficult bug to troubleshoot is introduced.<sup>2</sup> This is what makes naming so hard. Naming is the translation of a purpose into code. In this case, _user id_ is probably the wrong name as it is vague.

So what does this have to do with _linting_ and _formatting_? Getting our definitions and understanding right, coming to terms, applies here too. If we conflate the meaning of these two, when and how should you use one verses the other?

<sub> 1. The title of this blog post is an intentional example. _A Swift Guide_ can mean different things: Is this post _Swift language_ focused? Is this post a _quick_ guide to linting and formatting?</sub> <br>
<sub> 2. Tagging, aka phantom types, is one possible solution to this problem. For more information check out these resources: [Strings Are Evil](https://www.youtube.com/watch?v=UTm5p96KlEc), [swift-tagged](https://github.com/pointfreeco/swift-tagged).  </sub>

## Linting
[Linting](https://en.wikipedia.org/wiki/Lint_(software)) is the act of analyzing your code to determine if that code is potentially flawed in some way. Another way to think about linting is that the intent is to flag [code smells](https://en.wikipedia.org/wiki/Code_smell). Therefore, the nature of the insights that linters provide typically requires human intervention. Some linters do provide some automatic _fixing_ functionality, but like mentioned above, we should focus on using a linter for its intended purpose to get the most out of the tool.

Some examples of what a linter can flag: [cyclomatic complexity](https://realm.github.io/SwiftLint/cyclomatic_complexity.html), force unwrapping, and excessively nested types, etc. 

With a linting tool in place, developers can have a higher degree of confidence when shipping their code since there is another set of _eyes_ on your code.

## Formatting
Where linting typically focuses on the behavior of your code, formatting focuses on your [code's style](https://en.wikipedia.org/wiki/Programming_style). At first glance, formatting may seem unimportant, but formatting helps keep a code base consistent so that the code is more easily read and understood. Think about where you store your utensils. If they were all mixed together, you would have to think slightly harder to find something. Now extend that to two drawers with the same items. Let's say this time they were organized, but laid out differently, you would have to first reorient yourself when searching for things between drawers before going about finding what you want. This may not amount to a lot of time, but there is still mental overhead required and we have a finite amount of that! Think about all the different areas of a code base. If they were styled differently, you will have a harder time reading and understanding the code, and that would likely take a mental toll on you without you even realizing it.

It is also worth noting that formatting can typically be done automatically since we are only focused on the _style_ of the code.

---

# Conclusion

It's worth noting that not all linter and formatter tools follow the strict definitions. For example:
- [SwiftLint](https://github.com/realm/SwiftLint) can do some automatic formatting. I usually don't use that capability since I've had issues where SwiftLint thought some code should be formatted one way and the formatting tool I was using thought the same code should be formatted another way 🙃. I could have probably configured the two to agree, but I opt to use these tools for their intended purposes.
- Despite the name, Kotlin's [ktlint](https://github.com/pinterest/ktlint) is more of a formatter than a linter. So in cases like this, a little more mental overhead is required when evaluating and using the tools.

— — 

This post was more philosophical in nature, but I find giving our craft thought on many levels to be enjoyable, insightful, and applicable to more areas than just the subject at hand. Many times, including this discussion on linting and formatting, I find that the concepts can be applied to other areas!

With all this said, _Part 2_ will be less thinking and more doing! Until then 👋.

---

Thanks for reading, and feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!

---


