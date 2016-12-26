---
layout: post
title: "Xcode: Trick shows how to create constraint to Top of UIViewController's view instead of Top To Layout Guide"
tags: ios, auto layout, xcode
post_id: post-33
redirect_from: "/constraint-to-top-instead-of-top-layout-guide/"
---
This is a pretty useful trick that my colleague [Piotr Tobolski][piotr-twitter]
shown me recently when I was struggling with Auto Layout (mainly cleaning it up).

When you press and hold Control and try to attach top of a subview to UIViewController's
view you can see you're able to create few types of them and there is a "Vertical
Spacing to Top Layout Guide" for example, but there is no a "Vertical Spacing to
Top" which you probably want to use when you need to place a map on the entire
view or you want to place some view filled with shadow (gradient from black to transparent)
from the top of the screen under status bar.

![image-1][img-1]

The same issue is when you try to create constraint to bottom, You can create only
"Vertical Space to Bottom Layout Guide" with such a simple drag - can't see
"Vertical Spacing to Bottom".

![image-2][img-2]

But hey, this is possible to create such a constraint. What you need to do is
to create e.g. "Center Vertically in Container" constraint, select it, and change
its items to "Superview.Top" which is UIViewController's view and another item
to "YourViewOnStoryboard.Top".

![image-3][img-3]

"Constraint to Bottom" can be created respectively by changing "Top" to "Bottom".

Thanks Piotr!

[piotr-twitter]: https://pl.linkedin.com/in/piotrtobolski

[img-1]: /uploads/{{page.post_id}}/1.png
[img-2]: /uploads/{{page.post_id}}/2.png
[img-3]: /uploads/{{page.post_id}}/3.png
