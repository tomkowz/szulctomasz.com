---
layout: post
title: "iOS: Prototyping button-like control with nice animations"
tags: ios, custom control, uicontrol, animations
id: post-50
excerpt: "Again inspired by work on dribbble.com - Building control on top of UIControl"
---

I browsed [dribbble.com](http://dribble.com) again and found cool UI control. *Unfortunately I cannot find the resource again on dribbble* -
It just disappeared and I do not have any link to it. Anyway, here is the final effect very similar to what I
found.

**Operation succeeded and failed**

![final-effect-2][gif-2] ![final-effect-3][gif-3]

**Touching down and disabling/enabling**

![final-effect-1][gif-1] ![final-effect-4][gif-4]

Looking nice, ha? You can find the project on [tomkowz/transitioning-button-demo](https://github.com/tomkowz/transitioning-button-demo).

### How does it work?
The logic behind the control is easy to understand. The button has 4 states.

- **Begin** - Blue button with just *Submit* title. You can push it down and it
will nicely be pushed deeper and colors will be inverted
- **Loading** - After button is pushed down and then released it gets back to the
previous appearance and animates to show a spinner. This is good time to do
e.g. network requests, submitting a form, etc.. - Please notice how button dimension
is changing to create circle with the spinner - This is why I like it.
- **Finish with success** - After you did your work and everything finished with
success, you can change to this state to show some success message and style
the button little bit differently. Here I used darker blue.
- **Finish with failure** - If something went wrong you can style the button to
indicate that operation failed. A bit later state of the button goes back to
*Begin* so you can tap it again and re-try an operation.

### View hierarchy
Not sure whether this is designed in an optimal way or not. Probably something
could be simplified. If you find such thing, I'd be glad to know it.

![schema][schema]

Schema explained below.

- **1** - Main view for the control.
- **2** - I called this *placeholder view* but this is not the correct name. This view
have constraint that specifies width of the control and do not have any other
subview in it. Every subview is added to **1**. This view is hidden.
- **3**, **4**, **5** - Each of these represent another state of the control.
The first one is label presented for **Begin** state, the second contains spinner
and the third contains success/failure message. Each of these has also the same
height as the **1** - It ensure that subviews of **3 - 5** are correctly centered
both vertically and horizontally while presented on screen. Each of these are horizontally
centered in **1**.
- **5** - This view does not have constraint to the bottom of the **1**.
- **6** - This is constraint between top of the **1** and top of the **3**.
It allows to easily scroll subviews during state changes.
- **7** - As mentioned above, this is a width constraint of the control.

### Implementation

The control has 8 properties that define colors of texts and background
for a specific appearance style - *Normal*, *Pressed*, *Disabled*, *Success*, *Failure*.

When state changes view appearance is changing.

{% highlight swift %}
class TransitioningButton: UIControl {

    enum State {
        case Begin
        case Loading
        case FinishWithSuccess
        case FinishWithFailure
    }

    @IBInspectable var normalBackgroundColor: UIColor?
    @IBInspectable var pressedBackgroundColor: UIColor?
    @IBInspectable var disabledBackgroundColor: UIColor?
    @IBInspectable var disabledTextColor: UIColor?
    @IBInspectable var successBackgroundColor: UIColor?
    @IBInspectable var successTextColor: UIColor?
    @IBInspectable var failureBackgroundColor: UIColor?
    @IBInspectable var failureTextColor: UIColor?

    @IBOutlet var firstLabel: UILabel!
    @IBOutlet var activityIndicator: UIActivityIndicatorView!
    @IBOutlet var secondLabel: UILabel!

    @IBOutlet private var containerBegin: UIView!
    @IBOutlet private var containerLoading: UIView!
    @IBOutlet private var containerFinish: UIView!

    @IBOutlet private var topConstraint: NSLayoutConstraint!
    @IBOutlet private var widthConstriant: NSLayoutConstraint!

    var buttonState: State = .Begin {
        didSet {
            updateAfterButtonStateChanged(self.buttonState)
        }
    }    
}
{% endhighlight %}

Added listeners for *TouchUpInside*, *TouchUpOutside*, *TouchDown* control events,
so style of tapping/pressing button and releasing it can be easily updated.

{% highlight swift %}
private func configureActions() {
    self.addTarget(self, action: #selector(touchUpInside), forControlEvents: .TouchUpInside)
    self.addTarget(self, action: #selector(touchUpOutside), forControlEvents: .TouchUpOutside)
    self.addTarget(self, action: #selector(touchDown), forControlEvents: .TouchDown)
}

...

private extension TransitioningButton {
    @objc private func touchUpInside() {
        pushOut()
    }

    @objc private func touchUpOutside() {
        pushOut()
    }

    @objc private func touchDown() {
        pushIn()
    }

    private func pushIn() {
        let pushAnimation = CABasicAnimation(keyPath: "transform.scale")
        pushAnimation.duration = 0.1
        pushAnimation.fromValue = 1
        pushAnimation.toValue = 0.95
        pushAnimation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseOut)
        pushAnimation.removedOnCompletion = false
        pushAnimation.fillMode = kCAFillModeForwards
        self.layer.addAnimation(pushAnimation, forKey: nil)

        updateAppearanceToPressed()
    }

    private func pushOut() {
        updateAppearanceToNormal()

        let pushAnimation = CABasicAnimation(keyPath: "transform.scale")
        pushAnimation.duration = 0.1
        pushAnimation.fromValue = 0.95
        pushAnimation.toValue = 1
        pushAnimation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseOut)
        pushAnimation.removedOnCompletion = false
        pushAnimation.fillMode = kCAFillModeForwards
        self.layer.addAnimation(pushAnimation, forKey: nil)
    }
}

{% endhighlight %}

Here are two animations that changes scale of a button, so it imitate pushing it and
creates some depth on the screen.

The next important thing is a method which is responsible for animating state
changes.

{% highlight swift %}
private func updateAfterButtonStateChanged(state: State) {
    // Get container view that will be presented for new state
    let containerView = containerViewForState(state)

    /*
     As the content might change in the same autolayout pass as this animation
     We have to make sure that UI is ready to be presented.

     This is especially important when you're about to present short or long
     text in case when previous text was oposite, meaning, you had short
    text and not it is long or vice-versa.

    Without doing this additional pass you'd see that text is moving up
    and left or right, depending whether new text it is longer or shorter.
     */
    let loopUntil = NSDate(timeIntervalSinceNow: 0)
    NSRunLoop.currentRunLoop().runMode(NSDefaultRunLoopMode, beforeDate: loopUntil)

    // Calculate point to where scroll view should offset its content
    let scrollToY = offsetForState(state)

    // Decide whether button should react on touches...
    userInteractionEnabled = state == .Begin

    // Set new width for new state
    self.widthConstriant.constant =
        max(self.bounds.height, containerView.subviews.first!.bounds.width +
            (self.bounds.height / 2.0))

    // And animate the change...
    UIView.animateWithDuration(0.2, delay: 0, options: .CurveEaseOut, animations: {
        self.topConstraint.constant = -scrollToY;
        self.layoutIfNeeded()

        // Update style of the control
        if state == .Begin {
            if self.enabled {
                self.updateAppearanceToNormal()
            } else {
                self.updateAppearanceToDisabled()
            }

        } else if state == .FinishWithSuccess {
            self.updateAppearanceToSuccess()
        } else if state == .FinishWithFailure {
            self.updateAppearanceToFailure()
        }

        }, completion: { _ in
            // If an operation performed by this control failed, it will
            // show the control in the .Begin state.
            if state == .FinishWithFailure {
                let time = dispatch_time(DISPATCH_TIME_NOW, Int64(1 * Double(NSEC_PER_SEC)))
                dispatch_after(time, dispatch_get_main_queue(), {
                    let transition = CATransition()
                    transition.type = kCATransitionFade
                    transition.duration = 0.3

                    self.layer.addAnimation(transition, forKey: kCATransition)

                    self.updateAfterButtonStateChanged(.Begin)
                })
            }
    })
}
{% endhighlight %}

Let's take a look what's going on here.

At the beginning we need to obtain container with a label/spinner that will
be presented after state did change.

The next two lines are interesting.
{% highlight swift %}
let loopUntil = NSDate(timeIntervalSinceNow: 0)
NSRunLoop.currentRunLoop().runMode(NSDefaultRunLoopMode, beforeDate: loopUntil)
{% endhighlight %}

I noticed this:

Changing text and animating entire control (scrolling its content) caused that
label (for *Final* state) was going up-left or up-right depending whether text
was changed to long or short.

This was caused because container with label was updating its layout in the same
pass as entire control was shifting its subviews.

Adding these two lines forces another quick pass on the current loop (main thread)
so UI gets refreshed and label is correctly positioned. In effect, when animating
change state, you can see that longer label is nicely animated keeping its centered
position.

This should better illustrate the issue. Notice how long label on right is
shifting while presenting.

![transition-correct][gif-5] ![transition-incorrect][gif-6]

Next thing is to set new width of the control. We need to take width of a subview
inside the container and add some margin. The margin is calculated to create
a circle when spinner is presented. Labels and spinners have the same leading
and trailing distances to the control leading and trailing edges.

{% highlight swift %}
self.widthConstriant.constant =
    max(self.bounds.height, containerView.subviews.first!.bounds.width +
        (self.bounds.height / 2.0))
{% endhighlight %}

Next lines of code just changes color of the labels and the background, so
nothing interesting.

In case when new state is *Finish with failure* there is *CATransition* animation
added to the control's *layer*, so it nicely animates state change to *Begin* with
a fade effect.

{% highlight swift %}
let transition = CATransition()
transition.type = kCATransitionFade
transition.duration = 0.3

self.layer.addAnimation(transition, forKey: kCATransition)

self.updateAfterButtonStateChanged(.Begin)
{% endhighlight %}



[gif-1]: /uploads/{{page.id}}/gif-1.gif
[gif-2]: /uploads/{{page.id}}/gif-2.gif
[gif-3]: /uploads/{{page.id}}/gif-3.gif
[gif-4]: /uploads/{{page.id}}/gif-4.gif
[gif-5]: /uploads/{{page.id}}/gif-5.gif
[gif-6]: /uploads/{{page.id}}/gif-6.gif
[schema]: /uploads/{{page.id}}/schema.png
