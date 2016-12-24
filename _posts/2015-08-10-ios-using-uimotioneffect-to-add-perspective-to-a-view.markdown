---
layout: post
title: "iOS: Using UIMotionEffect to add perspective to a view when tilting device"
tags: ios, swift, uimotioneffect
id: post-30
redirect_from: "/ios-using-uimotioneffect-to-add-perspective-to-a-view-when-tilting-device/"
---
`UIMotionEffect` is a class that provides motion-based modifiers for views.
It allows views to react on tilting of a device - nice class that I've learned
about today.

The [documentation][docs] about the class is short. It is an abstract class so
you cannot create object of it directly (need to subclass) but Apple provides
two classes that you can use as is - those are `UIInterpolatingMotionEffect`
and `UIMotionEffectGroup`.

In short - you set min and max relative changes of value of a property you select
and when a device is in motion, the system modifies such property and modified
view behaves like icons on the Springboard or alert views, and so on.

Let's add modifiers which cause shifting view in horizontal and vertical axes:
{% highlight swift %}
extension UIMotionEffect {
    class func twoAxesShift(strength: Float) -> UIMotionEffect {
        // internal method that creates motion effect
        func motion(type: UIInterpolatingMotionEffectType) -> UIInterpolatingMotionEffect {
            let keyPath = type == .TiltAlongHorizontalAxis ? "center.x" : "center.y"
            let motion = UIInterpolatingMotionEffect(keyPath: keyPath, type: type)
            motion.minimumRelativeValue = -strength
            motion.maximumRelativeValue = strength
            return motion
        }

        // group of motion effects
        let group = UIMotionEffectGroup()
        group.motionEffects = [
            motion(.TiltAlongHorizontalAxis),
            motion(.TiltAlongVerticalAxis)
        ]
        return group
    }
}
...
imageView.addMotionEffect(UIMotionEffect.twoAxesShift(40))
{% endhighlight %}

Simple, isn't it?

I think next good example is to change `transform` property of a view's layer.
This type of motion effect will imitate how Safari tabs behave, each is tilted
and move a bit when device is in motion.

{% highlight swift %}
extension UIMotionEffect {
    class func twoAxesTilt(strength: Float) -> UIMotionEffect {
        // get relative change with `strength` passed to the main method.
        func relativeValue(isMax: Bool, type: UIInterpolatingMotionEffectType) -> NSValue {
            var transform = CATransform3DIdentity
            transform.m34 = (1.0 * CGFloat(strength)) / 2000.0

            let axisValue: CGFloat
            if type == .TiltAlongVerticalAxis {
                // transform vertically
                axisValue = isMax ? -1.0 : 1.0
                transform = CATransform3DRotate(transform, axisValue * CGFloat(M_PI_4), 1, 0, 0)
            } else {
                // transform horizontally
                axisValue = isMax ? 1.0 : -1.0
                transform = CATransform3DRotate(transform, axisValue * CGFloat(M_PI_4), 0, 1, 0)
            }
            return NSValue(CATransform3D: transform)
        }

        // create motion for specified `type`.
        func motion(type: UIInterpolatingMotionEffectType) -> UIInterpolatingMotionEffect {
            let motion = UIInterpolatingMotionEffect(keyPath: "layer.transform", type: type)
            motion.minimumRelativeValue = relativeValue(false, type: type)
            motion.maximumRelativeValue = relativeValue(true, type: type)
            return motion
        }

        // create group of horizontal and vertical tilt motions
        let group = UIMotionEffectGroup()
        group.motionEffects = [
            motion(.TiltAlongHorizontalAxis),
            motion(.TiltAlongVerticalAxis)
        ]
        return group
    }
}
...
let strength: Float = 0.5
imageView.addMotionEffect(UIMotionEffect.twoAxesTilt(strength))
imageView2.addMotionEffect(UIMotionEffect.twoAxesTilt(strength))
imageView3.addMotionEffect(UIMotionEffect.twoAxesTilt(strength))
{% endhighlight %}

Before adding motion effect you can add some default tilt of to a view to make
it always a bit tilted down.
{% highlight swift %}
extension UIView {
    func addTilt() {
        self.layer.transform = CATransform3DIdentity
        self.layer.transform.m34 = 1/1000.0
        self.layer.transform = CATransform3DRotate(self.layer.transform, CGFloat(M_PI_4), 1, 0, 0)
    }
}
...
imageView.addTilt()
imageView2.addTilt()
imageView3.addTilt()

// add some depth
imageView.layer.zPosition = -500.0;
imageView2.layer.zPosition = -300.0;
imageView3.layer.zPosition = 0;
{% endhighlight %}

Here is the final effect:
<iframe width="320" height="315" src="http://szulctomasz.com/wp-content/uploads/2015/08/ScreenFlow.mp4"></iframe>

[docs]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIMotionEffect_class/
