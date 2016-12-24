---
layout: post
title: "iOS: Selecting audio inputs"
tags: ios, swift, avfoundation
id: post-46
excerpt: "Getting proper audio input for audio recording."
redirect_from: "/ios-selecting-audio-inputs/"
---
Recently I learned how to select microphone inputs and switch between them.
I think it is worth to share, so you might can find it useful. The case was
to switch between built-in and external headset microphone.


### Getting devices for media type
The system is returning just one audio device no matter whether headphones
are connected or not.

{% highlight swift %}
import AVFoundation
...

private var session: AVAudioSession!
private var inputs = [AVAudioSessionPortDescription]()
...

/// configure audio session
session = AVAudioSession.sharedInstance()
try! session.setCategory(AVAudioSessionCategoryPlayAndRecord)

/// get devices
let devices = AVCaptureDevice.devicesWithMediaType(AVMediaTypeAudio)
{% endhighlight %}


{% highlight plain %}
// for phone with just built-in microphone
<__NSArrayM 0x15e6a3d0>(
<AVCaptureFigAudioDevice: 0x15e676f0 [iPhone Microphone][com.apple.avfoundation.avcapturedevice.built-in_audio:0]>
)

// for phone with headset connected
<__NSArrayM 0x1665c3d0>(
<AVCaptureFigAudioDevice: 0x16663560 [Headphones][com.apple.avfoundation.avcapturedevice.built-in_audio:0]>
)
{% endhighlight %}

Well, by asking for device with specific media type we're not able to select proper
audio input - system selects the source on its own.

### Ports
Instead of asking for devices with specific media type we can ask AVAudioSession
to return all input ports, iterate through them and select the ones we need.

{% highlight swift %}
if let availableInputs = session.availableInputs {
    for input in availableInputs {
        if input.portType == AVAudioSessionPortBuiltInMic ||
            input.portType == AVAudioSessionPortHeadsetMic {
            inputs.append(input)
        }
    }
}
{% endhighlight %}

To select an input, we need to set it as a preferred input.

{% highlight swift %}
private func selectInput(input: AVAudioSessionPortDescription) {
    do {
        try session.setPreferredInput(input)
    } catch let error {
        print(error)
    }
}
{% endhighlight %}
