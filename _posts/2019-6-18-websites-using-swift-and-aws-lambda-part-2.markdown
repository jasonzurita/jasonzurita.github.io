---
title: "Websites using Swift and AWS Lambda — Part 2"
layout: post
date: 2019-6-18
image: /assets/images/posts/swift_aws_lambda_website_part_2/swift-aws-lambda-logos.jpg
headerImage: true
tag:
- Swift
- AWS
- AWS API Gateway
- AWS Lambda
- AWS Layers
star: false 
category: blog
author: jasonzurita 
description: Websites using Swift and AWS Lambda — Part 2 — Deploying & hosting the website using AWS
---

**If you haven't already, check out [Part 1](/websites-using-swift-and-aws-lambda).**

# Summary
<sub>Want to see this kind of Swift website in action!? Check out [the example website](https://swift-aws-lambda-website.jasonzurita.com), and here is the [source code](https://github.com/jasonzurita/Swift-AWS-Lambda-Website)!</sub>


This is the second part in a two part series of posts about making websites using the Swift programming language and Amazon Web Services (AWS). The first part focused more on the technical details of what this even means and how to set up a Swift project to generate a Swift website: both for local development and for making an executable ready for AWS Lambda. This post will pick up from there with a few notes about pricing before diving into the details on how to deploy & host this kind of website!

Even if you didn't look at the first part (or skimmed it 🙃), I tried to provide enough details to make this a useful overview of AWS.

_Yes, you will need an AWS account to do the following, and if you are worried about cost —  me too! Check out the section on [pricing below](#pricing-section)_.

---

# Hosting using Amazon Web Services (AWS)
As mentioned, this project makes use of AWS to provide the infrastructure to deliver a Swift generated website. To accomplish this from a high level, API Gateway provides us with a URL to put in a web browser that triggers an AWS Lambda function, which runs our _generate website_ Swift code (that we made in [Part 1](/websites-using-swift-and-aws-lambda)), to make HTML/CSS that is then returned and rendered as a website 🎉.

Before we start, a couple things worth noting:
- This post uses AWS, but there are other services out there to check out like [Vapor](https://vapor.codes), [IBM Kitura](https://www.kitura.io), and [OpenWhisk](https://openwhisk.apache.org).
- The dashboard for all Amazon Web Services, [the AWS console](http://aws.amazon.com), is notorious for having a steep learning curve. As we dive into the different services, I will to do my best to explain each step so that we don't get lost in all the different AWS buttons and levers 😶.
  + The easiest way to find a service that you are interested in (e.g., Lambda & API Gateway) is to type the name into the search bar at the top. For now, forget about the rest of the ways to navigate around. Here are two examples of what the search bar looks like on different pages:
<br>
![aws_search_bar_1](/assets/images/posts/swift_aws_lambda_website_part_2/aws_nav_1.png)
<br>
![aws_search_bar_2](/assets/images/posts/swift_aws_lambda_website_part_2/aws_nav_2.png)
<br>

{: #pricing-section}
## Pricing
You will need an AWS account for this post. Yes AWS is not free, but I wouldn't get caught up thinking about cost too much unless you think you will have _millions_ of requests coming in. Even at that point, you won't be paying much for the services used in this post. Let's take a look!

### Personal experience
Part 1 of this post ended up getting a decent amount of exposure having been picked up by some weekly newsletters ([iOS Dev Weekly issue 394](https://iosdevweekly.com/issues/394) and [Swift Weekly issue 150](https://swiftweekly.com)), and in turn so did the [example website](https://swift-aws-lambda-website.jasonzurita.com). With that flood of traffic, my AWS bill was **at one point showing a ~5000% increase in my monthly cost, totaling a whole $0** 😂. Here is a screenshot prior to hitting that ~5k% mark.
<br>
![aws_cost_explorer](/assets/images/posts/swift_aws_lambda_website_part_2/cost_explorer.jpg)
<br><br>

### Some math
Not fully convinced? Let's do some back of the envelope calculations to further prove the point:
- [AWS Lambda](https://aws.amazon.com/lambda/)
  + Compute charges — AWS Lambda has a free tier of 400,000 GB-s per month. If you back calculate from this, you can have over 3 million executions for free!! This calculation assumes 1 second per function run with under 128MB memory used. Each execution of the example website fits comfortably in these constraints, but even if it didn't I think I can spare some of the 3 million requests assumed to make up for the higher runtime/memory usage 😝.
  + Request charges — Also, assuming the 3 million requests above, AWS charges $0.20 per million after 1 million free requests. So, you would have to pay $0.40!
  + Here is a neat little [price calculator](https://s3.amazonaws.com/lambda-tools/pricing-calculator.html#) too!
- [API Gateway](https://aws.amazon.com/api-gateway/)
  + Pricing for API Gateway is pretty simple — $3.50 per million calls. Hopefully if you are experiencing millions of requests per month, you can afford the cost 🙃.
- [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/)
  + Public certificates are free.

Fair to say, running your website like this is effectively free! Costs of course may vary from the time of this writing, but the end conclusion will be likely similar.

**Additional things to note**
- AWS offers a great first year [free tier](https://aws.amazon.com/free/) with even more free offerings. If you are still hesitant, create a new account and cancel after the first year.
- As with most things, once you create an account with AWS, make sure you keep your credentials private. If you create an ssh key for AWS, you wouldn't want to accidentally expose that publicly and potentially have people running up your bill 😅.

{: #swift-runtime-section}
## Make a Swift runtime for AWS Lambda
How does Swift code run in an AWS Lambda function without being one of the _out of the box_ supported languages (e.g., python, javascript)!? We will need to make an environment that will allow our Swift code to run in AWS Lambda.

To accomplish this, we'll use a newer addition to AWS called [AWS Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) to create a Swift runtime for our AWS Lambda function to run Swift code. AWS Layers can do more than make runtimes for different languages and is worth checking out to see how powerful AWS Lambda is with this addition!

To make the runtime, clone [this project and follow the readme](https://github.com/tonisuter/aws-lambda-swift) to get a Swift runtime uploaded to AWS. The above project is also the same project that we use to let our Swift code communicate with Lambda function in the first part of these posts!

{: #swift-runtime-section-ARN-subsection}
After creating the Layer, copy the ARN! We will use it in the next section to reference this Layer from Lambda.
<br><br>
_Note: ARN stands for Amazon Resource Name, and is a string that uniquely identifies different AWS resources like the Layer you created here. If you want to learn more, [read up on ARNs](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)._

---

As I noted above, I used [this project](https://github.com/tonisuter/aws-lambda-swift) to create the Swift runtime for AWS Lambda. There are other similar projects that are worth checking out too. The below references come from this [Swift forum's post](https://forums.swift.org/t/aws-lambda-runtime-api/18498/1). 
- [Swift Custom Runtime Lambda](https://github.com/sebsto/swift-custom-runtime-lambda)
- [AWS Lambda Swift Runtime](https://github.com/giginet/aws-lambda-swift-runtime) — I am not sure about the level of maturity for this project.
- [Smoke Framework](https://github.com/amzn/smoke-framework) — This project hopes to add AWS Lambda support in the future.


## The AWS Lambda function
With the Swift runtime Layer ready, we can now set up our Lambda function with our Swift code to generate HTML/CSS for our website!
<br>
<sub>Note: The below goes through the manual way to create and configure a Lambda function. There are other ways like using the [AWS commandline interface (CLI)](https://aws.amazon.com/cli/).</sub>

1. In the AWS search type in, and select, `Lambda`.
   ![aws_lambda_creating_the_lambda_1](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/1.png)
<br><br>
1. Next, click to create a new Lambda function
<br>
   ![aws_lambda_creating_the_lambda_2](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/2.png)
<br><br>
1. You will be presented with this:
<br>
  ![aws_lambda_creating_the_lambda_3](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/3.png)
<br>
  + Give your Lambda a `Name`.
  + Choose to use a custom `Runtime` (we will be using the Swift runtime created in the previous section — more on this later).
  + The next two options are to define a `Role`, which is used to define permissions and security that you want to grant your Lambda function. I chose to go with an existing basic execution role. If you want to learn more about _roles_, [check here](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html).
  + Click to create your function!
<br><br>
1. When your newly created Lambda function opens, there will be an intimidating amount of things to sort through (at least there was for me). Working form top-right to top-left, some notable things.
<br>
   ![aws_lambda_top_menu_bar](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/4.png)
<br>
  + When you make changes to your Lambda function, you will need to press _Save_ for the changes to take effect. Think of this like updating your current git commit, git amend, but these changes are live!
  + You can test your Lambda by pressing _Test_ on the top right. This won't do anything meaningful right now since we haven't added any code to run/test yet.
  + The _Actions_ dropdown includes the ability to version your Lambda function. Think of this like a combination of a git commit and deployment. This lets you lock in a version of your Lambda function, which you can use later to roll back to a previous version of your function if needed. I recommend you use this in case you need to go back to a previous working version of your Lambda function, or if someone overrides your Lambda function accidentally 😅.
  + The _Qualifiers_ dropdown is where you manage your Lambda function versions.
<br><br>
1. Next, we need to link the Swift runtime that we [created above](#swift-runtime-section) to this newly created Lambda function. Find and click on the _Layers_ option.
<br>
   ![aws_lambda_how_to_add_a_layer_5](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/5.png)
<br>
   ![aws_lambda_how_to_add_a_layer_5](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/6.png)
<br>
  + Here you will be able to add the runtime Layer. You can either select one from a list or provide an ARN. I've had little luck with selecting from a list, so I usually default to providing the ARN. If you are using the ARN, simply use the ARN for the AWS Layer you created in the [previous section](#swift-runtime-section-ARN-subsection) and click _Add_.
<br><br>
1. Now that we have the Swift runtime setup, our Swift executable will be able to run! We just need to upload the Swift executable that we created in [Part 1](/websites-using-swift-and-aws-lambda). If you want to skip creating the executable, you can download an already compiled one from the Github repo for these posts: [example Swift website generator executable (lambda.zip)](https://github.com/jasonzurita/Swift-AWS-Lambda-Website/tree/master/Artifacts).
<br>
   ![aws_lambda_upload_executable_1](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/7.png)
<br>
   + Make sure you have your Lambda function's name selected, which is right above the _Layers button_ that we previously selected to add the Swift AWS Layer runtime.
   + Scroll down to the _Function code_ section and look for _Code entry type_.
   + Click the dropdown menu and select `Upload a .zip file`.
   + Below the dropdown menu is the upload button. Click this to upload the Swift executable.
   + Make sure the _Runtime_ is set to use a custom runtime.
   + Update the _Handler_ to be `Swift-AWS-Lambda-Website.handler`
     + This tells Lambda what Swift function to run when triggered. This just has to match the function name that we created in the first part of these posts.
   + Press _Save_, which should be at the top right of the screen.
<br><br>
1. We are ready to test!!
Click the _Test_ button at the top right of the screen (next to the _Save_ button).
<br>
   ![aws_lambda_test_it_1](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/8.png)
<br>
   + The first time you press this button, you will be prompted to configure the test event. You can edit the payload that will be delivered to your Lambda function. In our case this doesn't make much of a difference, but if your Swift website needs to take in some information like a _customer id_ to pull some data out of a database, you would probably want to make a test that provides a test customer id. In this example, you may also want a test an invalid _customer id_ to test some edge cases.
   + Since our use case is simple, choose whatever names you want to give the test, feel free to delete the json between the brackets, and press _Save_.
   + Press the _Test_ button again and you should see a green box with the result of running the Swift generate website code — HTML/CSS 🎉!
   ![aws_lambda_test_it_2](/assets/images/posts/swift_aws_lambda_website_part_2/lambda/9.png)
     + If you wanted to, you could copy the generated HTML/CSS to an `.html` file to see the website in action that was created by running your newly created Lambda function!
   + If your test runs and a red error box appears, don't panic! Read the error message and step through everything you did to troubleshoot. Also, feel free to reach out if you are really stuck :).

## AWS API Gateway
Now that we have a Lambda function that is ready to generate our HTML/CSS, we need a way to run it other than hitting the _Test_ button 🙃. API Gateway allows us to map an incoming URL request to trigger our Lambda function; The result of which then gets sent back and we have a website!

1. Search for and go to API Gateway from the AWS console.
<br><br>
1. Create a new API by clicking _Create API_ at the top.
<br><br>
1. Most of the defaults are fine. Just give your new API a name (and a description if you want).
<br>
   ![new_api_gateway_screen_1](/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/1.png)
<br><br>
1. With the API initially created, we need to configure it to return the generated HTML/CSS from our Lambda function. First step is to set up the method for your endpoint. In this case, we want a _GET_ method.
<br>
   ![get_resource_1](/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/2.png)
<br>
  + With your API selected on the left, select the _Actions_ dropdown.
  + Click on _Create Method_.
<br>
   ![get_resource_2](/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/3.png)
<br>
  + Click the dropdown to select the type of method. Click _GET_ from the options.
  + Accept the _GET_ method choice by clicking the little check button.
<br><br>
1. Next you will be asked to choose the integration point, which is similar to saying _what would you like to do when your URL gets called?_ If you don't see the below screen, click on the _GET_ request method that you just created to bring it up.
<br>
   ![configure_get_request_1](/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/4.png)
<br>
  + Choose _Lambda Function_ at the top.
  + Give the name of your Lambda function at the bottom. When typing in this field, you should get auto complete to help you select your Lambda function.
  + Press save.
  + You will be asked to add permission to the Lambda function. Press _ok_.
<br>{: #get-method-selected}<br>
1. Now that we have an API with a _GET_ method that is hooked up to call our Lambda function, we need to configure the _GET_ method a bit. If you don't have your _GET_ method selected, click it now.
   ![configure_get_request_2](/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/5.png)
<br>
   + Aside: The right side details that this opens up can be a little intimidating. From a high level, this shows the flow for a request coming in and the response going out. The left most rectangle shows the client and gives you the option to test your endpoint (we will be using this later). The top two boxes allow you to configure authentication and then where the request gets routed to, which ultimately ends up at the right most rectangle — at our Lambda function! Following the bottom two boxes, the response from our Lambda function gets processed and sent out back to the client (client = the thing calling your endpoint).
<br>
   + Let's start configuring the method by clicking on _Method Response_.
<br>
   ![configure_get_request_3](/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/6.png)
<br>
     + click on the dropdown under _HTTP Status_. Change the following settings:
       + Under _Response Headers for 200_ add header with name _Content-Type_
       + Delete what is under _Response Body for 200_
   + Return to _Method Execution_.
     + If you ever lose your place just reselect the _GET_ method as you did before to see the flow block diagram.
   + Select _Integration Response_.
<br>
   ![configure_get_request_4](/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/7.png)
<br>
     + Now, click to show the dropdown for both _Header Mappings_ and _Mapping Templates_.
     + First, we need to make sure that the respone that goes back has a content type of HTML. This tells the web browser to try and render the response as webpage.
     + Edit the _Header Mappings_ as follows:
       + _Response header_ -> `Content-Type`
       + _Mapping value_ -> `'text/html'`
     + Next, we need to edit the _Mapping Templates_ to pull the HTML/CSS out form our Lambda function's response and return that.
       + Under _Mappings Templates_ delete _application/json_, if that is listed.
       + Click _Add mapping template_ and add _text/html_
       + On the right, put this into the text box:
         + `$input.path('$').html`
         + Note: If you don't see a text box to the right, you will need to click on the _text/html_ that you just added in the previous step.
     + Press save at the bottom.
<br><br>
1. With your endpoint fully set up, let's test it!
   + Start by going back to the overview flow block diagram of your GET endpoint. The overview [looks like this](#get-method-selected).
   + Click  _Test_ on the left side of the flow block diagram.
   + On this screen click _Test_.
   + You should see something like this (you may need to scroll down a bit):
<br>
   <img src="/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/8.png" alt="configure_get_request_5" width="500"/>
<br><br>
1. We are real close now 🤓! With the above API Gateway setup complete and tested, we just need to make this endpoint publicly available. Go back to your API and click the dropdown for _Actions_ like you did before.
   ![deploy_api_1](/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/d1.png)
   + Under _API Actions_ select _Deploy API_.
   <img src="/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/d2.png" alt="deploy_api_2" width="500"/>
   + You will need to select a stage like production or development. Since we haven't set up a stage yet, let's make new one and call it _prod_ for production and click _Deploy_.
   + When this completes, you will have a fully functional AND publicly available URL!
   <img src="/assets/images/posts/swift_aws_lambda_website_part_2/api_gateway/d3.png" alt="deploy_api_3" width="500"/>
   + The URL will not be pretty, but you can copy and paste it into a web browser to see you webpage in action 🎉😎🎉!

## Custom URL

Even though we just made a publicly available URL, having a website with a custom URL is a nice professional and gratifying touch.

_The following custom URL steps use GoDaddy because that is where I started buying domains, but where you buy them is not really that important. As noted in the previous part of these posts, if I was going to buy a new domain(s) I would consider going with [Amazon Route 53](https://aws.amazon.com/route53/) to have everything under the AWS umbrella._

1. To start, we need to make a certificate to both verify that we own the domain and to give us a secure website (i.e., _https_). After the certificate is created, we will use this in API Gateway to create a _custom domain name_ that maps to the API we created in the last section.
   + In the AWS console, search for and go to _Certificate Manager_.
   + Click on _Request a certificate_.<br>
     <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/certificate_manager/1.png" alt="making_a_certificate_1" width="500"/>
     + Select _Request a public certificate_ and click _Request a certificate_ at the bottom.
   + Next, you will be asked to put in the domain name that you want.
     <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/certificate_manager/2.png" alt="making_a_certificate_2" width="500"/>
     + I choose to go with a subdomain because I already use _jasonzurita.com_ for this site — the subdomain used for the acompanying [example website](https://swift-aws-lambda-website.jasonzurita.com) is the _swift-aws-website_ before the main domain name. You should choose the domain, or subdomain, that is right for you.
   + Click _Next_.
   + Here you will need to select a validation method. Choose _DNS validation_ and click on _Review_.
     <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/certificate_manager/3.png" alt="making_a_certificate_3" width="500"/>
   + Make sure everything you put in looks correct and click on _Confirm and request_ at the bottom.
     <!-- Skipping image number 4 since it doesn't offer much. -->
   + On the main Certificate manager dashboard, you will see that your certificate is created and _Pending validation_.<br>
    <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/certificate_manager/5.png" alt="making_a_certificate_5" width="500"/>
      + We need to _validate_ the certificate to prove that we own the domain.
<br>{: #add-cname-to-dns}<br>
1. In order to _validate_ your certificate, we need to add the _CNAME_ that AWS generated to your DNS configuration, GoDaddy in my situation.
   + _Note: CNAME record (Canonical Name record) is using in DNS to map one domain name to another. In our case, a map from our custom domain to the API gateway domain we made._
   + To see the CNAME click on the certificate you just created and click on the dropdown in the _Domain_ section.
   + You should now see _Name_, _Type_, and _Value_, where _Type_ is CNAME.
   + Keeping the above information available, go to your DNS provider's website (GoDaddy in my case) and log in.
   + Navigate to your custom domain's DNS management section and add a CNAME entry using the Name and Value above.<br>
     <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/dns_setup/1.png" alt="custom_domain_dns_setup_1" width="600"/>
     <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/dns_setup/2.png" alt="custom_domain_dns_setup_2" width="600"/>
       + _Name_ goes into the _Host_ field and _value_ goes into the _Value_ field.
         + Note: For GoDaddy, when you enter the _Host_, make sure to leave off the root domain when doing so. I am not sure if other DNS providers do this, but GoDaddy implicitly assumes the root domain at the end of what you enter. Continuing on this tangent, if you ever want to enter the root domain, GoDaddy let's you put in a _@_ to represent that. This was a subtle _got ya_ that caused me some pain.
   + Press _Continue_.
   + After you do this, go back to AWS Certificate Manager and { refresh ? }, and you should see your certificate turn to say _Issued_!<br>
   <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/certificate_issued.png" alt="custom_domain_certificate_issued" width="650"/>
     + If certificate validation doesn't happen right away, you may need to give the newly added CNAME some time to propagate.
<br><br>
1. Now we are ready to use our newly created certificate. { this whole section needs to be verified }
   + Search for and go to API Gateway.
   + Select _Custom Domains_ from the left side bar.
   + Click on _Create Custom Domain Name_.
     <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/api_gateway/1.png" alt="custom_domain_api_gateway_1" width="500"/>
       + Put in your domain name.
       + You can select either _Edge Optimized_ or _Regional_. Select _Regional_ so that your custom domain gets deployed quickly.
         + _Edge Optimized_ will produce a custom domain that is faster to access for your end users. It is good practice to select this if you are deploying a production website.
       + Select your newly made certificate.
       + Click _Save_.
         + Your custom domain setup may take some time to process.
   + Once your custom domain has been set up, we need to add a mapping to trigger the right API path. This is simple in our case since we only have one path, but you can imagine that there could be other paths for our API like a customer screen (e.g., `<domain_name>/products`).
     <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/api_gateway/2.png" alt="custom_domain_api_gateway_2" width="500"/>
       + Set _Path_ to "`/`".
       + Set _Destination_ to our previously created API.
       + And, set the last field to `Prod`.
   + Click _Save_. Your _custom domain_ should look like this after being setup:
     <img src="/assets/images/posts/swift_aws_lambda_website_part_2/custom_domain/api_gateway/3.png" alt="custom_domain_api_gateway_3" width="500"/>
<br><br>
1. The final thing we need to do is add one final CNAME to our DNS to map the custom domain we want to the _custom domain name_ that we just set up!

   _Note: After setting this up, when a user types in your custom domain into the browser, they will get mapped to the custom domain in the previous step. When API Gateway receives the request, AWS will cross check against the legitimacy of the custom domain name using the certificate we created. Then API Gateway will know where to route the request — our Lambda function!_

   + Using the following, complete the same steps to [add a CNAME to your DNS](#add-cname-to-dns) as before. { check this link }
     + For the _Host_ field, enter the custom domain name you want. In my case it was _swift-aws-lambda-website_. If you want to use your root domain (e.g., jasonzurita.com), GoDaddy uses _@_ to represent the root domain.
     + For the _Value_ field, enter the API Gateway Custom Domain Name's _Target Domain Name_ as seen in the previous section's last screenshot.
   + Once the CNAME has been added and processed, when you type in your custom domain into the browser you will see your website that is generated from Swift 🎊 🎉 🍾!!
   
---

# Some parting thoughts
The current state of this implementation is a great first step! Some additional thoughts come to mind:
- The example website here is a single, simple, page. Pushing this to see where the trade-offs of this kind of website would be an interesting learning exercise. Given time, I may try to re-implement my blog using this type of tech stack. Here are a few suggestions to begin pushing exploring the true validity of a website like this:
  + Querying a database while generating the website.
  + Incorporating a content management system (CMS). Probably headless like [Forestry](https://forestry.io) and [Contentful](https://www.contentful.com).
  + Incorporating HTTPS requests while generating the website.
  + Multiple page routing. Maybe routing using API Gateway with separate Lambdas or Lambda Layers for each page.
- For dynamic things like animations we will still need to use JavaScript. Figuring out a way to incorporate JavaScript would be helpful and probably not too difficult. Taking this a step further, exploring Web Assembly with this tech stack would be interesting.
- With the introduction of [SwiftUI](https://developer.apple.com/tutorials/swiftui/), I would be very interested to see how that can be leveraged to generate the HTML/CSS. Here are some initial projects that came out shortly after SwiftUI was announced along these lines:
  + [Vaux](https://github.com/dokun1/Vaux)
  + [Understanding SwiftUI by reimplementing it to render to HTML](https://github.com/zhuowei/marina)
- This could make a pretty neat open source project that includes things like a script to make a template website for getting started, and AWS command line integration to facilitate updating the API Gateway, Lambda function, etc. If someone is interested in this, I would be happy to help!


---

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
