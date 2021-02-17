---
title: "Remove Xcode's File Header Comments"
layout: post
date: 2020-5-12
image: /assets/images/posts/remove-header-comments-logo.png
headerImage: true
tag:
- Xcode
- Swift
- Workflow
star: false 
category: blog
author: jasonzurita 
description: Remove Xcode's File Header Comments
---


# Summary
**[Skip to how to do it](#how-to-do-it)**

When I started iOS development, I thought the auto-generated comments at the top of all newly created files in Xcode was neat. Then, I went through a phase where I wanted something more custom. Now, I just delete them 🙃.

Why? The header comments don't add much to a source code file (unless there is an attribution or legal reason). Mostly, they just end up cluttering source files and force you to scroll down before starting to read the code.

Why automate this? I like the philosophy of applying automation or a time-saving process to something I find myself doing manually over and over again — Even if it saves only a few seconds each time, like in this case. Hey, that time adds up 😉.

The below steps uncover a bit of Xcode _dark magic_ to create a new custom file template with no auto-generated comments.


---

{: #how-to-do-it}
## Custom Template w/ No Header Comments 

- Create your custom template by copying an existing template<br>
  **Note: Change the name `Custom\ File` to whatever you want to name your template.**
  ```
ditto \
/Applications/Xcode.app/Contents/Developer/Library/Xcode/Templates/File\ Templates/Source/Swift\ File.xctemplate \
~/Library/Developer/Xcode/Templates/File\ Templates/Source/Custom\ File.xctemplate
```
  + A quick note about the files that were copied over for your new custom template
    + `___FILEBASENAME___.swift` — This is the new file template. We will edit this in a bit
    + `TemplateIcon.png` — Change this if you want a custom icon in Xcode for your new file template
    + `TemplateIcon@2x.png` — The higher resolution version of the above icon
    + `TemplateInfo.plist` — Metadata about the template. One thing to consider is the `SortOrder` key. Make the value's number lower and your new template will be ordered towards the beginning of the list of new file options
  + This uses macOS's [ditto](https://malcontentcomics.com/systemsboy/2005/12/command-line-smackdown-cp-vs-ditto.html) command to copy the Swift file template to a location where Xcode will look for templates on your system.

- Delete the auto-generated comment stub at the top of the base file
  + `cd ~/Library/Developer/Xcode/Templates/File\ Templates/Source/Custom\ File.xctemplate`
  + Open `___FILEBASENAME___.swift` in your favorite text editor (e.g., Vim)
  + Delete the `//___FILEHEADER___` comment from the top of that file
    + This line, if left, would get replaced with all that template information when a new file is created
  + [optional] Edit the imports to your liking
- **Done!** — Now, just choose your custom template in Xcode when creating a new file 🎉

---

If you want to learn more, check out these related resources:
- [Changing Xcode Header Comment](https://useyourloaf.com/blog/changing-xcode-header-comment/)
- [Create Xcode Templates](https://medium.com/overapp-ios/create-xcode-templates-c968d4b43f7b)

---

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
