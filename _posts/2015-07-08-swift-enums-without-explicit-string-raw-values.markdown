---
layout: post
title: "Swift: Enums without explicit string raw values"
tags: ios, swift
id: post-13
redirect_from: "/swift-2-enums-without-explicit-string-raw-values/"
---
Xcode 7 beta 3 brought us changes in Swift 2.0 language - It is not necessary
to set raw values explicitly for enums that inherit from String - Great!

If an element of an enum with string raw type does not have an explicit raw
value, it will default to the text of the enum's name.

For example:

{% highlight swift %}
enum CardinalDirection: String {
    case North, East, South, West
}
{% endhighlight %}

is the same like:

{% highlight swift %}
enum CardinalDirection: String {
    case North = "North"
    case East = "East"
    case South = "South"
    case West = "West"
}
{% endhighlight %}

Love this!
