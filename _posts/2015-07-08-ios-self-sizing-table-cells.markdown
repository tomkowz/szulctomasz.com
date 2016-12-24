---
layout: post
title: "iOS: Self-sizing table cells"
tags: ios, swift, wwdc, uitableview
id: post-12
redirect_from: "/ios-self-sizing-table-cells/"
---
Watched WWDC 2015 session about Auto Layout - [Mysteries of Auto Layout, Part 1][wwdc-218] and encountered issues during testing what Kasia Wawer presented there.

If you plan to watch the first one, I would also recommend to watch [Mysteries of Auto Layout, Part 2][wwdc-219].

There was a demo about self-sizing cells. I didn't used them before but read
about them and wanted to use self-sizing cells in [Quotes][quotes] app.
The concept was pretty easy - there are some elements inside cell and goal is
to make cell at least high as text is in contained `UILabel`. So what is needed
is to create constraints between top of the label and **Top Space To Container Margin**, the same with bottom of the label and **Bottom Space To Container Margin**.
The next thing was to set some estimated high of table view rows - just one line call:

{% highlight swift %}
tableView.estimatedRowHeight = 40
{% endhighlight %}

Let's run our app... Hmm, not working. Why?

![image-1][img-1]

This has been omitted by presented and it took me a while when I found the reason
of the issue. There is another property to set.

{% highlight swift %}
tableView.rowHeight = UITableViewAutomaticDimension
{% endhighlight %}

Let's run app again... It works. This works on iOS 7, 8 and 9.

![image-2][img-2]


[wwdc-218]: https://developer.apple.com/videos/wwdc/2015/?id=218
[wwdc-219]: https://developer.apple.com/videos/wwdc/2015/?id=219
[quotes]: https://github.com/tomkowz/Quotes

[img-1]: /uploads/{{page.id}}/1.png
[img-2]: /uploads/{{page.id}}/2.png
