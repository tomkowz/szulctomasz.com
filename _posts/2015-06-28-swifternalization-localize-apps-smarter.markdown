---
layout: post
title: "Swifternalization - localize apps smarter"
tags: ios, swift, open source, swifternalization
post_id: post-5
redirect_from: "/swifternalization-localize-apps-smarter/"
---
I am happy to annouce my new framework written in Swift I built this week,
Yay! As mentioned in previous post about [Test Driven Development][tdd-post]
I was working this week on framework that solves problem of app localization
and plural strings support in smarter way - And there it is - [Swifternalization][swifternalization].

It simplifies a lot process of localizing strings depending on numbers,
like e.g. "1 car", "2 cars", etc. This is easy in English but it might be tricky
in Polish. I don't want to copy-paste entire framework description from
[github page][swifternalization] - feel free to go there for implementation
details - I'll cover it a bit here.

Instead of creating localized strings like this below and coding all the logic in all places where you want to display proper localized string you can do it smarter...

*Localizable.strings (Base)*
{% highlight plain %}
"one-second" = "1 second ago";
"many-seconds" = "%d seconds ago";

"one-minute" = "1 minute ago";    
"many-minutes" = "%d minutes ago";

"one-hour" = "1 hour ago";
"many-hours" = "%d hours ago";
{% endhighlight %}

*Localizable.strings (Polish)*
{% highlight plain %}
"one-second" = "1 sekundę temu";
"few-seconds" = "%d sekundy temu";
"many-seconds" = "%d sekund temu";

"one-minute" = "1 minutę temu";
"few-minutes" = "%d minuty temu";
"many-minutes" = "%d minut temu";

"one-hours" = "1 hodzinę temu";
"few-hours" = "%d godziny temu";
"many-hours" = "%d godzin temu";
{% endhighlight %}


Logic for Polish localization:

- 0, (5 - 21), (25 - 31), ... - "few-seconds"
- 1 - "one-second"
- (2 - 4), (22-24), (32-34), (42, 44), ..., (162-164), ... - "many-seconds"


By smarter I meant using **expressions**. Like these:

{% highlight plain %}
"cars{ie:%d=1}" = "1 car";
"cars{ie:%d=0}" = "no cars";
"cars{ie:%d>1}" = "%d cars";
{% endhighlight %}

Or like this for tricky Polish ordering:

{% highlight plain %}
"police-cars{exp:^1$}" = "1 samochód policyjny";
"police-cars{exp:(((?!1).[2-4]{1})$)|(^[2-4]$)}" = "%d samochody policyjne";
"police-cars{exp:(.*(?=1).[0-9]$)|(^[05-9]$)|(.*(?!1).[0156789])}" = "%d samochodów policyjnych";
{% endhighlight %}

Expressions may be inequalities or regular expressions. To help you organize
it better there is functionality called "shared expressions". When you read
detail implementation on the github repo's page you'll learn that
`Localizable.strings` files could looks like this:

*Localizable.strings (Base)*
{% highlight plain %}
"time-seconds{one}" = "%d second ago";
"time-seconds{other}" = "%d seconds ago";

"time-minutes{one}" = "%d minute ago";
"time-minutes{other}" = "%d minutes ago";

"time-hours{one}" = "%d hour ago";
"time-hours{other}" = "%d hours ago";
{% endhighlight %}

*Localizable.strings (Polish)*
{% highlight plain %}
"time-seconds{one}" = "%d sekundę temu";
"time-seconds{few}" = "%d sekundy temu";
"time-seconds{many}" = "%d sekund temu";

"time-minutes{one}" = "%d minutę temu";
"time-minutes{few}" = "%d minuty temu";
"time-minutes{many}" = "%d minut temu";

"time-hours{one}" = "%d godzinę temu";
"time-hours{few}" = "%d godziny temu";
"time-hours{many}" = "%d godzin temu";
{% endhighlight %}

Nice, ha? And what is the best, all the logic is contained in Swifternalization
framework. You only have to pass some expressions. It works really nice with my
app that I am working on recently.

To get localized string do simple call:

{% highlight swift %}
// for regular key without expression
Swifternalization.localizedString("tomato")
// en: tomato, pl: pomidor

// for key with expression
Swifternalization.localizedExpressionString("time-seconds", value: 10)
// en: 10 seconds, pl: 10 sekund

// or with handy "I18n" typealias :)
I18n.localizedString("tomato")
I18n.localizedExpressionString("time-seconds", value: 10)
{% endhighlight %}

[Swifternalization][swifternalization] has also built-in expressions like:
*one, >one, two, other* (for every language) and *few* and *more* for Polish
which handles this tricky validation like in examples above. Framework is
easy to extend and I hope there will be more built-in expressions for other
countries too.

Swifternalization is ready for Swift 2, just checkout *swift2* branch.

I hope you find it useful as I did ;)

[tdd-post]: http://szulctomasz.com/test-driven-development-for-code-safety-and-your-peace/
[swifternalization]: https://github.com/tomkowz/Swifternalization
