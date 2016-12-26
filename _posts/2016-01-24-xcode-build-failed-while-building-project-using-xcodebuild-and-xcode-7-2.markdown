---
layout: post
title: "Xcode: BUILD FAILED while building project using xcodebuild and Xcode 7.2"
tags: xcode, bug, xcodebuild
post_id: post-45
excerpt: "Workaround for building project for simulator using xcodebuild and Xcode 7.2."
redirect_from: "/build-failed-while-building-project-using-xcodebuild-and-xcode-7-2/"
---
On Friday I worked on a framework and wanted to create script that would give me
Debug or Release version of a framework, so it can run both on simulator and
actual device. I managed to do this finally but it took me about a long time to
realize that there is a bug in Xcode 7.2 that does not allow you to build project
for simulator.

Reported bug: [rdar://23857648][rdar-1] (Copied below)

{% highlight plain %}
Summary:
xcodebuild fails to build a simple project on the command line. It seems there is
a problem setting the platform variable. Some discussion here:
https://forums.developer.apple.com/thread/27975

Steps to Reproduce:
1. Create a new iOS 'Single View Application' called 'TestBuildCmd' from the project template in Xcode 7.2
2. In a terminal cd to the directory containing the project file (TestBuildCmd.xcodeproj)
3. Execute the following command to build on the command line:
xcodebuild -project TestBuildCmd.xcodeproj -scheme TestBuildCmd -sdk iphonesimulator clean build

Expected Results:
project builds and completes with "** BUILD SUCCEEDED **" output

Actual Results:
build fails. Lots of errors like this:
/In file included from <module-includes>:1:
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator9.2.sdk/usr/include/sys/cdefs.h:707:2: error: Unsupported architecture
#error Unsupported architecture
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator9.2.sdk/usr/include/sys/_types.h:55:9: error: unknown type name '__int64_t'
typedef __int64_t       __darwin_blkcnt_t;      /* total blocks */
        ^
...

Version:
Xcode 7.2 (7C68) / iOS 9.2 SDK
OS X 10.10.5
{% endhighlight %}


To make it work you need to add argument to the xcodebuild command `-destination 'platform=iOS Simulator,name=iPhone 6,OS=latest'`.

[rdar-1]: http://www.openradar.me/23857648
