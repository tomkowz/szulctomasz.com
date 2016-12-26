---
layout: post
title: "Integrating Travis CI with Swift project"
tags: ci, swift, travis
post_id: post-7
redirect_from: "/integrating-travis-ci-with-swift-project/"
---
Finally I successfully integrated [Travis CI][travis] with [Swifternalization][swifternalization]. I spent some time finding ready-to-go 
solution how to configure my Swift 1.2 project to work with Travis and
I couldn't find it.

I tried many combination and of course my git history looks terrible now...

{% highlight sh %}
1776cb2 - Tomasz Szulc <mail@szulctomasz.com> - Update .travis.yml file.
5b979bb - Tomasz Szulc <mail@szulctomasz.com> - Update .travis.yml file.
8f3b9b3 - Tomasz Szulc <mail@szulctomasz.com> - Update .travis.yml file.
7c76e79 - Tomasz Szulc <mail@szulctomasz.com> - Change deployment target to 8.0
c594adf - Tomasz Szulc <mail@szulctomasz.com> - Update .travis.yml file.
7684071 - Tomasz Szulc <mail@szulctomasz.com> - Make schemes shared
346d5a1 - Tomasz Szulc <mail@szulctomasz.com> - Update .travis.yml file.
39a2c20 - Tomasz Szulc <mail@szulctomasz.com> - Update .travis.yml file.
d805f72 - Tomasz Szulc <mail@szulctomasz.com> - Add .travis.yml file.
{% endhighlight %}

But, I will be very nice colleague and will share a recipe :)

First thing is to make the test scheme shared. Go to
Xcode > Product > Scheme > Manage Schemes... and check "Shared" next to your
test target. This is necessary to make this scheme accessible outside of
the project.

![image-1][img-1]

Next go to Travis and select your github repository to be enabled for continuous integration. You can enable your repo [here][travis-profile] if you're signed in.

The next thing is to add *.travis.yml* file to your repo and configure it.
The configuration may look like following:

{% highlight sh %}
language: objective-c

branches:
 only:
 - master

xcode_project: Swifternalization.xcodeproj
xcode_scheme: SwifternalizationTests
osx_image: beta-xcode6.3
xcode_sdk: iphonesimulator8.3

script:
- xcodebuild clean build test -project Swifternalization.xcodeproj -scheme SwifternalizationTests
{% endhighlight %}

As you can see there is *objective-c* set as the language but this is fine
since the environment is the same both for Objective-C and Swift projects.

The *.travis.yml* file should be placed in project's base directory.

Commit it. Push. Go to [Travis CI site][travis] and wait for build.
Hope it will pass :)

#### Additional resources:

- [Xcode 6.3 availability - Travis CI][blog-1]
- [Building an Objective-C Project - Travis CI][travis-2]

[travis]: https://travis-ci.com
[swifternalization]: https://github.com/tomkowz/Swifternalization
[travis-profile]: https://travis-ci.org/profile/
[blog-1]: http://blog.travis-ci.com/2015-05-26-xcode-63-beta-general-availability/
[travis-2]: http://docs.travis-ci.com/user/languages/objective-c/

[img-1]: /uploads/{{page.post_id}}/1.png
