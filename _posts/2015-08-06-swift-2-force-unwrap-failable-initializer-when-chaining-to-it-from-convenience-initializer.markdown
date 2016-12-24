---
layout: post
title: "Swift 2: Force-unwrap failable initializer when chaining to it from convenience initializer"
tags: ios, swift, xcode
id: post-28
redirect_from: "/swift2-force-unwrap-failable-initializer-in-convenience/"
---
Xcode 7 beta 5 is out and there is nice addition to Swift. It is more about
Xcode 7 beta 5 compiler, but still it is available in Swift 2.

When your class/struct has failable initializer and you wanted to have some
convenience initializer which was non-failable you were not able to simply
force-unwrap the failable one. With new Xcode release you can just do it.

{% highlight swift %}
class Person {

    let fullName: String

    init?(fullName: String?) {
        self.fullName = fullName ?? ""
        if fullName == nil { return nil }
    }

    convenience init(firstName: String) {
        // Before: Cannot force unwrap value of non-optional type '()'
        self.init(fullName: firstName + " (No Surname)")!
    }
}
{% endhighlight %}

Like it!
