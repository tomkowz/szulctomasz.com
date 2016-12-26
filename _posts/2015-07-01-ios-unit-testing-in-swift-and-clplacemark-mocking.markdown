---
layout: post
title: "iOS: Unit testing in Swift and CLPlacemark mocking"
tags: ios, swift, unit testing, tdd
post_id: post-8
redirect_from: "/unit-testing-in-swift-with-clplacemark/"
---
Joined one project yesterday and I am responsible for converting few Objective-C
classes to Swift where I can get some experience in Objective-C and Swift
interoperability.

We're doing unit tests - this is the main requirement that new code should
contain unit tests. As you probably know there is no good framework for mocks
and stubs because of how Swift's runtime works. We're writing tests with
[Quick][quick] and [Nimble][nimble] - nice BDD frameworks for Swift.

I spent a lot of time trying to find a way how to mock `CLPlacemark` class to
give me `location`, `locality` and `administrativeArea`. I tried to subclass it
and so on, and test always ends test with `EXC_BAD_ACCESS`.

The class is tricky to initialize instance of it because it needs
another `CLPlacemark`  as initialize method argument. There is one initializer
which takes no arguments and I tried it with subclassing too. Even overriding
properties does nothing.. Object is initialized and properties are accessible
but still crashing at some point of test. I think it crashes because it requires `CLPlacemarkInternal` on some point of life or something...

Anyway, after hour of searching for solution I thought: *"hey, maybe MapKit?!"*. 
And guess what? There is subclass of `CLPlacemark` called `MKPlacemark` which
takes `CLLocation`  and `Dictionary` with address details instead of `CLPlacemark` like `CLPlacemark` does... Yeah! I found it, but why it took so long.

If you want to somehow use fake `CLPLacemark` in your Swift's unit tests here is ready solution:

{% highlight swift %}
func placemark() -> CLPlacemark {
    return MKPlacemark(coordinate: CLLocationCoordinate2D(latitude: 10, longitude: 20), addressDictionary: ["City": "Palo Alto", kABPersonAddressStateKey: "CA"])
}
{% endhighlight %}

[quick]: https://github.com/Quick/Quick
[nimble]: https://github.com/Quick/Nimble
