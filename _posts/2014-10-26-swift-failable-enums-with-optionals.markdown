---
layout: post
title: "Swift: Failable Enums with Optionals"
tags: ios, swift
post_id: post-2
redirect_from: "/failable-enums-with-optionals/"
---

This is response to [Natasha Murashev's post][natasha-the-robot-post].
She asked if someone found some better way of handling initializing enums with
optional values. I think here is described a better one.

Natasha published few ways to do that:

{% highlight swift %}
enum SegueIdentifier: String {
    case SegueToRedViewIdentifier = "SegueToRedViewIdentifier"
    case SegueToGreenViewIdentifier = "SegueToGreenViewIdentifier"
    case SegueToBlueViewIdentifier = "SegueToBlueViewIdentifier"

    init?(optionalRawValue: String?) {
        if let value = optionalRawValue {
            switch value {
            case "SegueToRedViewIdentifier": self = .SegueToRedViewIdentifier
            case "SegueToGreenViewIdentifier": self = .SegueToGreenViewIdentifier
            case "SegueToBlueViewIdentifier": self = .SegueToBlueViewIdentifier
            default: return nil
            }
        }
        return nil
    }
}
{% endhighlight %}

This is using check if passed `rawValue` is `nil` and appropriate value is
assigned using switch to check all cases (IMO almost perfect), but what if
enum will have 10-15 values? This switch will escalates so quickly!

{% highlight swift %}
enum SegueIdentifier: String {
    case SegueToRedViewIdentifier = "SegueToRedViewIdentifier"
    case SegueToGreenViewIdentifier = "SegueToGreenViewIdentifier"
    case SegueToBlueViewIdentifier = "SegueToBlueViewIdentifier"
}

override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    if let segueIdentifier = segue.identifier {
        if let segueIdentifierValue =  SegueIdentifier(rawValue: segueIdentifier) {

            switch segueIdentifierValue {
            case .SegueToRedViewIdentifier:
                println("red")
            case .SegueToGreenViewIdentifier:
                println("green")
            case .SegueToBlueViewIdentifier:
                println("blue")
            }
        }
    }
}
{% endhighlight %}

This is using check if `rawValue` is `nil` outside enum which looks poor,
because everywhere enum is created with optional string this check must
be performed.

{% highlight swift %}
enum SegueIdentifier: String {
    case SegueToRedViewIdentifier = "SegueToRedViewIdentifier"
    case SegueToGreenViewIdentifier = "SegueToGreenViewIdentifier"
    case SegueToBlueViewIdentifier = "SegueToBlueViewIdentifier"

}

override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {

    if let segueIdentifier =  SegueIdentifier(
        rawValue: segue.identifier ?? "") {

        switch segueIdentifier {
        case .SegueToRedViewIdentifier:
            println("red")
        case .SegueToGreenViewIdentifier:
            println("green")
        case .SegueToBlueViewIdentifier:
            println("blue")
        }
    }
}
{% endhighlight %}

This has been described by Natasha as "not as clear / readable" because of
using `??` operator. And this check should be also performed inside some
enum initializer.

Here are two checks to be performed before enum is able to be created:
1. Check if rawValue passed to enum initializer is nil
2. Check if enum can be created with passed rawValue

I found that this is a little better option of handling initializing
enums with optional rawValues.

{% highlight swift %}  
enum SegueIdentifier: String {
    case ToRed = "Red"
    case ToGreen = "Green"
    case ToBlue = "Blue"

     init?(_ rawValue: String?) {
        /// check if rawValue is nil
        if rawValue == nil {
            return nil
        }

        /// check if enum value can be created with rawValue
        if let output = SegueIdentifier(rawValue: rawValue ?? "") {
            self = output
        } else {
            return nil
        }
    }
}

/// usage:
if let identifier = SegueIdentifier(segue.identifier) {
    switch identifier {
    case .ToRed: println("red")
    case .ToGreen: println("green")
    case .ToBlue: println("blue")
    }
}
{% endhighlight %}

All checks are performed and switch-case check used by Natasha is avoided
so even if enum has 100 values implementation is the same.

Going further it may be simplified using more complex if-statement:

{% highlight swift %}
enum SegueIdentifier: String {
        case ToRed = "Red"
        case ToGreen = "Green"
        case ToBlue = "Blue"

         init?(_ rawValue: String?) {
            let out = SegueIdentifier(rawValue: rawValue ?? "")
            if rawValue != nil &amp;&amp; out != nil {
                self = out!
            } else {
                return nil
            }
        }
    }
{% endhighlight %}


**Update**: In article at medium.com there is even better [solution][better-solution].

[natasha-the-robot-post]: http://natashatherobot.com/swift-failable-enums-with-optionals/
[better-solution]: https://medium.com/@tomkowz/failable-enums-with-optionals-83b7a2df3605
