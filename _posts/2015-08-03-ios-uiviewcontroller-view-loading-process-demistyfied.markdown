---
layout: post
title: "iOS: UIViewController's view loading process demystified"
tags: ios, ios9, swift
post_id: post-26
redirect_from: "/ios-uiviewcontrollers-view-loading-process-demystified/"
---
[@NatashaTheRobot][natasha-twitter] posted about testing view controllers and
issues with loading views on her blog in
[The One Weird Trick For Testing View Controllers in Swift][natasha-post] post.

She also wrote:
> The key here is that Apple overrides the viewController’s view getter to call the loadView function and do a bunch of other things we have no access to. If anyone else has other great insights into why this works, feel free to add it in the comments!

Yeah.. this is interesting what is going on under the hood. She inspired me
and I started digging and debugging a bit. I've noticed there are two flows -
one when view controller is assigned to window as root view controller and one
when it is not (e.g. when you want to test view controller and you
  instantiated it from storyboard).

### View Controller as rootViewController
{% highlight swift %}
if self.window == nil {
    self.window = UIWindow(frame: UIScreen.mainScreen().bounds)
}

let storyboard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
let vc = storyboard.instantiateViewControllerWithIdentifier("ViewController")
self.window!.rootViewController = vc
{% endhighlight %}

This is the flow which takes a place e.g. in
`application:DidFinishLaunchingWithOptions:` method.

![image-1][img-1]

The flow starts by calling `makeKeyAndVisible` method of `UIWindow` which calls
its private `addRootViewControllerViewIfPossible` method that tries to add view
of root view controller to itself and presents it. `UIWindow` accesses `view` property
of the root view controller which starts a chain of loading view process.
The accessor calls `loadViewIfRequired` method which calls `loadView` method.
The `loadView` calls internal methods of `UIViewController` that loads nib with
a view that will be set.

There is one great talk from this year's WWDC which covered how Storyboard and
their nibs behaves at runtime - [Implementing UI Design in Interface Builder][wwdc-407].

After the loaded view is set, view controller calls its internal `_window` method
and read all the things like `preferedInterfaceOrientation`,
`supportedInterfaceOrientations`, `shouldAutorotate`, etc. Actually it calls
`_window` many times and there are some other methods called too.

Next `viewDidLoad` method is called and private `__viewWillAppear` which calls
`viewWillAppear` on view controller. View is about to be presented so there are
calls to `willMoveToWindow:`, `willMoveToSuperview:` and private
`_didMoveFromWindow:toWindow:` methods.

The next thing is setting up Auto Layout in the view, so `layoutMarginsDidChange`,
`didMoveToWindow`, `didMoveToSuperview`, `updateViewConstraints`,
`updateConstraints`, `layoutSublayersOfLayer`, `viewWillLayoutSubviews`, `layoutSubviews`
methods are called - there is a lot of them.

Finally `viewDidAppear:` is called and view is presented.

### View Controller loaded in testing
This is another important case worth to investigate. In this case you probably
don't want to create window and put your view controller under the test to it.
You just want to instantiate view controller instance from storyboard and test it.

Before Natasha's post I known just one method to load view of a view controller
and make sure it is loaded. I did it by accessing view property directly after
view controller was created. Today I've learned about `loadViewIfNeeded` method
which is available since iOS 9 and behaves the same way like accessing `view`
property. I've also learned third - IMO the best method to load the view controller
during testing and make sure it is ready to use - presented by [Ørta][orta-twitter].
This is about calling `beginAppearanceTransition:animated` and
`endApperanceTransition` methods. [Here is solution he shared][artsy-github] - covered
below.

### Accessing view directly
Let's take a look on a flow when accessing view directly.
{% highlight swift %}
let storyboard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
let vc = storyboard.instantiateViewControllerWithIdentifier("ViewController")
_ = vc.view
{% endhighlight %}

And the flow is like below:

![image-2][img-2]

Okaaay... There is much less calls. View has been accessed and loaded.
The window is nil so I think no more methods have been called and the flow
ended up in `viewDidLoad`. A bit strange. Some configuration might be inside
`viewWillAppear` or `viewDidLoad`. This method should work in most test cases.
`loadViewIfNeeded` works the same way.

Let's check how `beginApperanceTransition:animated:` behaves.

### beginApperanceTransition:animated
{% highlight swift %}
let storyboard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
let vc = storyboard.instantiateViewControllerWithIdentifier("ViewController")
vc.beginAppearanceTransition(true, animated: false)
vc.endAppearanceTransition()
{% endhighlight %}

![image-3][img-3]

Great! This flow is more similar to the first one. You can see there is no window
here so the view of view controller is not added to the flow so no auto layout
configuration code is running. Just view controller configuration.

### Conclusion
You can see the loading view flow depends of in which context view controller
and view are used.

In the first case there was a view and view controller was the root one so it
has a lot of configuration and view was configured too.

In the last case there is just view controller configuration since there is no
window and configuring view is not necessary. This approach also simulated
presenting view controller so `viewWillAppear` and `viewDidAppear` method have
been called - This might be important in some cases.

And the last case I would mention is the second one. I didn't know earlier
that it behaves like this. I would expect that viewWill/viewDid method are
called too, but they didn't. IMO the approach presented by [Ørta][orta-twitter]
is the best one when testing view controllers.

[natasha-twitter]: http://twitter.com/NatashaTheRobot
[natasha-post]: http://natashatherobot.com/ios-testing-view-controllers-swift/
[orta-twitter]: twitter.com/orta
[wwdc-407]: https://developer.apple.com/videos/wwdc/2015/?id=407
[artsy-github]: https://github.com/artsy/eigen/blob/master/Artsy_Tests/Extensions/UIViewController+PresentWithFrame.m#L20-L22

[img-1]: /uploads/{{page.post_id}}/1.png
[img-2]: /uploads/{{page.post_id}}/2.png
[img-3]: /uploads/{{page.post_id}}/3.png
