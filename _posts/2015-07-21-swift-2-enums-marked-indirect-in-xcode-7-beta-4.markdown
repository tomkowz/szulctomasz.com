---
layout: post
title: "Swift 2: Enums marked \"indirect\" in Xcode 7 beta 4"
tags: swift, indirect enum, xcode
id: post-20
redirect_from: "/swift2-enums-marked-indirect-in-xcode-7-beta-4/"
---
Apple released Xcode 7 beta 4 today with some fixes and many improvements of
the Swift language. One of them is `indirect` attribute for enums and its cases.

It gives us possibility to create recursive enums. Let's take a look on example.
It is not possible in Xcode 7 beta 3, and Xcode will tell you
that *"Recursive value type Tree is not allowed"*.

{% highlight swift %}
enum Tree<T> {
    case Leaf(T)
    case Branch(left: Tree<T>, right: Tree<T>)
}
{% endhighlight %}

With Xcode 7 beta 4 and new `indirect` attribute you can create such recursive enum.

{% highlight swift %}
indirect enum Tree<T> {
    case Leaf(T)
    case Branch(left: Tree<T>, right: Tree<T>)
}
{% endhighlight %}

I believe next update will bring ability to use `indirect` with structs.

Nice, isn't it?
