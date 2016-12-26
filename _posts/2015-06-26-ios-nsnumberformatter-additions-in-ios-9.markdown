---
layout: post
title: "iOS: NSNumberFormatter additions in iOS 9"
tags: ios, ios9, swift
post_id: post-4
redirect_from: "/nsnumberformatter-additions-in-ios-9/"
---
I've read changes in iOS 9 APIs and found interesting addition in
`NSNumberFormatter` class. There are additional `NSNumberFormatterStyle`
enum options to format numbers.

The new ones are:

- `OrdinalStyle` - used for converting numbers to ordinal equivalent
- `CurrencyISOCodeStyle` - used for display amount with currency code
- `CurrencyPluralStyle` - used for display amount with unit in plural form
- `CurrencyAccountingStyle` - used for display amount and unit in accounting style

Let's see how it works.
{% highlight swift %}
extension NSNumberFormatter {
    func format(style: NSNumberFormatterStyle) -> NSNumber -> String {
        self.numberStyle = style
        return { number in self.stringFromNumber(number)!}
    }
}

let formatter = NSNumberFormatter()
formatter.format(.OrdinalStyle)(1)
// US - 1st
// PL - 1.

formatter.format(.CurrencyISOCodeStyle)(9.99)
// US - USD9.99
// PL - 9,99 PLN

formatter.format(.CurrencyPluralStyle)(99)
// US - 99.00 US dollars
// PL - 99,00 Polish zlotys (english language)
// PL - 99,00 złotego polskiego (polish language)

formatter.format(.CurrencyAccountingStyle)(3.14)
// US - $3.14
// PL - 3,14 PLN (english language)
// PL - 3,14 zł (polish language)
{% endhighlight %}

These new enums (iOS 9+) play nicely with language and region set on the phone.
The last two examples shows how it looks like with English language and
Polish region and with both Polish language and region.
