---
layout: post
title: "Xcode: XCConfig files for managing targets configurations"
tags: swift, xcode
post_id: post-40
excerpt: "Easier configuration management using XCConfig files."
redirect_from: "/xcode-xcconfig-files-for-managing-targets-configurations/"
---
Let's take a look how XCConfig files can be useful when working on multiple
targets with some differences in configuration.

I browsed [mozilla/firefox-ios][firefox] repository today with plan that I'll
learn something new, and I found that they are using a lot of Configuration
Settings Files.

I used xcconfig files in the past in some project, but do not use them in
current project that I am working on daily. In the project there are multiple
targets with some differences in configuration and I was thinking how could I
manage these things efficiently and easily.

### Use case
The project has been inherited by my team. Customer's team worked on it first
for like half a year and decided to outsource it completely. One of the poor
things of this project are targets with differences in configurations and how
the problem has been solved - or better, not yet solved.

The project consists of 10 application targets, 2 aggregate targets that do
some stuff and 1 testing target. Every target is using different backend and
different pair of let's say "api keys" and other keys like tokens for hockeyapp.
Every target has own preprocessor macro like "TARGET_A", "TARGET_B", etc..
(fictional names). Next, details like tokens, api keys, urls of backends are
stored in plist file, so naturally there need to be some class that is wrapper
for this file and can parse it and give us proper keys. And this class has
over 200 lines of code, which for me is a way to much just for reading such data.

So, I thought that maybe I could use xcconfig files and keep this simple,
instead of using parsers and juggle of 10 preprocessor macros (one per target)
to decide which values from plist file should be returned. You can find my
solution below. It might not be the best one, but for this moment it is. If
you have better one, I'll willingly read about it :)

### Overview
The idea is to have few levels of configuration files. First level is for storing
most common data, second level is for differentiating between debug and release
modes and last level is for settings related to specific targets.

![imgage-1][img-1]


### Common.xcconfig
This file stores informations like app name, app version and bundle version,
and other most commmon settings that are the same for debug and release targets.

{% highlight objc %}
//
//  Common.xcconfig
//  <truncated>
//

APP_NAME = App
APP_VERSION = 1.6
APP_BUNDLE_ID = 153
{% endhighlight %}

Changing app version and bundle for every target may be time consuming when
considering doing it for 10 targets. Other option may be creating Aggregate
target that could update Info-plist files every Cmd+B but I would like to avoid
this and do not complicate the project more than it is now.

### Common.debug and Common.release
These files can store most common configuration for both debug and release targets.
Files includes Common.xcconfig and can override its variables. E.g. you can
easily change app's name for every debug target to "App Debug" by overriding
one variable. This is also good place to store common API Keys that are used
by each develop or release targets.

### Tip: Using custom configuration files and CocoaPods
If you're using CocoaPods you should include Pods.debug.xcconfig or
Pods.release.xcconfig respectively in one of your configuration files. I
recommend to set your configuration file first in Project Info tab and the execute
`pod install` to let the Pods project reconfigure. After installation is ended
you'll be prompted to include one of Pods' configuration files to your file.


Error:

> [!] CocoaPods did not set the base configuration of your project because your project already has a custom config set. In order for CocoaPods integration to work at all, please either set the base configurations of the target `TARGET_NAME` to `Pods/Target Support Files/Pods/Pods.debug.xcconfig` or include the `Pods/Target Support Files/Pods/Pods.debug.xcconfig` in your build configuration.</em>

{% highlight objc %}
//
//  Common.debug.xcconfig
//  <truncated>
//

#include "Common.xcconfig"
#include "Pods/Target Support Files/Pods/Pods.debug.xcconfig"

APP_NAME = App Debug
API_KEY_A = API_KEY_HERE
API_KEY_B = API_KEY_HERE
{% endhighlight %}

### PerTarget.xcconfig
I do not need debug/release configuration files on this level
(because of other legacy issues in the project), so I just have PerTarget.xcconfig
files that include proper Common.debug.xcconfig or Common.release.xcconfig
file to it. But better would be to have debug and release configuration files.
On this level you can keep things related to a specific target.

{% highlight objc %}
//
//  Develop.xcconfig
//  <truncated>
//

#include "Common.debug.xcconfig"

BACKEND_URL = http:\/\/develop.api.szulctomasz.com
SOME_KEY_A = VALUE_HERE
SOME_KEY_B = VALUE_HERE
{% endhighlight %}

### Accessing variables
All configuration data is stored. Now is time to use them. With so many targets
like in my case I can reduce number of Info.plist files to just 1 instead of
having multiple files because all differences are in xcconfig files now.

You can see that after you build the app with such configuration files there
will be some values in Project's Build Settings in "User-Defined" section.

If you want to use variable from configuration file e.g. in Info.plist file of a
target you have to use this pattern: `$(VARIABLE)`. This way you can set
"Bundle Identifier", "Bundle name", "Bundle version" and other stuff you
need to configure.

Accessing other variables in the code looks different and a way that I
found the easiest one is to create additional fields in Info.plist with
the same name as a variable and assign value to them using pattern presented above. Then you can read the value in the code.

{% highlight swift %}
if let dictionary = NSBundle.mainBundle().infoDictionary {
    let appName = dictionary["APP_NAME"] as! String
    let appVersion = dictionary["APP_VERSION"] as! String
    let appBuildVersion = dictionary["APP_BUILD_VERSION"] as! String
    print("\(appName) \(appVersion) (\(appBuildVersion))")

    let backend = (dictionary["BACKEND_URL"] as! String).stringByReplacingOccurrencesOfString("\\", withString: "")
    print("backend: \(backend)")
}
{% endhighlight %}

Here is the [tomkowz/demo-xcconfig][gh-demo] repository where you can find
example of using xcconfig files.

### Conclusion
Xcode Configuration Settings files give easy way of configuring targets and
allow to maintain configurations easily. In my use case it is great to switch
to these files, because now maintenance of configurations is really easy
compared to what I had before using this solution.

[firefox]: https://github.com/mozilla/firefox-ios
[gh-demo]: https://github.com/tomkowz/demo-xcconfig

[img-1]: /uploads/{{page.post_id}}/1.png
