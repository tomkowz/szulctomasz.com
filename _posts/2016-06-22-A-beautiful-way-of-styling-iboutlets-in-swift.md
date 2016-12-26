---
layout: post
title: "iOS: A Beautiful Way of Styling IBOutlets in Swift"
tags: ios, swift, iboutlets, storyboards
post_id: post-52
excerpt: "Trim Your View Controller With This IBOutlet Trick"
permalink: /ios-a-beautiful-way-of-styling-iboutlets-in-swift/
---

Today I was struggling with styling views. Usually I am trying to do what I am
able to do in storyboard files. Next, I am creating `@IBOutlet` references and
do the rest in the view controllers code - That's not ideal, I know.

Today's problem was a bit more complicated. I am using a lot of placeholder
views in storyboards that will hold more complicated custom controls. These type
of controls that have only placeholder reference in storyboard is very difficult
to fully customize in storyboards. I mean, you can use `@IBDesignable` but at
some point you'll be duplicating properties of the internal views. I am a big fan
of exposing views contained in this control and customize them outside - but not
about it this time.

So, I was thinking how can I setup control without messing around with the
view controller code and logic. Googling a bit took me to [iOS: A Beautiful Way of Styling IBOutlets in Swift](https://www.natashatherobot.com/ios-a-beautiful-way-of-styling-iboutlets-in-swift/) by [@NatashaTheRobot](http://twitter.com/natashatherobot) (Go there and read, I'll wait).

A code example taken from the original post:

{% highlight swift %}
import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var myLabel: UILabel! {
        didSet {
            myLabel.textColor = UIColor.purpleColor()
        }
    }

    @IBOutlet weak var myOtherLabel: UILabel! {
        didSet {
            myOtherLabel.textColor = UIColor.yellowColor()
        }
    }

    @IBOutlet weak var myButton: UIButton! {
        didSet {
            myButton.tintColor = UIColor.magentaColor()
        }
    }

    override func viewDidLoad() {
        super.viewDidLoad()
    }
}
{% endhighlight %}

The idea she described is kinda beautiful, and I adopted it. Then I realized I had
to do a lot of customization with my controls. I immediately stopped liking the solution
and started to hating it because I noticed the view controller is getting bigger
and bigger.

Styling outlets this way cost me *2 + n* lines of code, when *n* is number of
lines of styling code. You can also add this empty line between outlets just for
readability. Having 10 properties for styling is a lot of additional code and lines
we can probably somehow avoid.

*I am not saying this way is poor and you have to stop doing this way. That'd not be
true. I actually like it, but it is not the best way to go in case of my current project.*

### Another beautiful(?) way

Let's start with a bit simplified example. I'll be styling labels, a button and a view,
but you can think about more real-life example that I mentioned above - every control
is a custom one that you have to style somewhere - you can also call some DRY method
you created to avoid repeating styling-code - I hope you wrote such method :)

{% highlight swift %}
class ManyLabelsViewController: UIViewController {
    @IBOutlet private var label1: UILabel!
    @IBOutlet private var label2: UILabel!
    @IBOutlet private var label3: UILabel!
    @IBOutlet private var label4: UILabel!
    @IBOutlet private var label5: UILabel!
    @IBOutlet private var label6: UILabel!
    @IBOutlet private var button1: UIButton!
    @IBOutlet private var view1: UIView!

    override func viewDidLoad() {
        super.viewDidLoad()
        // something goes here...
    }
}
{% endhighlight %}

Let's add some styling.

{% highlight swift %}
import UIKit

class ManyLabelsViewController: UIViewController {
    @IBOutlet private var label1: UILabel! {
        didSet {
            label1.textColor = UIColor.redColor()
            label1.font = UIFont.systemFontOfSize(20)
            label1.backgroundColor = UIColor.blueColor()
        }
    }

    @IBOutlet private var label2: UILabel! {
        didSet {
            label2.layer.borderColor = UIColor.yellowColor().CGColor
            label2.layer.borderWidth = 1
            label2.backgroundColor = UIColor.blueColor()
            label2.clipsToBounds = true
        }
    }

    @IBOutlet private var label3: UILabel! {
        didSet {
            label3.textColor = UIColor.purpleColor()
        }
    }

    @IBOutlet private var label4: UILabel! {
        didSet {
            label4.textAlignment = .Center
            label4.textColor = UIColor.grayColor()
        }
    }

    @IBOutlet private var label5: UILabel! {
        didSet {
            label5.backgroundColor = UIColor.greenColor()
            label5.font = UIFont.boldSystemFontOfSize(28)
        }
    }

    @IBOutlet private var label6: UILabel! {
        didSet {
            label6.lineBreakMode = .ByClipping
            label6.numberOfLines = 0
        }
    }

    @IBOutlet private var button1: UIButton! {
        didSet {
            button1.layer.borderWidth = 1
            button1.layer.borderColor = UIColor.blueColor().CGColor
        }
    }

    @IBOutlet private var view1: UIView! {
        didSet {
            view1.backgroundColor = UIColor.cyanColor()
        }
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        // something goes here...
    }
}
{% endhighlight %}

Wow... This turned out to be 65 lines of code in the view controller and there
is no logic inside yet. It was 17 before we started. This does not sound like
beautiful styling at all. Can you see properties declarations? For sure yes,
that's clear and nicely visible.

It gets even worse when you have to create some outlets just for styling.
If there would be no custom styling you would not be creating such outlet.

*Can we do better than this? Yes. Is this an universal solution? Not sure, it works
in my case, and I get better results than the method described above.*

As we're using storyboards anyway, why don't we create some styling object
that would help as doing dirty work? We could create it by dragging **Object**
object on our view controller in storyboard, create class suffixed with `*ViewControllerStyle`
connect outlets there and do the styling. You can create reference to a specific
view/control in many objects, not just view controller.

![img1][img1]

Doing it this way, the code has been detached into `ManyLabelsViewControllerStyle` class.

{% highlight swift %}
class ManyLabelsViewControllerStyle: NSObject {

    @IBOutlet private weak var label1: UILabel!
    @IBOutlet private weak var label2: UILabel!
    @IBOutlet private weak var label3: UILabel!
    @IBOutlet private weak var label4: UILabel!
    @IBOutlet private weak var label5: UILabel!
    @IBOutlet private weak var label6: UILabel!
    @IBOutlet private weak var button1: UIButton!
    @IBOutlet private weak var view1: UIView!

    func style() {
        styleLabel1()
        styleLabel2()
        styleLabel3()
        styleLabel4()
        styleLabel5()
        styleLabel6()
        styleButton1()
        styleView1()
    }

    private func styleLabel1() {
        label1.textColor = UIColor.redColor()
        label1.font = UIFont.systemFontOfSize(20)
        label1.backgroundColor = UIColor.blueColor()
    }

    private func styleLabel2() {
        label2.layer.borderColor = UIColor.yellowColor().CGColor
        label2.layer.borderWidth = 1
        label2.backgroundColor = UIColor.blueColor()
        label2.clipsToBounds = true
    }

    private func styleLabel3() {
        label3.textColor = UIColor.purpleColor()
    }

    private func styleLabel4() {
        label4.textAlignment = .Center
        label4.textColor = UIColor.grayColor()
    }

    private func styleLabel5() {
        label5.backgroundColor = UIColor.greenColor()
        label5.font = UIFont.boldSystemFontOfSize(28)
    }

    private func styleLabel6() {
        label6.lineBreakMode = .ByClipping
        label6.numberOfLines = 0
    }

    private func styleButton1() {
        button1.layer.borderWidth = 1
        button1.layer.borderColor = UIColor.blueColor().CGColor
    }

    private func styleView1() {
        view1.backgroundColor = UIColor.cyanColor()
    }
}
{% endhighlight %}

As we don't need references to the controls in view controller we can remove all
of them. Or just keep these you need.

You need to call the `style()` method at some point, so the last thing is to
create outlet to such object in the view controller and call `style()` in the
`viewDidLoad`.

Here is how the view controller look like at the end.
{% highlight swift %}
class ManyLabelsViewController: UIViewController {

    // Controls you need to keep for calling them...

    @IBOutlet private var style: ManyLabelsViewControllerStyle!

    override func viewDidLoad() {
        super.viewDidLoad()
        style.style()
        // something goes here...
    }
}
{% endhighlight %}

---
**Updated 30 Jun 2016**

You can also use just *didSet* in this *styling* object and just let it configure
the outlets when they are set.

{% highlight swift %}
class ManyLabelsViewControllerStyle: NSObject {

    @IBOutlet private weak var label1: UILabel! {
      didSet {
        label1.textColor = UIColor.redColor()
        label1.font = UIFont.systemFontOfSize(20)
        label1.backgroundColor = UIColor.blueColor()
      }
    }

    @IBOutlet private weak var label2: UILabel! {
      didSet {
        label2.layer.borderColor = UIColor.yellowColor().CGColor
        label2.layer.borderWidth = 1
        label2.backgroundColor = UIColor.blueColor()
        label2.clipsToBounds = true
      }
    }

    @IBOutlet private weak var label3: UILabel! {
      didSet {
        label3.textColor = UIColor.purpleColor()
      }
    }

    @IBOutlet private weak var label4: UILabel! {
      didSet {
        label4.textAlignment = .Center
        label4.textColor = UIColor.grayColor()
      }
    }

    @IBOutlet private weak var label5: UILabel! {
      didSet {
        label5.backgroundColor = UIColor.greenColor()
        label5.font = UIFont.boldSystemFontOfSize(28)
      }
    }

    @IBOutlet private weak var label6: UILabel! {
      didSet {
        label6.lineBreakMode = .ByClipping
        label6.numberOfLines = 0
      }
    }
    @IBOutlet private weak var button1: UIButton! {
      didSet {
        button1.layer.borderWidth = 1
        button1.layer.borderColor = UIColor.blueColor().CGColor
      }
    }
    @IBOutlet private weak var view1: UIView! {
      didSet {
        view1.backgroundColor = UIColor.cyanColor()
      }
    }
}
{% endhighlight %}

And then your view controller look like this:
{% highlight swift %}
class ManyLabelsViewController: UIViewController {

    // Controls you need to keep for calling them...

    @IBOutlet private var style: ManyLabelsViewControllerStyle!
    // no extra call needed.
}
{% endhighlight %}


[img1]: /uploads/{{page.post_id}}/img1.png
