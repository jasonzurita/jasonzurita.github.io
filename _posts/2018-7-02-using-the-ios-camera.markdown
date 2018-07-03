---
title: "iOS Camera - A surface dive into a deep topic"
layout: post
date: 2018-7-02
image: /assets/images/profile2.jpg
headerImage: false
tag:
- iOS
- Swift
- Camera
- AVFoundation
- AVCaptureSession
- AVCaptureDevice
star: false 
category: blog
author: jasonzurita 
description: High level look at using the iOS camera (get started & a reference)
---

# Summary
There are different ways to access the camera on iOS devices. One simple way would be to use `UIImagePickerController`, which makes use of the stock system camera UI. This is useful if you want to quickly get up and running to take a picture or video, and you don't really need to leverage the full power of the camera (e.g., manual changing of the focus, exposure, how to process the camera feed, etc.).

If you are looking to do more _custom_ things with the camera, hopefully these high level notes help get you started and serve as a reference during your development! This post focuses on the camera, but these concepts can be extended to other phone sensors like the microphone.

---

## Resources
- [AVFoundation](https://developer.apple.com/av-foundation/) is the Apple framework for working with time-based audiovisual media.
- Apple's high level documentation on [Camera and Media Capture](https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture) is a good starting point read.
- Apple's awesome camera [sample app](https://developer.apple.com/library/archive/samplecode/AVCam/Listings/LICENSE_txt.html) is a valuable reference. I learned a whole lot by examining Apple's implementation.<br>
<sub>**Note**: The download link is a small, not obvious, button at the top 🙄. There are both Objective-C and Swift implementations included 👏!</sub>
- [AVCaptureSession](https://developer.apple.com/documentation/avfoundation/avcapturesession) - High level object that manages your camera session I/O (e.g., input camera feed and output such as video, photo, audio). Once your session is configured, you tell the session to run which starts everything!
- [AVCaptureDevice](https://developer.apple.com/documentation/avfoundation/avcapturedevice) - This naming is a bit confusing, but this class represents the different sensors on your phone. A _device_ might be the telephoto camera on a phone with dual lenses or a microphone. You use this class to define which sensor you want to use and how you want to use it (e.g., check for camera capabilities like the existence of a dual camera, supported frame rates, sensor selection like the front camera, change focal position). You pass an instance of `AVCaptureDevice` via `AVCaptureInput` to the session (before session configuration or after) to tell it what sensor to use and how to use it.

**Note** - _Input_ refers to data that the device can collect (camera feed, microphone, etc.). _Output_ refers to the ability for the iOS device to take the input and produce an output (video file, sound file, etc.).

## Steps to set up the camera

1. [Request authorization to use the camera](https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture/requesting_authorization_for_media_capture)<br>
<sub>**Note**: you need to add `NSCameraUsageDescription` to your plist with a description of why you want to access the camera. Your users will see this when asked to grand permission to use the camera. Apple has been pretty vocal about including an accurate and detailed description of why you want permission, so don't take this _description_ lightly. Also, if you don't add this, your app will just crash!</sub>

2. [Set up the capture session](https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture/setting_up_a_capture_session) - You use `AVCaptureSession` to configure your camera's session inputs (e.g., which camera & related settings such as focus type) and outputs (e.g., how to consume the input from the camera such as making video or pictures). You configure the capture session inputs by calling the capture session's `canAddInput` with an `AVCaptureInput`. Similarly, you configure the output by calling the capture session's `canAddOutput` with an `AVCaptureOutput`. Also, make sure to do this in the capture session's _begin_/_commit_ scope:<br>

   ```swift
func configure(session: AVCaptureSession) {
    . . .
    session.beginConfiguration()
    defer { session.commitConfiguration() }
    // configure your sesison
}
```
   <sub>**Note**: you are able to change configuration inputs and outputs after the session is running, but you will still need to wrap those changes in the above.</sub>

3. Get the camera preview ready - You will typically want to see what your camera is looking at, so this is an important step. Also, setting up the camera and seeing it work is such a cool feeling!
  
   Apple recommends using a `UIView` subclass with an `AVCaptureVideoPreviewLayer` as the backing layer (see the sample app in _Resources_ above), which ends up playing nice with interface builder. If you are like me and like to do things programmatically, something like this also works:<br>

   ```swift
final class CameraViewController: UIViewController {

    private let _previewLayer: AVCaptureVideoPreviewLayer = {
        let layer = AVCaptureVideoPreviewLayer()
        layer.videoGravity = .resizeAspectFill // fill the view
        return layer
    }()
    
    init(session: AVCaptureSession) {
        super.init(nibName: nil, bundle: nil)
        _previewLayer.session = session
    }
    
    required init?(coder _: NSCoder) { fatalError("\(#function) has not been implemented") }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        _previewLayer.frame = view.layer.bounds
        view.layer.addSublayer(_previewLayer)
    }
}
```
   <sub>**Note 1**: microphone only or _spy_ camera apps, may not want a camera preview 😅.</sub><br>
   <sub>**Note 2**: if your app supports multiple orientations, you will want to set the preview layer's `AVCaptureConnection`'s `videoOrientation` property (`videoPreviewLayer.connection?.videoOrientation = someOrientation`). This is usually done when initially setting up the `AVCatpureSession` and during orientation changes like in a view controller's [viewWillTransition(to:with:)](https://developer.apple.com/documentation/uikit/uicontentcontainer/1621466-viewwilltransition).</sub><br>
   <sub>**Note 3**: typically the camera preview is shown full screen, but since the preview is simply a view/layer you can shape and resize the preview how you would like! I have done a camera preview that lives in a small rectangle in a modal popup, which was smaller than the overall screen size.</sub>

4. Run the session! - Simply call `session.startSession()`. If you have permission to use the camera, configured the session properly, and added your preview, you will see a camera feed on your phone! This never gets old 😎!

**Note** - Apple recommends that you start your session off the main thread as it is a blocking operation; this keeps the UI responsive. Apple uses a _serial queue_ in their sample documentation and also uses it to put the other camera configuration changes on this queue such as setting up the capture session, changing inputs/outputs, capturing camera input to make an output, and changing device settings like focus or exposure. This keeps the UI as responsive as possible by offloading work from the main queue.

Also, Apple notes this in their sample project (see _Resources_ above), _"In general it is not safe to mutate an `AVCaptureSession` or any of its inputs, outputs, or connections from multiple threads at the same time."_ If you didn't use a serial queue, there could be a race condition when changing the camera session and related settings, which could result in undefined behaviors.

## What is my phone capable of?
- You can use [AVCaptureDevice.DiscoverySession](https://developer.apple.com/documentation/avfoundation/avcapturedevice/discoverysession) to get a list of the available devices (via the `devices` property. Also, devices refers to the camera, microphone, etc.). To do this, simply use `DiscoverySession.init` with the query parameters of `deviceTypes`, `mediaType`, and `position`!
- When you have an `AVCaptureDevice` object, you can see what video formats a given camera supports by looking at `device.formats`. You can then check to see what frame rates a given format supports by looking at the format's `videoSupportedFrameRateRanges`. Apple provides sample code to get the highest frame a little bit down on [this documentation page](https://developer.apple.com/documentation/avfoundation/avcapturedevice).
- Setting frame rate - You can use your `AVCaptureDevice`'s [activeVideoMinFrameDuration](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1389290-activevideominframeduration) and [activeVideoMaxFrameDuration](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1387816-activevideomaxframeduration) to bound the frame rate of the capture session. By setting these both to the same value, you can effectively _lock_ the frame rate to a given value.<br>
   <sub>**Note 1**: you can't arbitrarily set the camera session to a specific frame rate. The value used needs to be in the [videoSupportedFrameRateRanges](https://developer.apple.com/documentation/avfoundation/avcapturedevice/format/1387592-videosupportedframerateranges) mentioned above. The `videoSupportedFrameRateRanges` can be found on your `AVCaptureDevice` object. Assuming your device object is assigned the name _device_: `device.formats[0].videoSupportedFrameRateRanges` (this shows how to get the frame rate ranges for the first format of a given device.)</sub><br>
   <sub>**Note 2**: you can also adjust things like the exposure. Look at the documentation for `AVCaptureDevice`.</sub>

- Focus - You can automatically or manually adjust the focus of the camera's lens.
  - Automatic focusing - you can configure the camera to auto focus on whatever the camera is looking at or you can pass a coordinate from the camera preview to the `AVCaptureDevice`, which will auto focus on that area (set `device.focusPointOfInterest` to the `CGPoint` that you want to focus on. This is typically done by translating the user's tap on the camera preview. See Apple's sample app (see _Resources_ above).
  - Manual focusing - you can manually set and lock the focus to a particular value (between 0.0 and 1.0). Zero represents the shortest distance at which the lens can focus and 1.0 the furthest. To manually set the focus, use [setFocusModeLocked](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1624617-setfocusmodelocked).<br>
    <sub>**Note 1**: for more details, see _Managing Focus Settings_ and _Managing the Lens Position_ in the `AVCaptureDevice` documentation.</sub><br>
    <sub>**Note 2**: if you are going to try and change the focus to a specific value, make sure to check [isLockingFocusWithCustomLensePositionSupported](https://developer.apple.com/documentation/avfoundation/avcapturedevice/2361529-islockingfocuswithcustomlensposi).</sub><br>
    <sub>**Note 3**: make sure to wrap changes in [lockForConfiguration()](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1387810-lockforconfiguration) and [unlockForConfiguration()](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1387917-unlockforconfiguration). If you don't do this, either no changes will take effect or your app will crash. The documentation says that there is a return boolean if the lock is successful, but this is only in Objective-C.</sub>

---

# Additional Thoughts 
- Not included in this post, but here are some other things you may be interested in:
  - You should handle the camera session getting interrupted by things like an incomming phone call. If you want to continue your capture session after the interruption, you will need to tell the session run again.
  - Microphone setup & use.
  - The flashlight / camera flash is called `Torch` in Apple's documentation.
  - Photo image capture.

Please feel free to reach out, I appreciate any feedback!
