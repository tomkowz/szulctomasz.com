---
layout: post
title: "Swift: Detach completion handler of a method to a function"
tags: swift
post_id: post-43
excerpt: "It makes your code a bit cleaner."
redirect_from: "/swift-detach-completion-handler-of-a-method-to-a-function/"
---
I think it is worth to share, I show it to my coworker and he likes it.

Sometimes we need to perform a big chunk of work to do in a completion handler
of a function, like completion handler in a get/post network request. Usually
we do it in a block that can get really messy. Here is a simplified example.

{% highlight swift %}
private func executeRequest(request: NSURLRequest, completionHandler: (success: Bool, elements: [String]) -> Void) {
    completionHandler(success: true, elements: ["A", "B", "C"])
}

private func prepare() {
    let request = NSURLRequest() // some request.
    executeRequest(request) { (success, elements) -> Void in
        if success == true {
            print("success")
            for element in elements {
                print(element)
            }
        } else {
            print("failure")
        }
    }
}
{% endhighlight %}

The other option you can handle completion block is to pass a function instead
of a block. As we know, closures are functions without names, or differently,
functions are closures with names :)

{% highlight swift %}
private func executeRequest(request: NSURLRequest, completionHandler: (success: Bool, elements: [String]) -> Void) {
    completionHandler(success: true, elements: ["A", "B", "C"])
}

private func prepare() {
    let request = NSURLRequest()
    executeRequest(request, completionHandler: processElements)
}

private func processElements(success: Bool, elements: [String]) {
    if success == true {
        print("success")
        for element in elements {
            print(element)
        }
    } else {
        print("failure")
    }
}
{% endhighlight %}
