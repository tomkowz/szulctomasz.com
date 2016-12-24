---
layout: post
title: "iOS: Second try to NSOperation and long running tasks"
tags: ios, nsoperation, swift
id: post-17
redirect_from: "/ios-second-try-to-nsoperation-and-long-running-tasks/"
---
Yesterday I wrote about [using NSOperation for background tasks][previous-post].
It was more about long running tasks. When I finished writing the code I wasn't
so happy about how it looked. There were few bad things that I didn't like.
If you didn't read the first version give it a try and next continue this one,
I'll wait :)

**If you're not familiar with [Quotes][quotes] yet, please read this [brief introduction][previous-post-2].**

I was also happy that [you were interested in this topic][twitter-post] and
this was the second thing that pushed me to do it better and share results
with you.

### The bad things
I found few bad things in operations I implemented:
1. Few operations were using semaphores! I'm not a big fun of using them,
I know how to use them, but when I see them in the code I think the code gets complicated. I want to avoid using them in operations.
2. Not all cases were handled. It worked in ideal world but it must improve.
3. `DownloadOperation` was mistake and `NetworkOperation` is better idea.
I wrote it in the previous post's update, but want to mention this here too:
I decided to keep decoding downloaded data out of `NetworkOperation` to have
ability to decode it depending on what I need in other tasks that depends on received data.

### Let's improve it
Spent a while reading docs for `NSOperation` and subclassing, and found few
points that changed my thinking about implementing custom operations.

`main()` method should be overridden for not-concurrent operations. Task starts, `main()` is called by `start()` method, `main()` ends and task finishes.

If operation is concurrent one, following methods should be overridden: `start()`, `asynchronous()`, `executing()`, `finished()`. And this is what I wanted from
the beginning. My operations are long running that hit backend and I wanted to
code them in this manner.

### Concurrent operations
In concurrent operation `start()` method is responsible for starting operation. Asynchronous methods should be called from this one. When subclassing you are
also responsible for changing `executing` and `finished` properties of the class.
What is important you must generate appropriate KVO notifications to let objects
that observe this operation react correctly. When KVO notification is not send
task will not be checked again after it started - so... yeah, it will never finish.

The next important thing is that `main()` may be called from `start()` or may
not be called at all. When overriding `start()` you're responsible for calling
`main()`.

Task also will never finish itself. You must change `finished` property (and accordingly `executing` property) and generate KVO notifications.

Important functionality that should be supported is cancellation. In subclass
you're responsible for handling cancellation. When `cancel()` is called on the
operation or `cancelAllOperations()` on a queue the `cancelled` property changes
and operation should correctly react when task is cancelled. Operation may not
be cancelled immediately but you should stop doing things as fast as possible.

You're responsible for checking cancellation:

1. In `start()` method before doing anything. If task is cancelled just break
what was planned to do.
2. In `main()` method during operation is in executing state. Stop doing all
the things.

When operation is cancelled you're responsible to change `executing` and `finished` property accordingly to `false` and `true` and generate KVO notifications.

Few things to do and to check, but not a nightmare, right?

### Building custom operation
I thought about improvement of my previously implemented operations and decided
to subclass `NSOperation` to create a bit more intelligent `Operation` class.

Main goals of this class are:

1. keeping its completion block and pass it in <span class="lang:swift decode:true  crayon-inline " >init</span> optionally.
2. starting on the same thread as queue is working on or starting on main thread.
This is useful for operations that uses e.g. `NSURLConnection` and such operations
must call `start()` method on the main thread.
3. finishing in `main()` method optionally. I don't want every time to stop
operation in `main()`, especially when this is asynchronous operation.
4. support cancellation.


Implementation is presented below. I think that the class is pretty
self-explanatory. It contains functionality listed above and is still simple.
That's good.

{% highlight swift %}
import Foundation

class Operation: NSOperation {
    private var startOnMainThread: Bool
    private var finishInMain: Bool

    // keep track of executing and finished states
    private var _executing = false
    private var _finished = false

    init(completionBlock: (() -> Void)? = nil, startOnMainThread: Bool = false, finishInMain: Bool = true) {
        self.startOnMainThread = startOnMainThread
        self.finishInMain = finishInMain
        super.init()
        self.completionBlock = completionBlock
        self.name = "custom"
    }

    override func start() {
        if cancelled {
            finish()
            return
        }

        if startOnMainThread {
            // Check if start is on main thread.
            // If not, call it on main thread and wait for execution.
            if NSThread.isMainThread() == false {
                dispatch_async(dispatch_get_main_queue()) { self.start() }
                return
            }
        }

        willChangeValueForKey("isExecuting")
        _executing = true
        didChangeValueForKey("isExecuting")
        // Call main, maybe other subclasses will want use it?
        // We have to call it manually when overriding `start`.
        main()
    }

    override func main() {
        if cancelled == true &amp;&amp; _finished != false {
            finish()
            return
        }

        if finishInMain { finish() }
    }

    /// If `finishInMain` is set to `false` you are responible to call
    /// `finish()` method when operation is about to finish.
    func finish() {        
        // Change isExecuting to `false` and isFinished to `true`.
        // Taks will be considered finished.
        willChangeValueForKey("isExecuting")
        willChangeValueForKey("isFinished")
        _executing = false
        _finished = true
        didChangeValueForKey("isExecuting")
        didChangeValueForKey("isFinished")
    }

    override var executing: Bool {
        return _executing
    }

    override var finished: Bool {
        return _finished
    }

    override func cancel() {
        super.cancel()
        finish()
    }
}
{% endhighlight %}

Let's improve operations presented in the previous post.

Since operation has custom `finish()` method implemented and we're responsible
for finishing it, it never stops in `main()` if specified to not finish there.
This provides simplicity to the code. This is what I wanted to achieve. Cool :)

{% highlight swift %}
import Foundation

typealias NetworkOperationResult = (success: Bool, data: NSData?)
protocol NetworkOperationDelegate {
    func networkOperation(operation: NetworkOperation, didFinishWithResult result: NetworkOperationResult)
}

class NetworkOperation: Operation, NSURLConnectionDelegate, NSURLConnectionDataDelegate {
    private var connection: NSURLConnection!
    private var delegate: NetworkOperationDelegate!

    init(request: NSURLRequest, delegate: NetworkOperationDelegate) {
        super.init(startOnMainThread: true, finishInMain: false)
        connection = NSURLConnection(request: request, delegate: self)
        self.delegate = delegate
        self.name = "network"
    }

    override func start() {
        // Call super to start operation on main thread.
        // NSURLConnection must start on main thread.
        super.start()
        self.connection.start()
    }

    override func cancel() {
        connection.cancel()
        super.cancel()
    }

    // MARK: - NSURLConnectionDelegate
    func connection(connection: NSURLConnection, didFailWithError error: NSError) {
        delegate.networkOperation(self, didFinishWithResult: (false, nil))
        finish()
    }

    // MARK: - NSURLConnectionDataDelegate
    func connection(connection: NSURLConnection, didReceiveData data: NSData) {
        delegate.networkOperation(self, didFinishWithResult: (true, data))
        finish()
    }
}
{% endhighlight %}

`NetworkOperation` and `ParseQuotesOperation` stopped using semaphores. Cool, we're done with those ugly semaphores in operations.

{% highlight swift %}
import Model

// Parses JSON content.
// Creates private queue to work with Core Data objects
// Creates or updates quote objects and save context before finish.
class ParseQuotesOperation: Operation {
    typealias JSONObject = Dictionary<String, AnyObject>

    private var context: NSManagedObjectContext
    var json: JSONObject!

    init(context: NSManagedObjectContext) {
        self.context = NSManagedObjectContext(concurrencyType: .PrivateQueueConcurrencyType)
        self.context.parentContext = context
        self.context.mergePolicy = NSOverwriteMergePolicy
        super.init(finishInMain: false)
        self.name = "parse"
    }

    override func main() {
        if let results = json["results"] as? [JSONObject] {
            context.performBlock {
                // Find existing quote and update it or create new one
                for quoteJSON in results {
                    if let author = quoteJSON["author"] as? String,
                        let content = quoteJSON["content"] as? String,
                        let readCount = quoteJSON["readCount"] as? Int,
                        let objectId = quoteJSON["objectId"] as? String {
                            if let quote = Quote.find(objectId, context: self.context) {
                                quote.author = author
                                quote.content = content
                                quote.readCount = readCount
                            } else {
                                _ = Quote(content: content, author: author, readCount: readCount, objectId: objectId, context: self.context)
                            }
                    } else {
                        print("Cannot parse \(quoteJSON)")
                    }
                }

                self.saveContext()
            }
        } else {
            finish()
        }
    }

    private func saveContext() {
        if self.context.hasChanges {
            do { try self.context.save() } catch {}
            do { try self.context.parentContext?.save() } catch {}
        }
        finish()
    }
}
{% endhighlight %}

`GetAllQuotesOperation` has been simplified a bit and has added one more operation
for finishing itself because we have to stop it manually now. Happy path for
the operation is like this: *Download data > Parse Data > Call completion block > Finish operation*.

{% highlight swift %}
import Foundation
import CoreData

class GetAllQuotesOperation: Operation, NetworkOperationDelegate {
    private var queue: NSOperationQueue!
    private var parseOp: ParseQuotesOperation!

    init(context: NSManagedObjectContext, completionHandler: () -> Void) {
        super.init(finishInMain: false)
        name = "get_all_quotes"

        // Create download, parse and finish operations
        let downloadOp = NetworkOperation(request: request(), delegate: self)

        parseOp = ParseQuotesOperation(context: context)
        parseOp.addDependency(downloadOp)

        let completionOp = NSBlockOperation(block: completionHandler)
        completionOp.addDependency(parseOp)

        let finishOp = NSBlockOperation(block: { self.finish() })
        finishOp.addDependency(completionOp)

        queue = NSOperationQueue()
        queue.maxConcurrentOperationCount = 1
        queue.name = "get_all_quotes_q"
        queue.addOperations([downloadOp, parseOp, completionOp, finishOp], waitUntilFinished: false)
    }

    private func request() -> NSURLRequest {
        return NSMutableURLRequest.parseRequest("classes/Quote", method: "GET")
    }

    // MARK: - NetworkOperationDelegate
    func networkOperation(operation: NetworkOperation, didFinishWithResult result: NetworkOperationResult) {
        if result.success {
            do {
                parseOp.json = try NSJSONSerialization.JSONObjectWithData(result.data!, options: NSJSONReadingOptions.MutableLeaves) as? Dictionary<String, AnyObject>
            } catch {
                queue.cancelAllOperations()
                finish()
            }
        } else {
            queue.cancelAllOperations()
            finish()
        }
    }
}
{% endhighlight %}

And here is the `UpdateReadCountOperation` not presented in the previous post
which shows how nice is to keep data decoding out of `NetworkOperation`.
It works the same way as `GetAllQuotesOperation` but sends request for increasing
read count of quote that user is currently reading. `NetworkOperation` returns
data with most recent read count and quote object is updated (someone might read
  the same quote before request is send).

{% highlight swift %}
import Foundation
import Model

class UpdateReadCountOperation: Operation, NetworkOperationDelegate {
    private var quote: Quote
    private var queue: NSOperationQueue!

    init(quote: Quote, completionBlock: () -> Void) {
        self.quote = quote
        super.init(finishInMain: false)
        name = "up_read_count"

        let updateOp = NetworkOperation(request: request(), delegate: self)
        let completionOp = NSBlockOperation(block: completionBlock)
        completionOp.addDependency(updateOp)
        let finishOp = NSBlockOperation { self.finish() }
        finishOp.addDependency(completionOp)

        queue = NSOperationQueue()
        queue.maxConcurrentOperationCount = 1
        queue.name = "up_read_count_q"
        queue.addOperations([updateOp, completionOp, finishOp], waitUntilFinished: false)
    }

    private func request() -> NSURLRequest {
        return NSMutableURLRequest.parseRequest("functions/incrementReadCount",
            method: "POST", params: ["objectId": quote.objectId])
    }

    // MARK: - NetworkOperationDelegate
    typealias JSONObject = Dictionary<String, AnyObject>
    func networkOperation(operation: NetworkOperation, didFinishWithResult result: NetworkOperationResult) {
        guard result.success == true, let data = result.data else {
            queue.cancelAllOperations()
            return
        }

        do {
            if let json = try NSJSONSerialization.JSONObjectWithData(data, options: .MutableLeaves) as? JSONObject {
                if let readCount = (json["result"] as? [JSONObject])?.first?["readCount"] as? Int {
                    // update count
                    quote.readCount = NSNumber(integer: readCount)

                    // save changes
                    if quote.hasChanges {
                        do {
                            try quote.managedObjectContext?.save()
                        } catch let error as NSError {
                            print("cannot save context: \(error)")
                        }
                    }
                }
            }
        } catch let error as NSError {
            print("cannot serialize data to json: \(error)")
            queue.cancelAllOperations()
            finish()
            return
        }
    }
}
{% endhighlight %}


### Conclusion
I am fine to say *"It looks a lot better now and I like operations!"*.
Operations are powerful and makes code cleaner and more loosely coupled.
The Operation class is ready to re-use and easy to use which is great.
No semaphores anymore :)

I think the second try is much better, what do you think?

**Update 15 Jul 2015 16:17**

I've used the code in real project and improved it. I realized that I don't
need this `startOnMainThread` property because I always need to override `start()`
method to do some extra things anyway. I removed the property and `Operation`
class is even simpler.


{% highlight swift %}
// Part of Operation class.

override func start() {
    if cancelled {
        finish()
        return
    }

    willChangeValueForKey("isExecuting")
    _executing = true
    didChangeValueForKey("isExecuting")
    // Call main, maybe other subclasses will want use it?
    // We have to call it manually when overriding `start`.
    main()
}
{% endhighlight %}


[previous-post]: http://szulctomasz.com/ios-using-nsoperation-for-background-tasks/
[previous-post-2]: http://szulctomasz.com/starting-project-in-swift-2-for-ios-9-8-features-testing/
[quotes]: https://github.com/tomkowz/Quotes
[twitter-post]: https://twitter.com/codinginswift/status/620609220090540032
