---
layout: post
title: "iOS: Passing data back using unwind segue instead of delegate pattern"
tags: ios, swift
id: post-42
excerpt: "Nice way to pass data back without using delegate pattern."
redirect_from: "/ios-passing-data-back-using-unwind-segue-instead-of-delegate-pattern/"
---
This is just a little thing that I noticed today. I thought it might be worth
to share with you.

Usually when I created a view controller that is used as a picker, it shows up
from the bottom of a screen over the current context, and covers just a part of
the screen. After a value is picked, it is passed back to a view controller that
presented the picker using delegate pattern. It looked a bit like the following code.

{% highlight swift %}
class ViewController: UIViewController, AnimalPickerViewControllerDelegate {

    @IBOutlet var label: UILabel!

    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        if segue.identifier == "ShowAnimalPicker" {
            let pickerVC = segue.destinationViewController as! AnimalPickerViewController
            pickerVC.delegate = self
        }
    }

    func animalPicker(picker: AnimalPickerViewController, didSelectAnimal animal: String) {
        label.text = animal
    }
}
{% endhighlight %}

{% highlight swift %}
protocol AnimalPickerViewControllerDelegate: class {
    func animalPicker(picker: AnimalPickerViewController, didSelectAnimal animal: String)
}

class AnimalPickerViewController: UIViewController {

    weak var delegate: AnimalPickerViewControllerDelegate?

    @IBAction func dogButtonPressed(sender: AnyObject) {
        selectAnimal("Dog")
    }

    @IBAction func catButtonPressed(sender: AnyObject) {
        selectAnimal("Cat")
    }

    @IBAction func snakeButtonPressed(sender: AnyObject) {
        selectAnimal("Snake")
    }

    private func selectAnimal(animal: String) {
        delegate?.animalPicker(self, didSelectAnimal: animal)
        dismissViewControllerAnimated(true, completion: nil)
    }
}
{% endhighlight %}

So, as you can see I used a segue to present the picker, but I am using delegate
pattern to get the selected data back.

Today I learned that in such case unwind segue fits better. I do not have to use
delegate pattern. And this is how it look like.

{% highlight swift %}
class ViewController: UIViewController {

    @IBOutlet var label: UILabel!

    @IBAction func performUnwindSegue(segue: UIStoryboardSegue) {
        if segue.identifier == AnimalPickerViewController.UnwindSegue {
            label.text = (segue.sourceViewController as! AnimalPickerViewController).selectedAnimal
        }
    }
}
{% endhighlight %}

{% highlight swift %}
class AnimalPickerViewController: UIViewController {
    static let UnwindSegue = "UnwindAnimalPicker"

    private(set) var selectedAnimal: String!

    @IBAction func dogButtonPressed(sender: AnyObject) {
        selectAnimal("Dog")
    }

    @IBAction func catButtonPressed(sender: AnyObject) {
        selectAnimal("Cat")
    }

    @IBAction func snakeButtonPressed(sender: AnyObject) {
        selectAnimal("Snake")
    }

    private func selectAnimal(animal: String) {
        selectedAnimal = animal
        performSegueWithIdentifier(AnimalPickerViewController.UnwindSegue, sender: nil)
    }
}
{% endhighlight %}

Is it better? For me, in this specific case - YES.

I hope you find it useful :)
