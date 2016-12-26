---
layout: post
title: "iOS: Strings with length variations"
tags: ios, ios9, swift
post_id: post-19
redirect_from: "/ios-strings-with-length-variations/"
---
iOS 9.0 SDK provides new way to select and display proper localized string
variation depending on screen width.

Apps run on different screen sizes and different orientations. Apps present
different texts, sometimes longer that not fit a screen width and sometimes you
want to display text that perfectly fit in just one line. There is new method
which helps us with this. It uses **stringsdict** file and new `NSStringVariableWidthRuleType` key.

### Example
Let's say you want to display *"Forgotten Your Password? Get password Help."*
label with touchable *"Help"* word. On some screens and layouts it might perfectly
fit, but on some smaller it might be too long. In that case you would like to
display *"Forgot password? Get password Help."* which is a bit shorter. And let's
pretend there is another version of screen which is a bit shorter than previous
(e.g. Apple Watch) and then you would like to display the shortest one:
*"Forgot password? Help."*.

### Documentation
> For strings with length variations, such as from a stringsdict file, this
method returns the variant at the given width. If there is no variant at the
given width, the one for the next smaller width is returned. And if there are
none smaller, the smallest available is returned. For strings without variations,
this method returns self. The unit that width is expressed in is decided by the application or framework. But it is intended to be some measurement indicative
of the context a string would fit best to avoid truncation and wasted space.


### Usage
What you need to do is to create **stringsdict** file and add dictionary like this:

{% highlight xml %}
<key>Forgot Password</key>
<dict>
    <key>NSStringVariableWidthRuleType</key>
    <dict>
        <key>100</key>
        <string>Forgot Password? Help.</string>
        <key>200</key>
        <string>Forgot password? Get password Help.</string>
        <key>300</key>
        <string>Forgotten Your Password? Get password Help.</string>
    </dict>
</dict>
{% endhighlight %}


Then, to get one variant of these above you must call `NSLocalizedString()`
with key "Forgot Password" and next `variantFittingPresentationWidth`() method
to get variant for specified width. The example shows how to get string that
fits 100px width.

{% highlight swift %}
let localized: NSString = NSLocalizedString("Forgot Password", comment: "")
label.text = localized.variantFittingPresentationWidth(100)
{% endhighlight %}


### Wrap-up
This addition is nice, it will simplify a bit code where there is necessity to
deal with such strings. I don't like a bit that the new API is available only
for `NSString`, because `NSLocalizedString()` returns `String` so you have to
do extra cast.
