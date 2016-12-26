---
layout: post
title: "Strings with length variations in iOS 8 with Swifternalization"
tags: ios, swift, swifternalization
post_id: post-25
redirect_from: "/strings-with-length-variations-in-ios-8-0-using-swifternalization-1-2/"
---
iOS 9.0 provides new way to select and display proper localized string variation
depending on screen width. With [Swifternalization 1.2][swifternalization] you
can use the feature also in iOS 8.0.

*If you don't know how the feature works in iOS 9 you can read about it in previous post [iOS: Strings with length variations][previous-post].*

Today I've released Swifternalization 1.2 which has the strings length variations
functionality but you can use it in iOS 8.0 too - The framework works with JSON
files instead of *Localizable.strings* and avoids using *stringdicts* files.

Using the feature in a system manner you have to create *stringsdict* file for
you key and create a dictionary inside with key-value pairs and in the code you
have to call `variantFittingPresentationWidth:` method to get a proper string variant.

{% highlight xml %}
<em>Stringsdict</em> file:
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

And in the code:

{% highlight swift %}
let localized: NSString = NSLocalizedString("Forgot Password", comment: "")
label.text = localized.variantFittingPresentationWidth(100)
{% endhighlight %}

Doing this with Swifternalization you just need to extend/add key-value
translation pair in JSON file like below:
{% highlight json %}
"forgot-password": {
    "@100": "Forgot Password? Help.",
    "@200": "Forgot password? Get password Help.",
    "@300": "Forgotten Your Password? Get password Help."
}
{% endhighlight %}

And somewhere in code just call localizedString method and fill one of default
parameters.

{% highlight swift %}
label.text = I18n.localizedString("forgot-password", fittingWidth: 200)
{% endhighlight %}

Easy, isn't it? :)

[swifternalization]: https://github.com/tomkowz/swifternalization
[previous-post]: http://szulctomasz.com/ios-strings-with-length-variations/
