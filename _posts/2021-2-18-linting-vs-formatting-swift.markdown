---
title: "Linting vs Formatting: A Swift Comprehensive Guide"
layout: post
date: 2021-2-12
image: /assets/images/posts/lintingvsformatting/logo.png
headerImage: true
tag:
- Swift
- Xcode
- Linting
- Formatting
- Code style
star: false 
category: blog
author: jasonzurita 
description: Linting vs Formatting - A Swift Comprehensive Guide
---

<!-- 
TODO:
- logo
- Check links
- title _vs_ or _&_
- In general is the title good?
<p style="text-align:center;"><img src="/assets/images/posts/watchos_testing/1.png" alt="Screenshot showing the options to pick for the local swift package" width="375"/></p>
-->

# Summary
From a high level, linting and formatting are meta-processes that you apply to your code to improve code quality, consistency, style, and to reduce [bikeshedding](https://en.wiktionary.org/wiki/bikeshedding).  With that said, the subject of _linting_ and _formatting_ code is an interesting one. These terms tend to be conflated together to mean the same thing, but the distinction is useful to make and understand. Why? As programmers, we are constantly translating our knowledge of the world into code and processes. Without knowing differences between things (even if subtle), how can we use it most effectively?

_Note: If you want to skip all the explanation and jump to something more concrete, check out [A recommended setup](#a-recommended-setup) below._

---

# Linting vs Formatting
Linting and formatting are useful to enforce coding style and conventions, so that you and your team can better focus on the more meaningful parts of your code. Below are some Swift style guides that linting and formatting tools attempt to codify.
- [Swift.org](https://swift.org/documentation/api-design-guidelines/)
- [Github](https://google.github.io/swift/)
- [Swift Style: An Opinionated Guide to an Opinionated Language](https://www.amazon.com/Swift-Style-Opinionated-Guide-Language/dp/1680502352/ref=sr_1_1?dchild=1&keywords=Swift+Style%3A+An+Opinionated+Guide&qid=1615490806&sr=8-1)
- [Ray Wenderlich](https://github.com/raywenderlich/swift-style-guide)
- [Airbnb](https://github.com/airbnb/swift)
- [Linkedin](https://github.com/linkedin/swift-style-guide)

## Why is the difference important?
As mentioned, we as programmers translate our understanding of things into code and processes. In order to make this translation, our definitions and understandings need to be the same between us and other sources of information such as other developers, stakeholders, and documentation — we need to _come to terms_. If our definition and understanding are not the same, then there's potential that expectations are not aligned and the outcome of some work isn't what everyone expected, including yourself<sup>1</sup>.

A small but interestingly subtle example is if you are asked to send up a _user id_ to the backend for whatever reason. A typical assumption is to think that this refers to _your business'_ _user id_, but is that really the case? Maybe this means the _user id_ for the analytics provider or another service? Once you get this definition correct, you go and create a function that takes in a _string user id_. Down the line, when this definition is forgotten, you or someone else goes and passes a different _string user id_ to that function...a weird and difficult bug to troubleshoot is introduced<sup>2</sup>. This is what makes naming so hard. Naming is the translation of a purpose into code. In this case, _user id_ is probably the wrong name as it is vague.

So what does this have to do with _linting_ and _formatting_? Getting our definitions and understandings right, coming to terms, applies here too. If we conflate the meaning of these two, when and how should you use one verses the other?

<sub> 1. The title of this blog post is an example. _A Swift Comprehensive Guide_ can mean different things: Is this post Swift language focused? Is this post a quick, but comprehensive guide? The potentially ambiguous nature of this title was intentional 😏.</sub> <br>
<sub> 2. Tagging, aka phantom types, is one possible solution to this problem. For more information check out these resources: [Strings Are Evil](https://www.youtube.com/watch?v=UTm5p96KlEc), [swift-tagged](https://github.com/pointfreeco/swift-tagged).  </sub>

## Linting
[Linting](https://en.wikipedia.org/wiki/Lint_(software)) is the act of automatically analyzing code to determine if the code paths are flawed in some way (e.g., errors, bugs, etc.). Another way to think about linting is that the focus is to identify [code smells](https://en.wikipedia.org/wiki/Code_smell). A key part of that first sentence to focus on is _analyzing code_. Linters are smart and typically require resolution by a human.

Linters are typically focused on the behavior of your code like [cyclomatic complexity](https://realm.github.io/SwiftLint/cyclomatic_complexity.html), force unwrapping, and excessively nested types. With a linting tool in place, developers should have a higher degree of confidence when shipping their code since there is another set of _eyes_ on your code.

Linters have grown over time to include code style and even automated code formatting in some cases.

{ should ^^ be a bulleted list }
- Linting tools can be run manually, integrated directly into Xcode to run automatically, or automatically as part of your continuous integration (CI) system

#### Available Tools
[SwiftLint](https://github.com/realm/SwiftLint)
- Related talk: [Watch Your Language!](https://academy.realm.io/posts/slug-jp-simard-swiftlint/)
- Generally the most used linter in the community.
- Makes use of Clang and SourceKit to use the AST representation (uses SourceKitten)
- SwiftLint is based on rules which you can enable/disable and add your own rules.
  + You can disable rules either globally or in specific cases. Useful when you are knowingly breaking a rule.
- You can define custom rules.
  + e.g., [Identifying issues when using the Firebase SDK](https://medium.com/google-developers/creating-linting-rules-for-firebase-ec51fcb7623a)
- https://medium.com/google-developers/creating-linting-rules-for-firebase-ec51fcb7623a
  + Firebase has a fork they are testing that provides rules to let you know if you are integrating their SDK incorrectly.
- Does increase compile time due to its use of SourceKit (i.e., Swift code needs to be turned in to Swift AST)
- SwiftLint makes use of the Swift Compiler and therefore takes some time to execute. Having SwiftLint tied as a run script will slow down the compile time. I measured about 5-10s per build.

## Formatting
Style and conventions:
- Formatting helps keep the code base clean so that the structure is easily read.
- Formatting is primitive enough that it can typically be done by a machine.
- By having a consistent code style, the code will be easier to read code because the formatting is the same for everyone.

### Tools
[Swift Format](https://github.com/nicklockwood/SwiftFormat)
- Reformats Swift code by applying rules.
- Leaves meaning intact.
- Integration with:
  + Command Line
  + Xcode source editor extension (Editor > SwiftFormat)
  + As an Xcode build phase
  + As a Git pre-commit hook
- SwiftFormat first converts the source file into tokens, then iteratively applies a set of rules to the tokens to adjust the formatting. The tokens are then converted back into text.
- SwiftFormat's configuration is split between rules and options. Rules are functions in the SwiftFormat library that apply changes to the code. Options are settings that control the behavior of the rules.
- Does not use SourceKit. This uses a custom Swift parser that is faster than using SwiftLint.

#### Resources
- Swift coder's podcast with nick lockwood

# A recommended setup
- How to use linting and formatting (local vs CI)

- { breakdown different workflows for different size teams / CI vs no-CI, etc. }
- Include any of the linters/formatters as a stand alone build phase run script.
- Keep the L&Fs as command line only.
- Use SwiftLint on CI to enforce style and conventions like "no implicit unwrapping". use SwiftFormat on local machine tied to either a different scheme or command line only. Run SwiftFormat once when first starting to use it and every now and again (with everything commited) and take a look at the diff to make sure everything is good.
  + Some people tie SwiftFormat to when running tests
  + SwiftLint can be removed as a dependency (build time will be faster).

- Danger plugin
  + Ruby gem that runds in CI during pull request/merge request process
  + Higher level "linting" where we can make sure PR sizes are kept to a certain size, unit tests are included in a PR of x size, common typos.
  + Danger can leave messages, comments, or even fail CI builds.
  + Integrating it with SwiftLint and we can have SwiftLint warnings as inline comments in the PR.

---
to sort resources
---

- Available tools (include Swift's built in linter + new community push)
### Make your own
Home grown
- Erica Sadun has a great blog post about making your own [link](http://ericasadun.com/2015/05/18/swift-alternative-lintage/)

### Other notable mentions
[Tailor](https://github.com/sleekbyte/tailor)
- Doesn't seem be be in active development
- Written in Swift
- Cross platform linter (Mac OS X, Xcode, Linux, Windows)
- Supports different out of the box style guides.
- Tailor parses Swift source code using the primary Java target of ANTLR
- Several different types of output such as inline, json, HTML, etc.
- The git repo doesn't look as active as SwiftLint.

Taylor
- Checks conformance to code metrics like excessive class length, too many methods, n-path complexity, etc.
- Built on SourceKitten

Swift-Clean
- More comerical (Mac app)
- Community vote based for rules (optional opt in though)

SonarQube
- Open source platform for CI code quality.
- Commerical for Swift (expensive...)

Checkmarx
- Security platform linter
- Converts Swift code to Objective-C, then scans it.

Swimat (https://github.com/Jintin/Swimat)
- Xcode plug-in to format your Swift code.
- Smaller open source project.

Swift language
- Swift language itself has a limited formatter built in.
- Not much activity with the native formatter

- https://github.com/apple/swift-evolution/blob/master/proposals/0250-swift-style-guide-and-formatter.md


---

# Additional thoughts

---

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
