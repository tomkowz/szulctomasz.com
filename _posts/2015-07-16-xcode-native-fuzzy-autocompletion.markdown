---
layout: post
title: "Xcode: Native Fuzzy Autocompletion"
tags: xcode
id: post-18
redirect_from: "/xcode7-native-fuzzy-autocompletion/"
---
[@hamzasood][hamzasood] (Thanks!) mentioned about hidden feature in Xcode7 - Fuzzy Autocompletion.

Finally Apple added it to the Xcode. I think they will release it as new feature
in Xcode 8. You can turn it on manually if you want. Just quit Xcode first,
open terminal and paste:

{% highlight sh %}
defaults write com.apple.dt.Xcode IDECodeCompletionFuzzyMode 3
{% endhighlight %}

Open Xcode and type something, it works :)

![image-1][img-1]

**UPDATE:**

Setting this key does not work in Xcode 7 beta 4.

To make it work in the newest Xcode you have to set **IDEWorkaroundForRadar6288283**
key instead of **IDECodeCompletionFuzzyMode**.

{% highlight sh %}
defaults write com.apple.dt.Xcode IDEWorkaroundForRadar6288283 3
{% endhighlight %}

[hamzasood]: http://twitter.com/hamzasood

[img-1]: /uploads/{{page.id}}/1.png
