---
layout: post
title: "Swift: Why Float, Double and Int are AnyObjects?"
tags: ios, swift
id: post-36
redirect_from: "/swift-why-float-double-and-int-are-anyobjects/"
---
This is a [question][so-1] I asked yesterday, and copy-pasted my question details
and answer I get on StackOverflow topic just to have it here.

### The Question
I read inline documentation of Swift and I am bit confused.

1. `Any` is a protocol that all types implicitly conform.
2. `AnyObject` is a protocol to which all classes implicitly conform.
3. `Int`, `Float`, `Double` are structs

Here is a sample code:

{% highlight swift %}
import UIKit

func passAnyObject(param: AnyObject) {
    print(param)
}

class MyClass {}
struct MyStruct {}

let a: Int = 1
let b = 2.0
let c = NSObject()
let d = MyClass()
let e = MyStruct()

passAnyObject(a)
passAnyObject(b)
passAnyObject(c)
passAnyObject(d)
//passAnyObject(e) // Argument type 'MyStruct' does not conform to expected type 'AnyObject'


if a is AnyObject { // b, d, e is also AnyObject
    print("\(a.dynamicType) is AnyObject")
}
{% endhighlight %}

What I don't understand is why `Int`, `Double`, `Float` are `AnyObjects`?
Why compiler doesn't say anything? Those types are declared as structs.
Struct `MyStruct` cannot be passed to the method on the top because it does
not conform to AnyObject.

Could you help me understand why it(...)?

### The Answer

[vacawama][wacavama] from SO answered:

> Because you have Foundation imported, Int, Double, and Float get converted to
NSNumber when passed to a function taking an AnyObject. This is done to make life
easier when calling Cocoa and Cocoa Touch based interfaces. If you remove import
UIKit (or import Cocoa for OS X), you will see:
>    `error: argument type Int does not conform to expected type 'AnyObject'` when you call `passAnyObject(a)``


[so-1]: http://stackoverflow.com/q/32554357/1046965
[wacavama]: http://stackoverflow.com/users/1630618/vacawama
