---
layout: post
title: "Asynchronous NSBlockOperation"
tags: ios, swift, operation, async
post_id: post-54
redirect_from: "/async-nsblockoperation/"
---

Here is an useful [snippet] for asynchronous `NSBlockOperation`.

{% highlight swift %}
class AsyncBlockOperation: NSOperation {
    typealias Block = (Void -> Void) -> Void

    private let block: Block
    private var _executing = false
    private var _finished = false
    
    init(block: Block) {
        self.block = block
        super.init()
    }
    
    override func start() {
        guard (self.executing || self.cancelled) == false else { return }
        self.executing = true
        self.block(finish)
    }
    
    private func finish() {
        self.executing = false
        self.finished = true
    }
    
    override var asynchronous: Bool {
        return true
    }
    
    override var executing: Bool {
        get { return _executing }
        set {
            let key = "isExecuting"
            willChangeValueForKey(key)
            _executing = newValue
            didChangeValueForKey(key)
        }
    }
    
    override var finished: Bool {
        get { return _finished }
        set {
            let key = "isFinished"
            willChangeValueForKey(key)
            _finished = newValue
            didChangeValueForKey(key)
        }
    }
}

{% endhighlight %}

Usage:
{% highlight swift %}
let op = AsyncBlockOperation { completion in
    print("Szulc Tomasz")
    completion()
}
{% endhighlight %}

[snippet]: https://gist.github.com/tomkowz/2734cf25318b7cfcd475b1149ab3ee7a
