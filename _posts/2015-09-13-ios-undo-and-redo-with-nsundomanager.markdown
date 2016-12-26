---
layout: post
title: "iOS: Undo and redo with NSUndoManager"
tags: ios, ios9, nsundomanager, swift
post_id: post-35
redirect_from: "/ios-undo-and-redo-with-nsundomanager/"
---
`NSUndoManager` was mysterious thing to me for a really long time. I wanted to
learn using it, and actually never got a time. To date. I wrote a simple app
that allows to create rectangles that can be moved around and change their
properties like background color or corner radius.

You can find the demo app here on the github: [tomkowz/undo-manager-practice][gh-repo].
And here is a short video which presents [how the demo app works][yt-demo].

![image-1][img-1]

### NSUndoManager

The `NSUndoManager` allows to record operations performed by user and reverse
effect of such operations.

When you invoke some method that change something or you perform action that
changes property value by e.g. set accessor you can register an operation
that can reverse the action.

An undo operation consists of object that is supposed to receive a message,
the message to send and the arguments - generally you want to pass original value.

The undo manager supports redo actions, so actions are reversible. You can think
of this manager as of manager that manages two stacks. Actually, it manages two
stacks, undo stack and redo stack - `_undoStack` and `_redoStack` are private
properties of `NSUndoManagers` where operations are stored.

When an undo operation is registered it is added to the undo stack. When `undo()`
is called manager is undoing the operation, the recorded operation is performed
and it moves to redo stack, so you can redo it. When you register few undo
operations you can undo them and redo them in backward direction. You don’t
want to register operations directly onto redo stack - it is even impossible.

You can set a level of undo operations, meaning, how many undo operations a
manager should store on its stack. If after adding operation the level exceeds,
the oldest operations is removed from the stack.

You can check states of undo and redo stacks by `canUndo` and `canRedo` properties.
Those states are important because you might want to update UI based on stacks status.

Another case when these accessors are useful is when you have set the level of undo
operations and there is one on redo stack and limit exceeds. What `NSUndoManager`
will do in this case is to remove the redo operation because it was the latest
undo operation in the history of operations.

### Registering undo operations
API allows to register operation in two ways.
The first is a simple undo with `registerUndoWithTarget(_:selector:object:)`:

{% highlight swift %}
func registerUndoAddFigure(figure: FigureView) {
    undoManager.registerUndoWithTarget(self, selector: Selector(“removeFigure:”), object: figure)
    undoManager.setActionName(“Add Figure”)
}
{% endhighlight %}

The second type of undo is undo based on `NSInvocation`. You can register such
operation with `prepareWithInvocationTarget(_:)` method.

{% highlight swift %}
func registerUndoAddFigure(figure: FigureView) {
    undoManager.prepareWithInvocationTarget(self).removeFigure(figure)
    undoManager.setActionName("Add Figure")
}
{% endhighlight %}

You will get a `NSUndoManagerProxy` object in return and you can call on it
whatever method you want (but call only those that target conforms to,
  otherwise app throws runtime exception). It will record it by creating
  `NSInvocation` internally which will be called during undoing operation on
  passed target.

The important thing is that during registration the target isn’t retained.
You are responsible for managing it. If such undo method is called and the target
is deallocated you will get runtime exception.

In a time when object is about to be deallocated you’re responsible to call `removeAllActionsWithTarget(_:)`
to remove actions associated with specific target, or `removeAllActions()` to
remove all the operations from both undo and redo stacks.

### Grouping actions
Grouping operations is useful thing. Operations are grouped by events by default.
It means that operations are grouped around each pass of the run loop.
You can turn it off and call `beginUndoGrouping()` and `endUndoGrouping()`
methods manually.

### Naming actions and displaying them
`NSUndoManager` has support for storing names of actions. You can set a name
for an action by `setActionName(_:)` method. The manager comes with localization
of "Undo" and "Redo" words. There is even nice API to get ready to use strings
that conists of “Undo/Redo”.

Here is a method I created for the demo app that updates Undo and Redo
buttons after every new undo action is registered or undo/redo action performed.

{% highlight swift %}
private func updateUndoAndRedoButtons() {
    undoButton.enabled = undoManager.canUndo == true
    if undoManager.canUndo {
        undoButton.setTitle(undoManager.undoMenuTitleForUndoActionName(undoManager.undoActionName), forState: .Normal)
    } else {
        undoButton.setTitle(undoManager.undoMenuItemTitle, forState: .Normal)
    }

    redoButton.enabled = undoManager.canRedo == true
    if undoManager.canRedo {
        redoButton.setTitle(undoManager.redoMenuTitleForUndoActionName(undoManager.redoActionName), forState: .Normal)
    } else {
        redoButton.setTitle(undoManager.redoMenuItemTitle, forState: .Normal)
    }
}
{% endhighlight %}

### Notifications
Manager consists of couple notification types you can observe. In the demo app
I was focused on `NSUndoManagerDidUndoChangeNotification` and `NSUndoManagerDidRedoChangeNotification`.
To let my app works perfectly I should probably observe all the will/did notifications
because operations might take a while, some part of code might be asynchronous, etc.,
and the app should display UI properly in such cases. I used those notifications
to refresh Undo/Redo buttons.

### Context
Apps can have multiple managers to be used in multiple contexts. The demo app
uses two managers in two different contexts.

The first context is a board. The board is used to display rectangle on it and
to move rectangles around. The possible operations on the board context are to
add, move or remove a rectangle. The remove operation is accessible only as an
“Undo Add Rectangle” operation.

The second context is a context of rectangle itself. You can change its color
or corner radius. I decided to keep track of background color and corner radius
regardless of position of a rectangle on the board.

In effect you can add a rectangle, move it, change its color and corner radius
and undo moving operation without rolling back background color and corner
radius. Number of contexts you should use depends on how you want your app to behave.

### Responder Chain
Every `UIView` object inherits from `UIResponder` which defines interface for
objects that responds to and handles events.

`UIResponder` class declares `undoManager` property. When the application
receives undo event, `UIResponder` goes up the responder chain looking for a
responder that returns an `NSUndoManager` object from undoManager. The first
that is found is used for the undo or redo operation.

To make use of a responder chain you need to override `canBecomeFirstResponder()`
property and return `true`, and make the object that owns undo manager a first
responder by calling `becomeFirstResponder()`. When you have things correctly
configured you can perform a shake gesture and the app should display alert
asking if you want to perform undo operation.

![image-2][img-2]

### Example code
When I worked on the example I noticed that I spent more time thinking about
one-purpose methods. It is very important when you want to support undo and
redo actions because you want to do only specific things calling such methods.

Here is a simple code from the demo that shows how adding, removing and moving
figures on the board is implemented. Here is all the code that works with undo
manager.


{% highlight swift %}
/// MARK: Actions on Figures
func addFigure(figure: FigureView) {
    registerUndoAddFigure(figure)

    boardView.addSubview(figure)
    figures.append(figure)

    updateUndoAndRedoButtons()
}

func removeFigure(figure: FigureView) {
    registerUndoRemoveFigure(figure)

    figure.removeFromSuperview()
    if let index = figures.indexOf(figure) {
        figures.removeAtIndex(index)
    }
}

func moveFigure(figure: FigureView, center: CGPoint) {
    registerUndoMoveFigure(figure)
    figure.center = center
}

/// MARK: Undo Manager
override func canBecomeFirstResponder() -> Bool {
    return true
}

private var _undoManager = NSUndoManager()
override var undoManager: NSUndoManager {
    return _undoManager
}

private func observeUndoManager() {
    NSNotificationCenter.defaultCenter().addObserver(self, selector: Selector("updateUndoAndRedoButtons"), name: NSUndoManagerDidUndoChangeNotification, object: undoManager)
    NSNotificationCenter.defaultCenter().addObserver(self, selector: Selector("updateUndoAndRedoButtons"), name: NSUndoManagerDidRedoChangeNotification, object: undoManager)
}

@objc private func updateUndoAndRedoButtons() {
    undoButton.enabled = undoManager.canUndo == true
    if undoManager.canUndo {
        undoButton.setTitle(undoManager.undoMenuTitleForUndoActionName(undoManager.undoActionName), forState: .Normal)
    } else {
        undoButton.setTitle(undoManager.undoMenuItemTitle, forState: .Normal)
    }

    redoButton.enabled = undoManager.canRedo == true
    if undoManager.canRedo {
        redoButton.setTitle(undoManager.redoMenuTitleForUndoActionName(undoManager.redoActionName), forState: .Normal)
    } else {
        redoButton.setTitle(undoManager.redoMenuItemTitle, forState: .Normal)
    }
}

/// MARK: Undo Manager Actions
func registerUndoAddFigure(figure: FigureView) {
    undoManager.prepareWithInvocationTarget(self).removeFigure(figure)
    undoManager.setActionName("Add Figure")
}

func registerUndoRemoveFigure(figure: FigureView) {
    undoManager.prepareWithInvocationTarget(self).addFigure(figure)
    undoManager.setActionName("Remove Figure")
}

func registerUndoMoveFigure(figure: FigureView) {
    undoManager.prepareWithInvocationTarget(self).moveFigure(figure, center: figure.center)
    undoManager.setActionName("Move to \(figure.center)")
}
{% endhighlight %}

I decided to create simple methods for registering undo operations.
I noticed they work nice and logic of registering undo operation is better
separated from the operation logic. Just a simple call with some parameters.

I decided also to drop `registerUndoWithTarget(_:selector:object:)` method because
it based on `Selector` passed with a string which is dangerous. `prepareWithInvocationTarget(_:)`
looks a lot better, safer and easier to use.

Although, you may need the method which takes `Selector` when you want to set
some property.

You have to call directly a method you want to record. You'll not be able to set
value of some property this way. Two ways here: The first is to expose some `setPropertyName(_:)`
method, or use `registerUndoWithTarget(_:selector:object:)` method and pass `Selector("setPropertyName:")`
as one of parameters.

### Conclusion
`NSUndoManager` is a powerful mechanism that adds undo and redo functionality
to an app in not difficult way. It requires a bit more careful way of designing
app architecture, because you have to think about one-purpose methods that you
can undo or redo as a user's actions, but overall this is even better for apps,
right? The code is better designed.

[gh-repo]: https://github.com/tomkowz/undo-manager-practice
[yt-demo]: https://youtu.be/3Pk85X8bugk

[img-1]: /uploads/{{page.post_id}}/1.png
[img-2]: /uploads/{{page.post_id}}/2.png
