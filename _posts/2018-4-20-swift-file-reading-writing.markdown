---
title: "Swifty File Reading and Writing"
layout: post
date: 2018-4-20
image: /assets/images/profile.jpg
headerImage: false
tag:
- iOS
- Swift
- File Read
- File Write
- Protocol oriented programming
star: false 
category: blog
author: jasonzurita 
description: Swifty File Reading and Writing
---

# Summary
There are situations where you may want to read/write to a file for later processing. Here are a couple cases that come to mind:
- Test logs: This is typically useful when testing an app or feature in the wild and you aren't able to have the debugger connected.
- Looking at sensor data: This has been a more common scenario for me. For example, I had one project where I was working with the sensors in the Apple Watch to determine specific wrist movement, and another project where I wanted to analyze the raw face geometry data coming from the iPhone X TrueDepth camera (lots of fun!). In both cases, I wanted to look at the sensor data in a spreadsheet, so I needed a way to record, view, and export the data.

As we will see, making use of the power of Swift protocols by providing default implementations with and without constraints can make adding additional functionality to an entity (class, struct, enum) a breeze. In this case, we will set up two protocols to allow an entity to write and read a file. When using the `FileReadWritable` protocol, writing will be as simple as calling `write()` and reading will be as simple as calling `read()`! 

---
# The Code
This reading/writing implementation is done using protocols with default implementations. As you will see, using protocols like this makes reading/writing very simple. Let's take a look at each protocol!

### Writing to a File
{% highlight swift %}
// 1
protocol FileWritable {
    static var fileName: String { get }
    static var encoding: String.Encoding { get }
    func write(_ text: String) throws
}

// 2
enum FileWriteError: Error {
    case directoryDoesntExist
    case convertToDataIssue
}

// 3
extension FileWritable {
    // 4
    static var fileName: String { return "File.txt" }

    // 5
    static var encoding: String.Encoding { return .utf8 }

    func write(_ text: String) throws {
        // 6
        guard let directory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else {
            throw FileWriteError.directoryDoesntExist
        }

        // 7
        let fileUrl = directory.appendingPathComponent(Self.fileName)

        // 8
        guard let fileHandle = FileHandle(forWritingAtPath: fileUrl.path) else {
            try text.write(to: fileUrl, atomically: true, encoding: Self.encoding)
            return
        }

        // 9
        guard let data = text.data(using: Self.encoding) else {
            throw FileWriteError.convertToDataIssue
        }

        // 10
        fileHandle.seekToEndOfFile()
        fileHandle.write(data)
    }
}
{% endhighlight %}

1. First we create a simple writing protocol.
2. We define an error enum to be used for error situations (you will see how this is used later).
3. We provide a default implementation for the `FileWritable` protocol. This will make using this protocol to write super easy and is one of the many reasons I love Swift! More on how to use this protocol in a bit.
4. The first protocol requirement is satisfied by using a computed property. This is the file name that will be written to, and has to be implemented as a computed property here because we are in a protocol extension. If you don't want to use the name defined in this computed property, you can simply provide your own `fileName` property in the entity that makes use of this protocol (class, struct, enum) instead of using the default file name; So don't think you are tied down to a specific file name or only one file for writing.
5. The second protocol requirement is satisfied in a similar way. This property is the string encoding for writing (I chose `utf8` for no particular reason). Later on, we will see this tie in nicely with the `FileReadable` protocol!
6. Here we get the url for the document directory. This is the directory where we will create and write to our file. If we aren't able to get the document directory the guard's else block gets called and we throw one of our custom `FileWriteError` errors defined in step 2.
7. We make a full url to the file that we want to use for writing including the file name. The file doesn't exist the first time we want to write and, as we will see, that is okay.
8. We attempt to get a file handle for writing to the file at the path that we created in step 7. This guard's else block gets called if that file doesn't exist, which will be the case the first time we want to write to a new file. In the else block, we simply create the file and write to it!
9. At this point, we know the file exists and that we have a file handle to it for writing. Here we convert the text to write to data using the encoding constant defined in step 5. If we cannot convert the text to data (i.e., `text.data(using: encoding)` returns nil), the guard's else block will be called and we throw one of our custom `FileWriteError` errors defined in step 2.
10. Hey, we now have data to write and a file handle ready to use for writing! All we have to do here is go to the end of the file (so that we don't overwrite it) and write our data.

#### Using FileWritable:
{% highlight swift %}
// 1
struct SensorData: FileWritable {
    let log: String
    func writeNameToFile() {
        do {
            write(log)
        } catch {
            // handle error
        }
    }
}
{% endhighlight %}

1. To write to a file, simply pick an entity (class, struct, enum), have it conform to the `FileWritable` protocol, and call write!! Make sure to handle the possibility that the call can throw (try!, try?, or in a do/try/catch). Neat, right?!

### Reading a File
{% highlight swift %}
// 1
protocol FileReadable {
    func fileUrl(for fileName: String) throws -> URL
    func read(fileName: String, encoding: String.Encoding) throws -> String
}

// 2
extension FileReadable {
    func fileUrl(for fileName: String) throws -> URL {
        guard let directory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else {
            throw FileWriteError.directoryDoesntExist
        }
        return directory.appendingPathComponent(fileName)
    }

    func read(fileName: String, encoding: String.Encoding) throws -> String {
        let url = try fileUrl(for: fileName)
        return try String(contentsOf: url, encoding: encoding)
    }
}

// 3
extension FileReadable where Self: FileWritable {
    // 4
    // 5
    func fileUrl(for fileName: String = Self.fileName) throws -> URL {
        // 6
        guard let directory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else {
            throw FileWriteError.directoryDoesntExist
        }

        // 7
        return directory.appendingPathComponent(fileName)
    }

    // 8
    func read(fileName: String = Self.fileName, encoding: String.Encoding = Self.encoding) throws -> String {
        // 9
        let url = try fileUrl(for: fileName)
        // 10
        return try String(contentsOf: url, encoding: encoding)
    }
}
{% endhighlight %}

1. We create a simple reading protocol with two methods. One for getting the url to a file given a file name (useful if you want to export a text file), and the second for reading a file given a name and string encoding. You may ask yourself, "wait, didn't we already define the file name and string encoding?". Yes, yes we did...😉
2. Similar to the `FileWritable` protocol, we provide a default implementation for the reading protocol. This extension is useful if you want to use this protocol in isolation, but Swift lets us marry this reading protocol with the writing protocol in a really nice way. So, lets skip this extension and work through the next default implementation in detail. Don't worry, these two default implementations are nearly identical, so by explaining the next one you will also understand what is going on with this one.
3. As mentioned in 2, this extension is very similar to the previous one. The only difference is that this extension is constrained to entities that conform to the `FileWritable` protocol. This is a nice strength of Swift protocols! By limiting this extension to the writing protocol, we can take full advantage of the fact that the file name and the string encoding has already been defined. We can do this because since we constrained this protocol extension to entities that implement `FileWritable`, and we know that the static properties `fileName` and `encoding` exist AND are the same as used for writing!
4. Here is the default implementation for the fileUrl protocol function requirement. Again, since we know that the static property `fileName` exists, we can use it as a default parameter. This makes calling this function real short, clean, and bug free.
5. One quick thing to note about the default argument though: We call `Self.fileName` with a capital `S` because we want the static property. If we used lowercase `s`, we would be referring to the instance of the entity that uses this protocol.
6. Similar to `FileWritable` his guard simply gets the document directory and throws if it doesn't exist.
7. Then we return the full path to the file including the file name.
8. Here is the default implementation for the read protocol function requirement. It also makes use of the _connection_ with the write protocol by setting default parameters for file name and encoding.
9. We need to get the file url, so we save a few electrons and call the previously defined function in step 4.
10. Finally, return the contents of the file using the given encoding. This can throw, so it is marked with `throws`.

#### Let's Get Swifty (FileReadWritable):
{% highlight swift %}
// 1
typealias FileReadWritable = FileReadable & FileWritable 
{% endhighlight %}

1. Similar to the [`Codable`](https://developer.apple.com/documentation/swift/codable) protocol, we make a new type alias which is the combination of the write and read protocols!! Hopefully you see how nice implementing the two protocols becomes. If not, let's look at some code.

#### Using FileReadWritable:
{% highlight swift %}
// 1
final class ReadLogsVc: UIViewController, FileReadWritable {
    override func viewDidLoad() {
        super.viewDidLoad()
        addTextView()
    }
    
    private func addTextView() {
        let textView = UITextView()
        textView.isEditable = false

        // 2
        // 3
        let text = try? read()

        // 4
        textView.text = text ?? "Failed to read text: \(ReadLogsVc.fileName)"

        // 5
        view.addSubview(textView, constraints: [
            equal(\.heightAnchor),
            equal(\.widthAnchor),
            equal(\.centerXAnchor),
            equal(\.centerYAnchor)
            ])
    }

    // 6
    func downloadLogs() {
        do {
            // 7
            let url = try fileUrl()

            // 8
            let objectsToShare = [url]
            let activityController = UIActivityViewController(activityItems: objectsToShare, applicationActivities: nil)
            strongSelf.present(activityController, animated: true, completion: nil)
        } catch {
            // handle error case 
        }
    }
}
{% endhighlight %}

1. This is a simple view controller that will read the text file you have written to and provide you with the option to view and download the logs. This view controller conforms to `FileReadWritable`!
2. Here we use our protocol to read the text!! A super simple call, right?! This is because this view controller conforms to both read and write protocols via the FileReadWritable alias, and that let's us make use of the more specialized version of our protocol extension.
3. Swift will use the most specialized version of a protocol implementation. Since this class conforms to the write protocol, Swift will use the second read protocol extension implementation because that one has the `where Self: FileWritable` constraint. If we put another implementation of `read()` in this class, Swift would use that implementation because it would be considered more specialized.
4. Since we used the `try?` to convert the throwing read function to an optional, we use [nil-coalescing operator](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/BasicOperators.html#//apple_ref/doc/uid/TP40014097-CH6-ID72) to provide a default text value if the read was unsuccessful.
5. This is out of the scope of this blog post, but this adds the text view to the main view and sets up its constraints. If you are interested in this syntax, read this great blog post by Chris Eidhof: [A Micro Auto Layout DSL](http://chris.eidhof.nl/post/micro-autolayout-dsl/)!
6. This function is used to download the logs (the text file).
7. Here is our other protocol function call, which gets the file url. Again, super simple to call given the specialized read protocol extension!!
8. This code lets you email, text, open the file in notes, AirDrop (my favorite), etc. the text file!

---

# Additional Thoughts 
- You don't have to use the document directory, but this typically works just fine. You may want to consider a different directory, or directories, if you write a lot of files to disk -- like if you are adventurous enough to persist all your model objects in text files 😶. I will leave this change as a reader exercise.
- The protocols can be written without throwing functions, but I tend to like throwing in cases like this for a few reasons:
  - Some of the existing API that we call are already throwing, so an error from those function will bubble up nicely by default and be processed together with our additional custom `FileWriteError` error cases.
  - When failure can happen for a number of reasons, knowing the specific reason can be useful.
  - And, you could simply ignore the potential throws with a `try!` or convert the possible errors into an optional using `try?`.


Please feel free to reach out, I would appreciate any feedback!
