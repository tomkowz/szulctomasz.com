---
layout: post
title: "iOS: Initialization process of a view loaded from a storyboard"
tags: ios
post_id: post-32
redirect_from: "/ios-initialization-process-of-a-view-loaded-from-a-storyboard/"
---
This post is continuation of the
[UIViewController's view loading process demystified][prev-post-1] post I've wrote
some time ago. I'd like to cover how custom view is loaded when it is placed in
a storyboard.

There are several ways to add view to a view controller. You can put it in
storyboard or you can create it in code and add as subview of view controller's
view.

I sat down and spent some time on debugging, trying to maybe find something
useful and to learn about entire process of creating and presenting view.
And here are my findings.

### View without subviews loaded from a storyboard
Let's say there is a storyboard which has one view controller, and in the view
controller there is a view which is of some subclass of `UIControl`, and has
some constraints set, and has few `IBInspectable` properties defined so the
properties will be loaded and set after our view is loaded.

Here is how the flow look like:

![image-1][img-1]

I thought here will be some magic but didn't find any. There are two or three
private methods called between public ones but do nothing special.

At first `initWithCoder:` is called and next is `setNeedsDisplay`.

All constraints of this view are set by calling `addConstraints:`. Here are set
constraints that not determine position of a view in its superview. So, only like
height and width. I thought that all constraints are set here but makes more
sense that not all are set in here, because position of this view is important
for its superview. When view has subviews, constraints that define position of
subviews will be added to the view that contains these subviews.

Next called method is `awakeAfterUsingCoder:` and view is prepared to be added
to superview because `willMoveToSuperview:` method is called and the next one
is `didMoveToSuperview`.

After view is added to superview `awakeFromNib` is called. I thought that `awakeFromNib`
is called right after `awakeAfterUsingCoder` even before adding view to its superview,
but was wrong. `awakeFromNib` is important method. This is a moment when you've
got your `IBInspectable` properties and key-value pairs defined in a storyboard
set and ready to use. Those loaded values are available before super `awakeFromNib`
is called. `awakeFromNib` is called by `UINib`'s `instantiateWithOwner:options:`
method, so it loads the view, decodes and set properties and those are ready to be used.

Next `willMoveToWindow` and `didMoveToWindow` methods are called. I was a bit
confused why those are called (because view is added to its superview, not to a window directly)
but read documentation which says that default implementation of these
methods do nothing and this is useful if you want to do additional things
when view moves to another window.

When view is on screen the `updateConstraints:`, `layoutSublayersOfLayer:` and
`layoutSubviews:` methods are called. That's all for this case. Easy to remember.

### View with subviews loaded from a storyboard
The flow is bit different because there are few subviews to add and layout.

![image-2][img-2]

First difference is that after `initWithCoder:` method is called the next one
is adding subviews of this loaded view in `_addSubviews:positioned:relativeTo:` method.
Other things to time when `updateConstraints` method is called stays the same.
The second thing is, no matter how many views there are the `setNeedsLayout` method is
called twice and `layoutSublayersOfLayer:` is also called twice.

So... Yeah, I hope it helps in understanding the flow better.

[prev-post-1]: http://szulctomasz.com/ios-uiviewcontrollers-view-loading-process-demystified/

[img-1]: /uploads/{{page.post_id}}/1.png
[img-2]: /uploads/{{page.post_id}}/2.png
