---
layout: post
title: "Starting project in Swift 2 for iOS 9/8 features testing"
tags: ios, carthage, nimble, quick, bdd, swift, tdd
id: post-11
redirect_from: "/starting-project-in-swift-2-for-ios-9-8-features-testing/"
---
Today I started new project on github, called [Quotes][quotes]. It has been
created for testing features of iOS 9 and iOS 8 too which I was not able to
test in the past or not tried yet because things are new.

I thought it would be great idea to create project containing new features and
start doing this in project that could work mainly with text so UI work will
be a bit limited. I think that app storing quotes will be good for the beginning.
Let's give it a chance :)

I'll be also experimenting with [MVVM][mvvm] (Model View ViewModel) architectural
pattern so if you don't care about iOS 9 feature you probably will take a look
at least how this pattern works. It works nicely in one project I have
opportunity to work now and we'll see how it will work with *Quotes*. Project
is developing in Xcode 7 and Swift 2.

The next thing I would like to do are tests. These are often omitted which is
bad for the project and its safety. I wrote about two frameworks in the past
and also here I will be using [Quick][quick] and [Nimble][nimble] which I like
the most recently. These will be integrated with the project using [Carthage][carthage] dependency manager.

Any contribution is welcome. If you notice that something could be done better / could be improved - feel free to comment in blog posts or file an issue on github - I hope you will :)

[quotes]: https://github.com/tomkowz/Quotes
[mvvm]: https://en.wikipedia.org/wiki/Model_View_ViewModel
[quick]: https://github.com/Quick/Quick
[nimble]: https://github.com/Quick/Nimble
[carthage]: https://github.com/Carthage/Carthage
