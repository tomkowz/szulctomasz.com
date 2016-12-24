---
layout: post
title: "iOS: Simplifying view controller's code and a view hierarchy in storyboard"
tags: ios, swift
id: post-3
redirect_from: "/simplifying-a-view-controllers-code-and-a-storyboard/"
---
I'll show a way how to create custom controls/components and how to reduce the
code inside view controller classes. This is a thing I learned few months ago,
and I'm using it daily when dealing with custom controls.

In this article we'll build a simple control to show how to deal with custom
controls. The control is a view which contains a text field and a button.
In the textfield user types a message and the button is for sending this
message. After message is sent the text field is cleared out and the button
is disabled. If there is no text inside the send button is disabled too.

This is how the control looks like in the app.
![image-1][img-1]

At the beginning we'll build it in the wrong way, meaning the entire code will
be placed in a view controller. So in storyboard I added one yellow container
view to keep the textfield and the button together.

![image-2][img-2]

After outlets are connected and code is added the view controller's code looks 
like following:

{% highlight swift %}
class ViewController: UIViewController, UITextFieldDelegate {
    @IBOutlet weak var textField: UITextField!
    @IBOutlet weak var actionButton: UIButton!

    override func viewDidLoad() {
        super.viewDidLoad()
        self.actionButton.enabled = false
    }

    @IBAction func onActionPressed(sender: AnyObject) {
        let success = self.sendMessage(self.textField.text)
        let message = success ? "Message sent" : "Message not send"
        UIAlertView(title: nil, message: message, delegate: nil, cancelButtonTitle: "Ok").show()

        if (success) {
            self.textField.text = ""
            self.validateActionButton()
        }
    }

    private func validateActionButton() {
        self.actionButton.enabled = countElements(self.textField.text) > 0
    }

    private func sendMessage(message: String) -> Bool {
        /// some logic here ...

        return true
    }

    @IBAction func textFieldDidChange(sender: AnyObject) {
        self.validateActionButton()
    }
}
{% endhighlight %}

This is so far only 33 lines of code, but if you plan to add some table view
code, some other code it'll blow up quickly. And you've got two outlets in the
view controller instead of one only to the component. That might be avoided by
moving the code to a custom control class and moving the views to a separated
.xib file and using a placeholder view inside the storyboard to keep created
constraints.

After the class is created and component's related code is moved to the
component's class the code inside the view controller looks like this:

{% highlight swift %}
class ViewController: UIViewController {
   /// outlet for the control may be here if used
}
{% endhighlight %}

You may ask: "how should/may I talk with controller from my custom control?"
The answer is: e.g. using blocks - or delegate pattern. You can expose some
blocks like: `messageSentBlock` or `editingFinishedBlock.``

Next thing is to moving the views to the separated xib. It looks like
this in my xib:

![image-3][img-3]

And this is a view controller in the storyboard:

![image-4][img-4]

You can see there is only the view of type `MessageControl`. This is a
placeholder for the control created above. The last thing is to load the
control and replace it with this placeholder. It can be done using 
`awakeAfterUsingCode:` method in `MessageControl`.

{% highlight swift %}
override func awakeAfterUsingCoder(aDecoder: NSCoder) -> AnyObject? {
    if self.subviews.count == 0 {
        let nib = UINib(nibName: "MessageControl", bundle: nil)
        let view = nib.instantiateWithOwner(nil, options: nil).first! as MessageControl
        view.setTranslatesAutoresizingMaskIntoConstraints(false)
        return view
    }

    return self
}
{% endhighlight %}

This method may be used to substitute some object with the object returned by
this method. So after you launch the app the control will appear and can be
used like normal.

### Conclusion

`awakeAfterUsingCoder:` method is a powerful one which can be used
to simplify view hierarchy in a storyboard and is helpful during refactoring
code inside view controllers. I'm using this approach in the new project and
so far I see only benefits - the code is simpler, the code is cleaner, more
readable, easier to maintenance and the control is more reusable because in
the storyboard there is only the placeholder for the control.

[img-1]: /uploads/{{page.id}}/1.png
[img-2]: /uploads/{{page.id}}/2.png
[img-3]: /uploads/{{page.id}}/3.png
[img-4]: /uploads/{{page.id}}/4.png
