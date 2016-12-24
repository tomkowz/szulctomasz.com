---
layout: post
title: "iOS: Using NSOperation for background tasks"
tags: ios, gcd, nsoperation, swift, wwdc
id: post-15
redirect_from: "/ios-using-nsoperation-for-background-tasks/"
---
I has been inspired by this great [Advanced NSOperations][wwdc-226] WWDC 2015
session yesterday and wanted to perform tasks in [Quotes][quotes] app in NSOperation-manner.

**If you're not familiar with [Quotes][quotes] yet, please read this [brief introduction][previous-post].**

Guys at the session did great talk about `NSOperation` and I recommend to watch
it. I wasn't familiar with subclassing `NSOperation` class - I used the class
sometimes, but it was `NSBlockOperation` for scheduling few blocks to run one
after another in some order using `NSOperationQueue`. During the session I was
thinking: *Hey, why didn't I even subclass NSOperation in my code and why didn't
I create any task using NSOperation?* So, I played with the [Quotes][quotes]
today and created few tasks based on `NSOperation`.

## Idea
The idea was similar to idea of presenters - there is some backend which app
communicates with, and the app fetch data from it, parse it and updates UI.
Simple. I browsed sample code and it was a bit too complex for what I needed
so I wrote it completely from nothing but you can see some similarity.

I decided to integrate app with [parse.com][parse] because I haven't opportunity
to use it before. There is a database that contains `Quote` objects and all of
them can be downloaded by sending `GET` request to `/classes/Quote` endpoint.

Here is the flow of the long running task - user is entering main view, fetch
request is added to queue with QoS set to User Initiated so task are pretty
important for the system. Data is received, and next dependent task is running,
that is, parsing. Data is parsed, changes are saved into persistent store and
table is ready to be reloaded to present new data.

![image-1][img-1]

## NSOperation, GCD and QoS
`NSOperation` and `NSOperationQueue` were in the passed different mechanisms
than Grand Central Dispatch but with iOS 4 it have been rewrote on top of GCD.
The GCD is good for concurrent execution of a code that don't need to be
scheduled. There is no ready mechanism for cancelling or suspending blocks
that are in progress.

`NSOperation` is higher level mechanism that gives ability to schedule, cancel
and suspend. It also allows to add dependency among operations. So, If you want
to do some longer running operations like downloading and parsing data, this is
probably mechanism you want to use.

`NSOperationQueue` can perform one or more concurrent operations at the same
time. It is specified by `maxConcurrentOperationCount` property. It also has `qualityOfService` property that indicates to the system the nature and importance
of work. Downloading and parsing is initiated by the user when entering the app
and user wants to see results as soon as possible so `UserInitiated` QoS is good
choice. In `Foundation > NSObjCRuntime` you can find description of all of them.

{% highlight swift %}
@available(iOS 8.0, *)
enum NSQualityOfService : Int {
    // UserInteractive QoS is used for work directly involved in providing an
    // interactive UI such as processing events or drawing to the screen.
    case UserInteractive

    // UserInitiated QoS is used for performing work that has been explicitly
    // requested by the user and for which results must be immediately presented
    // in order to allow for further user interaction. For example, loading an email
    // after a user has selected it in a message list.
    case UserInitiated

    // Utility QoS is used for performing work which the user is unlikely to be
    // immediately waiting for the results.  This work may have been requested by the user
    // or initiated automatically, does not prevent the user from further interaction, often
    // operates at user-visible timescales and may have its progress indicated to the user by
    // a non-modal progress indicator.  This work will run in an energy-efficient manner,
    // in deference to higher QoS work when resources are constrained.  
    // For example, periodic content updates or bulk file operations such as media import.
    case Utility

    // Background QoS is used for work that is not user initiated or visible.  In general,
    // a user is unaware that this work is even happening and it will run in the most efficient
    // manner while giving the most deference to higher QoS work.  For example, pre-fetching
    // content, search indexing, backups, and syncing of data with external systems.
    case Background

    // Default QoS indicates the absence of QoS information.  Whenever possible QoS information
    // will be inferred from other sources.  If such inference is not possible, a QoS between
    // UserInitiated and Utility will be used.
    case Default
}
{% endhighlight %}


## Implementing tasks

As mentioned before the long running operation of updating content of the app
has been decoupled to three small tasks: download data, parse data, notify about
end of the long running operation. To implement this we need to create `DownloadOperation`, `ParseQoutesOperation` and `GetAllQuotesOperation` which contains these operations - also, the last operation will be added to the queue in `QuoteListViewController` which displays all quotes.

{% highlight swift %}
class GetAllQuotesOperation: NSOperation, DownloadOperationDelegate {
    private var queue = NSOperationQueue()
    private var parseQuotesOp: ParseQuotesOperation!

    // Init operation with context which will be used during downloaded data
    // parsing. From the context a child one will be created, after all is done
    // child and this passed will be saved.
    // Completion handler is called by the last operation.
    init(context: NSManagedObjectContext, completionHandler: () -> Void) {
        super.init()
        name = "get.all.quotes"

        // Create download, parse and finish operations
        let request = NSMutableURLRequest.parseRequest("classes/Quote", method: "GET")
        let downloadOp = DownloadOperation(request: request, delegate: self)

        parseQuotesOp = ParseQuotesOperation(context: context)
        parseQuotesOp.addDependency(downloadOp)

        let finishOp = NSBlockOperation(block: completionHandler)
        finishOp.addDependency(parseQuotesOp)

        queue.addOperations([downloadOp, parseQuotesOp, finishOp], waitUntilFinished: false)
    }

    // MARK: - DownloadOperationDelegate
    // Delegate method that updates parsing operation with data to be parsed
    func downloadOperation(operation: DownloadOperation, didFinishDownloadingWithResult result: DownloadOperationResult) {
        if result.success {
            parseQuotesOp.json = result.json!
        } else {
            queue.cancelAllOperations()
        }
    }
}
{% endhighlight %}

`DownloadOperation` is small and easy to be reused with any request that end up
with some downloaded JSON. It takes request, creates connection, gets results
data and tries to convert it to dictionary. Because there is asynchronous work
to do it uses semaphore to not finish itself too fast.

{% highlight swift %}
typealias DownloadOperationResult = (success: Bool, json: Dictionary<String, AnyObject>?)
protocol DownloadOperationDelegate {
    func downloadOperation(operation: DownloadOperation, didFinishDownloadingWithResult result: DownloadOperationResult)
}

// Operation uses internal connections to perform request passed in init method.
// After content is downloaded it convert NSData to json-like object and call
// delegate method.
class DownloadOperation: NSOperation, NSURLConnectionDelegate, NSURLConnectionDataDelegate {
    private var connection: NSURLConnection!
    private var delegate: DownloadOperationDelegate!
    private var semaphore = dispatch_semaphore_create(0)

    init(request: NSURLRequest, delegate: DownloadOperationDelegate) {
        super.init()
        connection = NSURLConnection(request: request, delegate: self)
        self.delegate = delegate
    }

    override func main() {
        // NSURLConnection need to be started on the main thread
        dispatch_async(dispatch_get_main_queue()) { self.connection.start() }
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER)
    }

    private func unlock() {
        dispatch_semaphore_signal(semaphore)
    }

    private func markFailed() {
        delegate.downloadOperation(self, didFinishDownloadingWithResult: (false, nil))
    }

    // MARK: - NSURLConnectionDelegate
    func connection(connection: NSURLConnection, didFailWithError error: NSError) {
        print("Error downloading quotes")
        markFailed()
        unlock()
    }

    // MARK: - NSURLConnectionDataDelegate
    func connection(connection: NSURLConnection, didReceiveData data: NSData) {
        do {
            if let json = try NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions.MutableLeaves) as? Dictionary<String, AnyObject> {
                delegate.downloadOperation(self, didFinishDownloadingWithResult: (true, json))
            }
        } catch {
            markFailed()
        }
        unlock()
    }
}
{% endhighlight %}

At the end of its work the download operation calls delegate method. This is
needed to pass data between two operations. Next operation that takes this data
is `ParseQuotesOperation`. It also takes `NSManagedObjectContext` to work with
`Quote` objects. This operation also perform some asynchronous work like saving
contexts so it also needs additional semaphore to work correctly.

{% highlight swift %}
// It parses json content set as its property.
// It creates private queue councurrency type context to work on with overwrite
// merge policy so we're sure that if there is some conflic it will be resolved
// by using downloaded data.
// If objectId is not found then new object is created, otherwise it is updated.
// At the end passed context and its parent context are saved
class ParseQuotesOperation: NSOperation {
    var context: NSManagedObjectContext
    var json: Dictionary<String, AnyObject>!
    var semaphore = dispatch_semaphore_create(0)

    init(context: NSManagedObjectContext) {
        self.context = NSManagedObjectContext(concurrencyType: .PrivateQueueConcurrencyType)
        self.context.parentContext = context
        self.context.mergePolicy = NSOverwriteMergePolicy
        super.init()
    }

    override func main() {
        guard let quotesJSON = json["results"] as? Array<Dictionary<String, AnyObject>> else { return }

        context.performBlock {
            // Find existing quote and update it or create new one
            for dictionary in quotesJSON {
                if let author = dictionary["author"] as? String,
                    let content = dictionary["content"] as? String,
                    let objectId = dictionary["objectId"] as? String {
                        if let quote = Quote.find(objectId, context: self.context) {
                            quote.author = author
                            quote.content = content
                        } else {
                            _ = Quote(content: content, author: author, objectId: objectId, context: self.context)
                        }
                } else {
                    print("Cannot parse \(dictionary)")
                }
            }

            self.saveContext()
        }

        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER)
    }

    private func saveContext() {
        if self.context.hasChanges {
            do { try self.context.save() } catch {}
            do { try self.context.parentContext?.save() } catch {}
            dispatch_semaphore_signal(semaphore)
        }
    }
}
{% endhighlight %}

The last thing to do is configuration of `NSOperationQueue` in view controller
and implementation of method that starts downloading process.

{% highlight swift %}
private func configureOperationQueue() {
    operationQueue = NSOperationQueue()
    operationQueue.maxConcurrentOperationCount = 1
    operationQueue.qualityOfService = .UserInitiated
    operationQueue.name = "quotes.list.queue"
}

private func downloadPublicQuotes() {
    let context = CoreDataStack.sharedInstance().mainContext
    operationQueue.addOperation(GetAllQuotesOperation(context: context) {
        dispatch_async(dispatch_get_main_queue()) { self.refreshAll() }
        })
}
{% endhighlight %}

## Conclusion
I like separation of tasks and dependency among operations. Tasks can be reused easily in the future in other places. Because all tasks are operations the view controller needed only few lines to support downloading additional data.

**Update 13 Jul 2015 17:15**

I played with these described operations and changed it a bit. `DownloadOperation` changed to `NetworkOperation` which can send POST/GET or whatever request you
want instead of being able only to download data. Start parsing requires
passing downloaded response so I decided to parse it outside of the
`NetworkOperation` - it could be done better I think, but let's keep it as is
for now - will update it when better/cleaner solution comes up. I've also
updated [Quotes][quotes] project and added `UpdateReadCountOperation` which
increments counter of readings selected quote.

**Update 14 Jul 2015 12:43**

*Please read newest update about [second try to NSOperation and long running tasks][previous-post-2] after you finish this one.*

[wwdc-226]: https://developer.apple.com/videos/wwdc/2015/?id=226
[quotes]: https://github.com/tomkowz/Quotes
[previous-post]: http://szulctomasz.com/starting-project-in-swift-2-for-ios-9-8-features-testing/
[parse]: https://parse.com
[previous-post-2]: http://szulctomasz.com/ios-second-try-to-nsoperation-and-long-running-tasks/

[img-1]: /uploads/{{page.id}}/1.png
