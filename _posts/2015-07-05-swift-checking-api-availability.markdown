---
layout: post
title: "Swift: Checking API availability"
tags: ios, swift
id: post-9
redirect_from: "/swift-2-check-availability/"
---
This is very nice addition in Swift 2. It provides API availability checking
and makes our lives easier.

If project has deployment target set to a lower than the SDK used in the
project we can add checks in the code to make sure that API to be executed
fits iOS version.

Before Swift 2 we're forced to check e.g. if class exists or if object responds
to selector or something similar. Checking class availability was IMO the worst
method ever because class might exist and be only not available to be used
(e.g. was private, or something). Responds to selector still works nice in code
before Swift 2 but this method is cursed by risk. selector might be wrongly
constructed and app simply crash during calling such API.

`#available` is very  simple to use and avoid inheriting Swift classes from
`NSObject` just because of necessity of using `respondsToSelector:` method.

`#available` consists of few parameters - list of lowest operating system
versions that can execute the code, and required `*` at the end which means
*unknown platforms* or just not listed.

Here is example of class which detects which one animation should be used
by `Animator`.

{% highlight swift %}
import UIKit

class Animator {
    private var animation: CAAnimation

    private init(animation: CAAnimation) {
        self.animation = animation
    }

    class func animator(keyPath: String, force: CGFloat = 1.0) -> Animator {
        var animation: CABasicAnimation!
        if #available(iOS 9.0, *) {
            animation = CASpringAnimation(keyPath: keyPath)
            let spring = animation as! CASpringAnimation
            spring.mass = 5.0
            spring.stiffness = 80 * force
            spring.damping = 1.0 * force
        } else {
            animation = CABasicAnimation(keyPath: keyPath)
        }

        animation.fromValue = 1
        animation.toValue = 0.3 * force
        animation.duration = 0.5
        animation.autoreverses = true
        return Animator(animation: animation)
    }

    func animate(view: UIView) {
        view.layer.addAnimation(animation, forKey: "animator")
    }
}
{% endhighlight %}

No more class presence or `respondsToSelector:` checks.

Besides checking we're also able now to mark methods availability using
`@available`. It works pretty same like `#available`.

{% highlight swift %}
@available(iOS 9, *)
func springIt(view: UIView, keyPath: String) {
    /// Super extra CASpringAnimation...
}
{% endhighlight %}

Xcode 7 is quite helpful when calling this method. It automatically examine
that line of code:

{% highlight swift %}
animator.springIt(view, keyPath: "transform.scale")
{% endhighlight %}

and mark this line with error *"springIt() is only available on iOS 9 or newer"*
and suggest to do convert this line of code into safety version:

{% highlight swift %}
if #available(iOS 9, *) {
    animator.springIt(view, keyPath: "transform.scale")
} else {
    // Fallback on earlier versions
}
{% endhighlight %}

This is super cool.
