---
layout: post
title: "iOS: Low Power Mode"
tags: ios, ios9, low power mode, swift, xcode
id: post-21
redirect_from: "/ios-low-power-mode/"
---
*"Even longer battery life."* - Apple says. Low Power Mode is a new feature of
iOS 9 which lets iOS users extend their devices battery life. For now up to one
hour, but hey, this is additional one hour, right?

### About Low Power Mode

This feature uses ambient light and proximity sensors to know if device is
facedown so system does not turn on device's screen e.g. when notification
came in. But this is not all about the feature. System changes behaviour
itself as much as it can, including:

1. Reduce CPU and GPU performance
2. Pause background activities including networking
3. Reduce screen brightness
4. Reduce the timeout for auto-locking the device
5. Disable Mail fetch
6. Disable motion effects
7. Disable animated wallpapers


Apple recommends to take additional steps and reduce animations
(including frame rate), stop location updates or just reduce number of those
updates, disable syncs and backups.

### Usage

You can determine Low Power Mode state changes by observing
`NSProcessInfoPowerStateDidChangeNotification` and current state by
checking `lowPowerModeEnabled` property of `NSProcessInfo` class.


{% highlight swift %}
override func viewWillAppear(animated: Bool) {
    super.viewWillAppear(animated)
    startObservingLowPowerModeChanges()
}

override func viewWillDisappear(animated: Bool) {
    super.viewWillDisappear(animated)
    stopObservingLowPowerModeChanges()
}

func startObservingLowPowerModeChanges() {
    NSNotificationCenter.defaultCenter().addObserver(self,
        selector: Selector("lowPowerModeDidChangeNotification"),
        name: NSProcessInfoPowerStateDidChangeNotification,
        object: nil)
}

func stopObservingLowPowerModeChanges() {
    NSNotificationCenter.defaultCenter().removeObserver(self,
        name: NSProcessInfoPowerStateDidChangeNotification,
        object: nil)
}

func lowPowerModeDidChangeNotification() {
    let state = NSProcessInfo.processInfo().lowPowerModeEnabled
    print("low power mode did change notification, current state: \(state)")
}
{% endhighlight %}

You can enable Low Power Mode on your actual device in
*Settings > Battery > Low Power Mode*.

Unfortunately this is not possible on
simulator, at least for now - [rdar://21945440][rdar-1].

[rdar-1]: http://www.openradar.me/21945440
