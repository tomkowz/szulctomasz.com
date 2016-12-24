---
layout: post
title: "iOS: Custom View Controller Transition by expanding view"
tags: ios, custom transition, view controller
id: post-49
excerpt: "Implemented custom view controller transition inspired by work on dribbble.com"
---
I was looking for inspirations on *dribbble.com* and found work
that shows nice transition between view controllers by expanding a view (e.g. a button) in
every direction. You can find it here: [Login & Home Screen](https://dribbble.com/shots/1945593-Login-Home-Screen).

It is more like Android material design style animation, but I think it may
fit nicely in any onboarding process animations.

### The animation
Let's disassemble this animation.

- There is a view that is a source of animation. Let's call it **expandable view**.
In this work on dribbble the **expandable view** view is a "Sign in" button.
(I'll simplify it and just use button that is ready to be scaled. You could use
additional delegate to notify the expandable view that it should prepare for
being expanded - It could also implement *Expandable* protocol or something similar)
- When user taps the button, the first step is to change the appearance of the
**expandable view**. The view is about to scale to fill entire screen, so
everthing that would not look good is removed from the view. In this case this
is loading spinner and plus sign that is removed from the view. The only thing
that will be scaling is simple circle button with solid background.
- The next step is the scaling process itself. The button expands so it fills
entire screen.
- After view expanded quickly next view controller is presented by simple fade.

### Custom transitions in iOS
One way to approach the problem is to create custom transition
animation. This can be done by [UIViewControllerAnimatedTransitioning](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewControllerAnimatedTransitioning_Protocol/) and [UIViewControllerTransitioningDelegate](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewControllerTransitioningDelegate_protocol/index.html) protocols.

**UIViewControllerAnimatedTransitioning** - The protocol is used by animator,
the class that knows how the animation look like.

**UIViewControllerTransitioningDelegate** - The protocol is used by transitioning
delegate. It knows what is the total length of the animation and can instantiate
and return animators for both presenting and dismissing view controllers.

*In this case I'll only care about presenting animation.*

### Example project

You can find demo project in [ExpandingViewTransition-Demo repository](https://github.com/tomkowz/ExpandingViewTransition-Demo) on github.

### Transitioning delegate
At first, we have to create transitioning delegate that will return animators
for presenting and dismissing view controller (we care only about presenting now).

{% highlight swift %}
import UIKit

class ExpandingViewTransition: NSObject, UIViewControllerTransitioningDelegate {

    private let expandableView: UIView
    private let expandViewAnimationDuration: NSTimeInterval
    private let presentVCAnimationDuration: NSTimeInterval

    init(expandingView: UIView,
         expandViewAnimationDuration: NSTimeInterval = 0.35,
         presentVCAnimationDuration: NSTimeInterval = 0.35) {
        self.expandableView = expandingView
        self.expandViewAnimationDuration = expandViewAnimationDuration
        self.presentVCAnimationDuration = presentVCAnimationDuration
    }

    func animationControllerForPresentedController(
        presented: UIViewController,
        presentingController presenting: UIViewController,
                             sourceController source: UIViewController) -> UIViewControllerAnimatedTransitioning? {

        return ExpandingViewTransitionAnimatorPresent(expandableView: self.expandableView,
                                                      expandViewAnimationDuration: self.expandViewAnimationDuration,
                                                      presentVCAnimationDuration: self.presentVCAnimationDuration)
    }

    func animationControllerForDismissedController(
        dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return nil
    }
}
{% endhighlight %}

The `init(expandableView:expandViewAnimationDuration:presentVCAnimationDuration:)` takes
reference to view that will be expanded (button), and animation durations that
specifies how long expanding process is and how long is fade of destination
view controller after *expandable view* is fully expanded.

Two remaining methods are the one from *UIViewControllerTransitioningDelegate*.
First returns *ExpandingViewTransitionAnimatorPresent* animator which is used
to present new view controller.

Second method returns *nil* as we don't care about dismissing animation.
Default dismiss animation will be used in this case.

### Animator

The foundation of animator look like this.

{% highlight swift %}
class ExpandingViewTransitionAnimatorPresent: NSObject, UIViewControllerAnimatedTransitioning {
  private let expandableView: UIView
  private let expandViewAnimationDuration: NSTimeInterval
  private let presentVCAnimationDuration: NSTimeInterval

  init (expandableView: UIView,
        expandViewAnimationDuration: NSTimeInterval,
        presentVCAnimationDuration: NSTimeInterval) {
      self.expandableView = expandableView
      self.expandViewAnimationDuration = expandViewAnimationDuration
      self.presentVCAnimationDuration = presentVCAnimationDuration
  }

  func transitionDuration(transitionContext: UIViewControllerContextTransitioning?) -> NSTimeInterval {
      return self.expandViewAnimationDuration + self.presentVCAnimationDuration
  }

  func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
      // ...
  }
}
{% endhighlight %}

The first thing is to calculate how much the *expandable view* should expand.
The idea is simple. We need to check how far the view is from the edges of the screen,
take the longest distance and use it to calculate scale that should be applied.

After we find such maximal offset, then we look for the biggest dimension of the
*expandable view* and for longest dimension of the source view controller
which should be full screen - I think we could use dimension of `UIScreen.mainScreen().bounds` too.
It might work even better for some cases.

With such informations we're able to calculate new scale properly.

{% highlight swift %}
/**
 The method calculates a scale that expandable view should transform to
 in order to fill entire screen no matter where it is located on the screen.
 */
private func calculateFinalTransformOfExpandingViewInSourceVC(
    expandingView: UIView,
    sourceVC: UIViewController) -> CGAffineTransform {

    // left, right, top, bottom
    let offsets = [expandingView.frame.origin.x,
                   sourceVC.view.bounds.width - expandingView.frame.origin.x,
                   expandingView.frame.origin.y,
                   sourceVC.view.bounds.height - expandingView.frame.origin.y]

    let minExpandingViewDim = min(expandableView.bounds.width,
                                  expandableView.bounds.height)

    let maxOffsetVal = offsets.maxElement()!
    let maxSourceDim = max(sourceVC.view.bounds.width,
                           sourceVC.view.bounds.height)

    // to make sure that source is filled by `expandingView`
    // especially if the view has rounded edges
    let threshold_scale: CGFloat = 2

    let scale = (maxSourceDim + maxOffsetVal) / minExpandingViewDim + threshold_scale
    return CGAffineTransformMakeScale(scale, scale)
}
{% endhighlight %}

Here is how the transition code look like.

{% highlight swift %}
func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
    /**
     Transitions involves:
     - adding source and destination view controller to the container view
     - transforming expandable view so it fills entire screen
     - finally displaying the destination view controller
     - bringing back previous state of expandable view
     */

    let sourceVC = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)!

    let destinationVC = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)!
    destinationVC.modalPresentationStyle = .Custom
    destinationVC.view.alpha = 0

    let containerView = transitionContext.containerView()!
    containerView.addSubview(sourceVC.view)
    containerView.addSubview(destinationVC.view)

    let beginZPosition = self.expandableView.layer.zPosition
    let beginTransform = self.expandableView.transform

    let finalTransform = calculateFinalTransformOfExpandingViewInSourceVC(self.expandableView, sourceVC: sourceVC)

    self.expandableView.layer.zPosition = CGFloat.max - 1

    UIView.animateWithDuration(self.expandViewAnimationDuration, delay: 0, options: .CurveEaseIn, animations: {
        self.expandableView.transform = finalTransform
        }, completion: { _ in
            UIView.animateWithDuration(self.presentVCAnimationDuration, animations: {
                destinationVC.view.alpha = 1
                }, completion: { _ in
                    self.expandableView.transform = beginTransform
                    self.expandableView.layer.zPosition = beginZPosition

                    transitionContext.completeTransition(true)
            })
    })
}
{% endhighlight %}

Using *UITransitionContextFromViewControllerKey* and *UITransitionContextToViewControllerKey*
keys, we can obtain source and destination view controllers via `viewControllerForKey` method.

Then as documentation says presentation style of destination view controller is set
to custom - `destinationVC.modalPresentationStyle = .Custom`.

*Animator* provides us `containerView` via `transitionContext` - see [UIViewControllerContextTransitioning](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewControllerContextTransitioning_protocol/) for details.
Source and destination view controllers are added to this container view.

We have to store somewhere begin state of `zPosition` and `layer.transform` of
*expandable view* so we can bring it back to the state before scaling.

After this there is just simple animation of a `transform` of *expandable view*
and after this finished, the destination view controller is presented.

After all of this is done and only destination view controller is on screen
we can bring previous appearance of the *expandable view*.

When everything is done you have to mark that the transition completed.
{% highlight swift %}
transitionContext.completeTransition(true)
{% endhighlight %}

![transition-2][img-2]
![transition-4][img-4]


### How to use it?
Here is one way you can use it programatically:

{% highlight swift %}
@IBAction func doAnimate(sender: UIView) {

    let transitionDelegate = ExpandingViewTransition(expandingView: sender,
                                                     expandViewAnimationDuration: 0.4,
                                                     presentVCAnimationDuration: 0.1)

    let storyboard = UIStoryboard(name: "Main", bundle: nil)

    let vc = storyboard.instantiateViewControllerWithIdentifier("SecondViewController")
    vc.transitioningDelegate = transitionDelegate

    self.presentViewController(vc, animated: true, completion: nil)
}
{% endhighlight %}

[img-1]: /uploads/{{page.id}}/transition-1.gif
[img-2]: /uploads/{{page.id}}/transition-2.gif
[img-3]: /uploads/{{page.id}}/transition-3.gif
[img-4]: /uploads/{{page.id}}/transition-4.gif
