---
title: "Try vim (even in Xcode!)"
layout: post
date: 2018-12-14
image: /assets/images/posts/try_vim/vim_xcode_logos.jpg
headerImage: true
tag:
- Vim
- Xcode
star: false 
category: blog
author: jasonzurita 
description: Try vim (even in Xcode!)
---

# Summary
A co-worker told me a while back that I should learn _vim_ or _emacs_. When asked why, he noted that learning one of them would not only help me edit text files faster, which means editing code faster, but learning one would help me become a more adaptable developer by reducing my reliance on IDEs. He suggested that I start writing notes using one of them to get started and to see what I thought. I chose to try out _vim_, and although I still write my notes like he suggested, _vim_ has turned into one of my most useful and rewarding tools (I am sure _emacs_ would have provided me with similar benefits). Some reasons why:
- Faster editing of text files, which means I can code faster! The more time you put into learning and customizing _vim_ the more efficient you will get at editing text and code.
- When learning new languages or diving into new projects, you continue to feel comfortable with editing and navigating that code since you continue using your already familiar _vim_ setup — no need for a new IDE or another text editor every time you change contexts (unless that platform strong arms you into using their IDE like iOS development and Xcode...). Ultimately, you end up worrying less about an IDE and better focus on learning what you need to be successful.
  + Beyond my iOS/macOS development, I have been venturing into all sorts of different projects ranging from this blog, Swift scripting, backend development in Python, some ReasonML script work, frontend React/Redux development, and AWS Lambda tinkering (and I am sure more to come). At each turn, I don't have to worry too much about my editing experience. I add some syntax highlighting and function tags to my _vim_ setup, and I am off! Even though I switch languages and projects, _vim_ helps remove one hurdle by always remaining constant. A similar thing can be said about architectures like Rx. If you learn Rx, you can more easily get up to speed on different projects that use Rx since the fundamental concepts are the same no matter what platform you are on.
- Maybe this is specific to me, but when I use _vim_ I feel like one of those _cool hackers_ on tv that is clicking away making _awesome things_ happen in their terminal. Because of this, as I continue to get better at using _vim_, I feel like more of a developer. You absolutely don't need to know _vim_ to be a great developer, but this mental trick helps me overcome some of my impostor syndrome.
- You learn more about how your system works.
  + _vim_ goes back a long way. So much so that most systems have _vim_, or _vi_, ready to be used. This is great if you are ssh-ing into a server and need to do a quick file edit.
  + As you discover all the power that _vim_ offers, you begin to see many different parts of your system like how program processes work, stdin/stdout, dotfiles, how to use Unix programs, and the list goes on.
- Learning a new technology helps in ways that you can't predict. When I first started using _vim_, I never thought that I would:
  + be able to port my _vim_ editing skills to Xcode. Turns out there is a great open source project called XVim2 to get _vim_ key bindings in Xcode — more on that later!
  + use _vim_ for all of my text/code editing needs.
  + enjoy the community of _vim_ developers / users. Learning from them has helped me grow from a totally different perspective since _vim_ transcends programming language and project.
  + find so much joy in using a text editor.

---

## Tips for getting started with vim
- Just like my friend suggested, try starting by taking notes in _vim_.
  + To start a _vim_ session, simply open up terminal and type `vim`.
- How _vim_ works (a great simplification):
  + There are two modes.
    + Normal mode — where you move around and execute _vim_ commands. You start in this mode when you enter _vim_. You can enter commands in _normal mode_ by pressing `:` followed by the command. Examples of commands are: find and replace text, searching like using grep, executing shell commands, opening quick fix menu, and using plugins.
    + Insert mode — where you can enter text. You enter this mode by pressing `i` when in normal mode. To enter _normal mode_ from _insert mode_ you press `esc`.
  + To exit _vim_ go to _normal mode_ and press:
    + `: + q` — quit if no changes were made to the file.
    + `: + wq` — quit and save changes. 
    + `: + q!` — quit without saving changes (force quitting).
    + See this [Stack Overflow question](https://stackoverflow.com/questions/11828270/how-to-exit-the-vim-editor) for more info on exiting _vim_.
- Start off by doing the built in _vim_ tutorial.
  + From terminal enter `vimtutor`.
- A great resource for learning _vim_ over time: [Seven habits of effective text editing](https://www.moolenaar.net/habits.html).
- If you like video tutorials, [Vimcasts](http://vimcasts.org) are really great!
- Great reference book on _vim_, [Practical Vim](https://www.amazon.com/Practical-Vim-Edit-Speed-Thought/dp/1680501275).
- If you are interested, here is a nice [history of _vim_](https://twobithistory.org/2018/08/05/where-vim-came-from.html).
- As you dive further into _vim_, you will learn that you can customize _vim_ however you want. You do this by adding to a `~/.vimrc` file (which you can edit using _vim_ 🤯). I host my [_vimrc_ file on github](https://github.com/jasonzurita/dotfiles/blob/master/vimrc), so feel free to take a look. I try to comment that file well enough to help other people, including future me, to better understand what is in there.


## How to get vim keybindings in Xcode

- The below steps are for [XVim2](https://github.com/XVimProject/XVim2).
- Assumes Xcode 9+.
- The [documentation](https://github.com/XVimProject/XVim2/blob/master/README.md) for XVim2 is pretty good. The only thing that can be a bit confusing is that the steps are broken up into two documents: re-signing Xcode & installing _vim_ keybindings in Xcode. The below steps are simply a combination of the two with some personal experience written in to help. 
- You will need to re-sign Xcode with your own certificate 😅. You used to be able to install 3rd party plugins like this one directly into Xcode, but Apple has since removed this capability. This may seem a little crazy, but if you want to have _vim_ integration, this is what you need to do until Apple brings this kind of plugin functionality back 🤞.
  + Something to keep in mind: Your self-signed Xcode certificate will expire in a year, so you will need to create a new certificate after a year and re-sign Xcode again.
- Similar to having a _vimrc_ file to customize _vim_, you can customize _vim_ in Xcode by creating a `~/.xvimrc` file. Personally, I am okay not having this since you are using _vim_ in Xcode and not all your customizations will work the same. Xcode has a bunch of shortcuts that I use instead.
- If you have any issues, make sure to read through the docs for XVim2 on github (link above).

#### Step A: Re-sign Xcode
_If you have done this before, you can skip down to #7_.
1. Close Xcode.
1. Open _Keychain Access_ and select _login_ in the left pane.

   ![keychain_login](/assets/images/posts/try_vim/keychains_login.png)
1. From the top menu, select: `Keychain Access` -> `Certificate Assistant` -> `Create a Certificate...`.

   ![keychain_create_certificate](/assets/images/posts/try_vim/keychain_create_certificate.png)
1. Change _name_ to `XcodeSigner`.
1. Change _Certificate Type_ to `Code Signing`.

   ![create_certificate](/assets/images/posts/try_vim/create_certificate.png)
1. Press _Create_.
1. Assuming Xcode is in your _Application folder_, run this in terminal to re-sign Xcode:
   + `sudo codesign -f -s XcodeSigner /Applications/Xcode.app`
1. Be patient as the code signing will take some time.

#### Step B: Integrate vim into Xcode
1. Either:
   + `git clone https://github.com/XVimProject/XVim2.git` or
   + If you have XVim2 cloned already, `cd` to the XVim2 repo and `git pull`.
1. Confirm _xcode-select_ points to your Xcode by running `xcode-select -p`.
  + You should see this output: _/Applications/Xcode.app/Contents/Developer_.
  + If this doesn't show your Xcode application path, use `xcode-select -s` to set it.
1. If you haven't already, `cd` to the _XVim2_ folder.
1. Run `make` in your terminal.
1. Optional: Create a _xvimrc_ to customize vim in Xcode -> `~/.xvimrc`.
   + One thing to note is that some things are a little difficult to get working such as _vim_ syntax highlighting since you are in the context of Xcode.
1. Launch Xcode.
1. When Xcode detects that there is an unexpected bundle _XVim2.xcplugin_, you want to _Load Bundle_.

   ![unexpected_plugin](/assets/images/posts/try_vim/unexpected_plugin.png)
1. Xcode will then ask you for keychain access, which you need to give it.
   + Be patient with the number of times Xcode prompts you for keychain access. This is just a quirk about Xcode.

   ![keychain_access](/assets/images/posts/try_vim/keychain_access.png)

1. Note: If you accidentally press _Skip Bundle_ or deny keychain access, close Xcode and enter this into terminal:
   + `defaults delete com.apple.dt.Xcode DVTPlugInManagerNonApplePlugIns-Xcode-X.X`.
     + Where _X.X_ is your Xcode version.
   
## More advanced tip
- If you have a code formatting tool that you like to use (e.g., Swift - [SwiftFormat](https://github.com/nicklockwood/SwiftFormat), Python - [Black](https://github.com/python/black)), you can easily format the code in your current buffer (the file you have open in vim) by doing:
  + `:%!SwiftFormat`
  + Here is a [tweet showing this in action using SwiftFormat](https://twitter.com/jasonalexzurita/status/1071048197550813184)

---

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
