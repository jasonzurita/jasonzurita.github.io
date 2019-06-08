---
title: "Websites using Swift and AWS Lambda — Part 1"
layout: post
date: 2019-3-05
image: /assets/images/posts/swift_aws_lambda_website/swift-aws-lambda-logos.jpg
headerImage: true
tag:
- Swift
- Swift Package Manager 
- AWS
- Docker
- Make
- AWS API Gateway
- AWS Lambda
- AWS Layers
star: false 
category: blog
author: jasonzurita 
description: Websites using Swift and AWS Lambda — Part 1 — Overview & Generating HTML/CSS using Swift
---

# Summary
<sub>Want to see this kind of Swift website in action!? Check out [the example website](https://swift-aws-lambda-website.jasonzurita.com), and here is the [Source code](https://github.com/jasonzurita/Swift-AWS-Lambda-Website)!</sub>

Ever since the announcement of the [Swift](https://www.swift.org) language, I have been deeply interested in expanding my use of the language to platforms other than the Apple ecosystem (iOS apps, macOS apps, etc.). This interest combined with taking on more web development at work got me thinking — _Can I make lightweight websites written in Swift that are simple, fun to write, and easy deploy?_ The result:
- A website written in Swift using HTML and CSS [domain-specific languages (DSLs)](https://en.wikipedia.org/wiki/Domain-specific_language), which comes with all the [benefits of using the Swift language](https://swift.org/about/) such as writing safer code that is easy to understand and fun to write.
- ~$0 to host!<sup>**</sup>
  + Hosted on [Amazon Web Services (AWS)](https://aws.amazon.com) using Lambda, Layers, API Gateway, and optionally Certificate Manager.
  + **At the time of this writing, [AWS Lambda](https://aws.amazon.com/lambda/) give you [1 million requests for free](https://aws.amazon.com/lambda/pricing/) per month, and [API Gateway](https://aws.amazon.com/api-gateway/) costs [$3.50 per million requests](https://aws.amazon.com/api-gateway/pricing/) per month. I would be surprised if hosting a website this way costs you much, if anything at all.
- Simple to deploy and update. All you need to do is update your AWS Lambda function!
  + Oh, and the breakdown below will get you set up with a way to do local development.
- A website that is dynamically generated, which means you can put your site together after doing things like querying a database, calling into another Lambda function, or making an API request.
- None of the traditional web JavaScript dependencies to manage, which can be quite overwhelming at times.
- As far as I know, the [example website](https://swift-aws-lambda-website.jasonzurita.com) is the first of its kind in that it uses Swift and AWS Lambda 🤓.

#### Due to the large number of things to discuss, this project will be broken down into two parts: Part 1 — _Making the Swift website_, and Part 2 — _Hosting the website_. <br><br>let's work through the first part!

---

# The Tech Stack

_This  may look like a lot, but don't worry. I will break everything down!_

- [Swift-Html](https://github.com/pointfreeco/swift-html) — A Swift domain specific language (DSL) for writing HTML. This package lets you write your HTML in Swift! Because of this, you get all the power of the Swift type system. This means that the [HTML spec](https://www.w3.org/TR/html52/) is codified to prevent you from writing invalid HTML such as putting a _paragraph_ element in an _ordered list_!

- [Swift-Css](https://github.com/pointfreeco/swift-web#css) — Similar to the _Swift-Html_ package above, but for cascading style sheets (CSS). Other modules in this package are available for use, but they aren't used in this project. I would think that splitting out the CSS part of this package into its own package like the _Swift-Html_ package is on the _to do_ list. Until then, ignore the other modules.

  _Note 1: If you are unfamiliar with HTML and CSS but know iOS development, think of HTML as the UI building blocks (UIViews, UIButtons, etc.) that you can drag around in Interface Builder. Think of CSS as autolayout combined with all the UI polish capabilities like setting background color and text font._

  _Note 2: The previous two packages, are from the folks at [Point-Free](https://www.pointfree.co). Their amazing educational site is about teaching functional concepts using the Swift programming language. One neat thing about their approach to education is that they start from core principles to build up to more complicated topics that they use themselves for their site — [dogfooding](https://en.m.wikipedia.org/wiki/Eating_your_own_dog_food) at its finest_ 🐶_!_

  _**Swift-Html** and **Swift-Web** are great examples of being built up piece-by-piece from scratch in their videos, all while the end result is used to generate the site that you are learning off!_

- [AWS Lambda](https://aws.amazon.com/lambda/) — This is an AWS's solution for serverless computing. You write some code that sits in the cloud and gets called to run. For this project, we will route URL requests to a Lambda that is setup to generate HTML/CSS in Swift to produce a website. One nice thing, is that you are not paying for a server to always be _on_. When called, the Lambda _wakes up_ runs your code and goes away, adapting to the volume of requests coming in. In most cases, you end up paying close to $0! Part 2 of this series will include more details about pricing.

  Note: I use the terms _AWS Lambda_ and _Lambda_ interchangeably.

- [AWS Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) — This is a new offering from AWS to allow you to create your own runtime for Lambdas. In this case, we make a Swift runtime to allow our Swift code to execute when the Lambda is invoked. Before AWS Layers, AWS Lambda functions needed to be written in an AWS Lambda supported language such as Python and Go.

- [AWS-Lambda-Swift](https://github.com/tonisuter/aws-lambda-swift) — This project provides two things:
  1. Makes a Swift _AWS Layer_ runtime for _AWS Lambda_ as mentioned above (more on this in Part 2).
  1. Bridges communication between your Swift code and AWS Lambda.
    + Let's you register a function that will get called when your Lambda function gets called.
    + Handles passing information to you Swift function when your Lambda function gets called.
    + Facilitates getting the response from your Swift function and passing it back to Lambda for further processing. In this case, handles passing back the generated HTML string.

- [API Gateway](https://aws.amazon.com/api-gateway/) — We use this to map a domain name URL that triggers the Lambda function. In this case, we are routing a URL request of [https://swift-aws-lambda-website.jasonzurita.com](https://swift-aws-lambda-website.jasonzurita.com) to trigger a Lambda function that then returns a website!

- [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/) — If you want to have a custom domain name for your website, you will need to establish that you are the owner of that domain for later use in API Gateway. Among other things, AWS Certificate Manager allows you verify domain ownership and therefore have a certificate for your domain. This will give your URL _https_, which denotes that your website/domain is secure. Nothing too much to pay attention to with this other than this is a good thing and is a way to get a custom domain name.

{: #spm-section}
- [Swift Package Manager](https://swift.org/package-manager/) — The Swift Package Manager (SPM) is the de facto way to manage Swift dependencies, build, test, and run Swift projects. To learn more about the ins and outs of the SPM, check out the above link and the below resources:
  + [Example Usage](https://swift.org/package-manager/#example-usage) — great way to get started
  + [GitHub reference](https://github.com/apple/swift-package-manager/blob/master/Documentation/README.md) — useful to learn about the package description API
  + [Product definitions](https://github.com/apple/swift-evolution/blob/master/proposals/0146-package-manager-product-definitions.md) — Learn more about SPM products and why they were introduced

- [Make](https://en.wikipedia.org/wiki/Make_(software)) — Used to simplify building our Swift Lambda function.

- [Docker](https://www.docker.com) — Docker may seem out of place here, but AWS Lambda functions [execute in a Linux environment](https://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html). If we were to just build our Lambda on a Mac and upload it, it wouldn't run because that Swift executable wasn't made to run on Linux. We use Docker to spin up a Linux environment where we build our Swift executable to target Linux, and therefore will run in AWS Lambda. The more I learn about Docker, the more useful I find it!

- Domain name hosting (such as [GoDaddy](https://www.godaddy.com)) — If you want a custom URL like what I have for this blog or the example Swift website, you will need to buy it. I use GoDaddy, but there are others out there including an AWS offering called [Amazon Route 53](https://aws.amazon.com/route53/). If I was starting from scratch, I would probably go with Route 53, but I have been using GoDaddy for a while now and I am used to using it.

---

# Generating HTML/CSS using Swift
The below source code [is available on Github](https://github.com/jasonzurita/Swift-AWS-Lambda-Website).

## Getting the project setup

- Open up your terminal.
- Make and switch to a directory for your Swift website project (the desktop is used as an example below).
  + `mkdir ~/Desktop/Swift-AWS-Lambda-Website`
  + `cd !$`
- Start a new Swift project.
  + `swift package init`
  + This will give you a new Swift project.

    ![New Swift Package Folder Structure](/assets/images/posts/swift_aws_lambda_website/swift-package-init.png)

  + Feel free to delete the `Tests` directory and everything inside of the `Swift-AWS-Lambda-Website` directory. We don't need them for this project.
- Open your _Package.swift_ for editing.
  + I use _vim_, but you can use any text editor of your choosing. If you want a little more insight into why I use _vim_, I break it down in [this blog post](/try-vim-even-in-xcode).
- Edit the _Package.swift_ file to match the following:
  + Check out the resources under the [SPM section above](#spm-section) to learn more about the format of this file.

```swift
// swift-tools-version:4.2
// 1

import PackageDescription

let package = Package(
    name: "Swift-AWS-Lambda-Website",
    // 2
    products: [
        .executable(name: "Swift-AWS-Lambda-Website", targets: ["Swift-AWS-Lambda-Website"]),
        .executable(name: "Local-Website", targets: ["Local-Website"]),
    ],
    // 3
    dependencies: [
        .package(url: "https://github.com/tonisuter/aws-lambda-swift.git", .branch("master")),
        .package(url: "https://github.com/pointfreeco/swift-html.git", .exact("0.2.1")),
        .package(url: "https://github.com/pointfreeco/swift-web.git", .revision("2c3d440")),
    ],
    // 4
    targets: [
        // 4a
        .target(
            name: "GenerateWebsite",
            // Note:
            // Html below comes from swift-html above & Css and HtmlCssSupport come from swift-web above
            dependencies: ["Html", "Css", "HtmlCssSupport"]),
        // 4b
        .target(
            name: "Local-Website",
            dependencies: ["GenerateWebsite"]),
        // 4c
        .target(
            name: "Swift-AWS-Lambda-Website",
            dependencies: ["AWSLambdaSwift", "GenerateWebsite"]),
    ]
)
```
  1. The comment `swift-tools-version` declares the minimum version of Swift required to build this package.
  1. _Products_ allow you to explicitly define the artifacts from your Swift project. This is where you define what your project produces such as a _libraries_ or _executables_. In this case, we are making a product to generate an executable to upload as the Lambda function and a product to generate an _index.html_ file that we can open and use for local development.
     + Explicit products are a more recent addition to the Swift Package manager. To read up on their addition, check out the Swift Evolution proposal: [Package Manager Product Definitions](https://github.com/apple/swift-evolution/blob/master/proposals/0146-package-manager-product-definitions.md).
  1. This is where you can define other Swift library projects as dependencies to use in your project. The _URL_ needs to resolve to a git repo that has a `Package.swift` in its root directory. You can also specify the version of the dependency that you want to use.
  1. Here we define the different targets that this Swift project produces. Targets may be used to produce products, as noted above, or as internal dependencies.
     + a. This target is an internal dependency that is the _heart_ of this project. This is where we write our website. The reason this is broken out into a separate target is that we wanted to be able to use this twice: once for making the AWS Lambda executable, and once to write the generated HTML/CSS to a file for local development.
     + b. This target will be used for local development by making a local `index.html` file so we can simplify working on the website. That is, we don't need to upload a compilied Lambda function to AWS Lambda each time we want to see what a change does to the website!
     + c. This target is used to make the Swift Lambda executable, which will be uploaded to AWS Lambda to handle incoming requests to produce the website.

## The website

We have a _Package.swift_, but we still need to implement the three different targets. Let's do that! Starting with the internal dependency — the _GenerateWebsite_ target.

**Do the following:**
- Change to the sources directory.
  + `cd ~/Desktop/Swift-AWS-Lambda-Website/Sources`
- Make and switch to a new source directory.
  + `mkdir GenerateWebsite`
  + `cd !$`
- Make a file named _GenerateWebsite.swift_ with the below code.
  + Copy the below code, then
  + `pbpaste > GenerateWebsite.swift`

**Note: if you haven't already, check out the website in action: [https://swift-aws-lambda-website.jasonzurita.com](https://swift-aws-lambda-website.jasonzurita.com)**

```swift
// 1
import Foundation
import Html
import Css
import HtmlCssSupport
import Prelude

// 2
public func zIndex(_ index: Int) -> Stylesheet {
  return key("z-index", "\(index)")
}

// 3
public func generateWebsite() -> String {

    // 4
    let bodyStyle = height("100%")
                        <> margin(all: 0)
                        <> backgroundColor(.rgb(0xCC, 0xCC, 0xCC))

    let flexContainer = height("100%")
                        <> margin(all: 0)
                        <> padding(all: 0)
                        <> display(.flex)
                        <> align(items: .center)
                        <> fontFamily(["Arial", "Helvetica", "sans-serif"])
                        <> justify(content: .center)
                        <> fontSize(.rem(1.5))
                        <> color(.white)

    let imageStyle = position(.absolute)
                        <> top(0)
                        <> left(0)
                        <> right(0)
                        <> bottom(0)
                        <> margin(all: .auto)
                        <> zIndex(-1)

    let centerRow = width(.auto)

    let item = height(.auto)
                   <> width(.auto)
                   <> textAlign(.center)
                   <> color(.rgb(0x33, 0x33, 0x33))

    let hrStyle = position(.relative)
                      <> width("40%")

    // 5
    let document = html([
        body([style(bodyStyle)], [
            // 5a
            img([src("{logo URL omitted for readability}"), alt(""), style(imageStyle)]),
            div([style(flexContainer)], [
                div([style(centerRow)], [
                    div([style(item)], [
                        h1(["Welcome!"]),
                        h3(["A demo website written in Swift & hosted using AWS Lambda"]),
                        hr([style(hrStyle)]),
                    ]),
                    div([style(item)], [
                        p([ "Check out the related ",
                            a([href("https://www.jasonzurita.com/websites-using-swift-and-aws-lambda/")], ["blog post."]),
                        ]),
                    ]),
                ]),
            ]),
        ]),
    ])

    // 6
    return render(document)
}

```
1. Here we import the dependencies that includes the HTML and CSS DSLs.
1. This free function isn't included in the CSS dependency yet, so we simply define it here for later use when placing the logo image behind the text on the website.
1. This public function _generateWebsite_ will return a _String_ that is HTML. This function will be called from our Lambda function and during local development.
1. I am not going to get into the specifics of the CSS here, but this CSS is used to style the website, much like other websites.
1. Same thing about the HTML DSL here, which is stored as a local constant named _document_. Just know that this uses the _Html_ library and the _HtmlCssSupport_ to connect the above CSS to the various HTML elements.
  + 5a. The logo image URL was omitted for clarity here, but you can use any image you want here.
1. Finally, all we need to do is take the _document_ created in step 5 above, call _render_ on it, and return that HTML string — and there is your  website 🎊!

## Local development

This is great, but what good is the ability to generate HTML/CSS if we don't have a way to view it? The following target breakdown, _Local-Website_, is for local development. That is, a way to speed up development of your website before deploying it to Lambda, which would take longer to see your changes. Having this will save time during development! This target is one of the two _products_ of our Swift project.

**Do the following:**
- Change to the sources directory.
  + `cd ~/Desktop/Swift-AWS-Lambda-Website/Sources`
- Make and switch to a new source directory.
  + `mkdir Local-Website`
  + `cd !$`
- Make a file named _main.swift_ with the below code.
  + Copy the below code, then
  + `pbpaste > main.swift`

```swift
// 1
import Foundation
// 2
import GenerateWebsite

// 3
let artifactsDirectory = FileManager.default.currentDirectoryPath + "/Artifacts"
let outputFilePath = artifactsDirectory + "/index.html"

// 4
let html = generateWebsite()

// 5
guard let fileHandle = FileHandle(forWritingAtPath: outputFilePath) else {
    try? FileManager.default.createDirectory(atPath: artifactsDirectory,
                                             withIntermediateDirectories: false,
                                             attributes: nil)
    try? html.write(toFile: outputFilePath, atomically: true, encoding: .utf8)
    exit(1)
}

if let data = html.data(using: .utf8) {
    fileHandle.write(data)
}
```

1. We need to use Foundation to write out to a file. The main thing to note here is that the imported _Foundation_ here is different depending on if this is run on macOS or Linux. Not an issue in this case, but is something to consider if you want to test on macOS but your production environment for your Swift project is on Linux.
1. We import our internal dependency which generates the HTML.
1. Get a path for where to put the generated `index.html` file.
1. Generate the HTML using the free function from our imported internal dependency.
1. The remaining lines simply overwrite or create the `index.html` file with the newly generated HTML.
  + The output file will be located in the directory named _Artifacts_, which is automatically created as part of creating the `index.html` file here.
  + If you want to learn more about writing to a file, check out my previous post called [Swifty File Reading and Writing](/swifty-file-reading-writing).

**To use this for local development:**
- Go to your root project directory. In this case:
  + `cd ~/Desktop/Swift-AWS-Lambda-Website`
- Run this in your terminal.
  + `swift run Local-Website`
  + Open the generated _index.html_ file in your web browser of choice, located in the _Artifacts_ directory.

**Note: If you try and run the above local flow, you will get an error because we defined the _Swift-AWS-Lambda-Website target_ in our _Package.swift_, but have not implemented it yet. Do the following step, and come back to running the local flow after.**

## Make the Lambda function to upload to AWS

Last but not least, let's breakdown the `Swift-AWS-Lambda-Website` target.

**Do the following:**
- Change to the auto-generated _Swift-AWS-Lambda-Website_ directory.
  + `cd ~/Desktop/Swift-AWS-Lambda-Website/Sources/Swift-AWS-Lambda-Website`
    + _this directory should have ready been created for you when initially running `swift package init`_
- If you haven't already, delete the auto-generated _Swift_AWS_Lambda_Website.swift_ to have a blank start.
  + rm _Swift_AWS_Lambda_Website.swift_
- Make a file named _main.swift_ with the below code.
  + Copy the below code, then
  + `pbpaste > main.swift`

```swift
// 1
import AWSLambdaSwift
import GenerateWebsite

// 2
struct Event: Codable { }

// 3
struct Result: Codable {
    let html: String
}

// 4
func handler(event: Event, context: Context) -> Result {
    return Result(html: generateWebsite())
}

// 5
let runtime = try Runtime()
runtime.registerLambda("handler", handlerFunction: handler)
try runtime.start()
```
1. Here we import both the library to interface with AWS Lambda and our internal dependency to generate the HTML/CSS using Swift.
1. The `AWSLambdaSwift` project nicely uses the Swift `Codable` protocol to pull out information when the lambda function gets called. In this case, we don't need any of that information, so we make an empty struct that conforms to `Codable`.
1. Similar to the `Event` struct, the `AWSLambdaSwift` project uses `Codable` as a vehicle for the response from the Lambda. The return from this Lambda is a struct with one property — a string that is HTML. We will pull the HTML out for the final response in API Gateway. More on that later in Part 2.
1. This is the Swift function that gets registered as the AWS Lambda handler. The implementation of this function is simple — call the `generateWebsite` free function from the `GenerateWebsite` module, and return the generated HTML.
1. Finally, we create the runtime, register the handler with the runtime, and start the runtime.
   + Note: the handler in AWS Lambda will need to to be set to `Swift-AWS-Lambda-Website.handler`. More on this in Part 2.

We can run this product like the local development product, but as mentioned before the executable will not be able to run in AWS Lambda since it will be targeting macOS. The executable needs to be built targeting Linux. This is where _Make_ and _Docker_ come in.

**Do the following:**
- Change to the root project directory.
  + `cd ~/Desktop/Swift-AWS-Lambda-Website`
- Create a _Makefile_ with the below code.
  + Copy the below code, then
  + `pbpaste > Makefile`

_Note: you will need Make and Docker installed_

```make
# 1
PROJECT=Swift-AWS-Lambda-Website 

# AWS Lambda needs a handler executable to be:
# - Built to run on Linux (Docker is used for this!)
# - Zipped up

# 2
build_lambda:
# 3
	docker run \
			--rm \
			--volume "$(shell pwd)/:/src/$(PROJECT)" \
			--workdir "/src/$(PROJECT)" \
			swift \
			swift build --product $(PROJECT)
# 4
	mkdir -p Artifacts
# 5
	zip -r -j Artifacts/lambda.zip $(shell pwd)/.build/debug/$(PROJECT)
```

1. This is a constant that is the name of the product that will be run to make the AWS Lambda executable. This can be changed if your product's name is different.
1. This is the command name to create our Lambda function.
1. As mentioned, we use Docker to build the Lambda function while targeting Linux.
  + `--rm`, automatically remove the container when running finishes
  + `--volume`, mount the project on your computer to a location in the Docker container, so Docker knows what source files to build
  + `--workdir`, working directory inside the container
  + `swift`, use the latest Swift Linux Docker image
  + Finally, run `swift build` using the project name defined in step 1.
1. Make sure the _Artifacts_ directory exists.
1. AWS Lambda requires that we zip up the executable for upload, so that is what we do here. The output of this step, named _lambda.zip_, is placed into the same _Artifacts_ directory as used above for local development.

**To generate the Lambda function zip:**
- Go to your root project directory. In this case:
  + `cd ~/Desktop/Swift-AWS-Lambda-Website`
- Run this in your terminal.
  + `make build_lambda`
    + or, simply `make`
- The output zip will be located in the _Artifacts_ directory!

---

**Now, we are all set up to make this website live 🎉! In Part 2, we will work through just that — deployment & hosting using AWS! This will include some notes about pricing, setting up the Swift runtime using AWS Lambda Layers, the Lambda function, and creating a custom URL using API Gateway and Certificate Manager.**

---

# Summary

**Note: While waiting for Part 2, try experimenting with making your own website using local development!**

We accomplished a lot!!
- We touched on the projects that help us create our website using Swift.
- We setup fast iterative local website development.
- We can generate the zipped up website executable that we will use to upload to AWS Lambda. To do this, we leveraged Docker and Make to simplifying building against a Linux environment.

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
