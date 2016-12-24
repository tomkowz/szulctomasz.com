---
layout: post
title: "Swift 2: Error handling using ErrorType, throws, try and do-catch"
tags: ios, swift
id: post-31
redirect_from: "/swift-2-error-handling-using-errortype-throws-try-and-do-catch/"
---
Swift 2.0 brought us `ErrorType`. Let's take a brief look on it.

### ErrorType
I believe this protocol is implemented by every error type in Apple's frameworks.
There is no [documentation][docs] for this protocol so far, just a page for it
with notice that this is preliminary document and by which other types this one
is adopted.

Protocol declaration and its extension is pretty much simple like the
documentation, meaning, it is empty.

{% highlight swift %}
public protocol ErrorType {}
extension ErrorType {}
{% endhighlight %}

It is empty but enums, error-like classes/structs have to implement it to be
able to throw them in methods.

`throws` is nice addition to Swift, it makes methods declaration simpler and
error handling easier.

### Before ErrorType
In Objective-C method declaration which might return error look like this

{% highlight swift %}
- (BOOL)removeFileAtURL(NSURL *)url error:(NSError **)error;
{% endhighlight %}

In Swift older than Swift 2.0 it changed a bit to

{% highlight swift %}
func removeFileAt(url: NSURL, error: NSErrorPointer) -> Bool
{% endhighlight %}

`error` parameter is not declared as optional but it may take nil. It is possible
because `NSErrorPointer` is just typealias of `AutoreleasingUnsafeMutablePointer<NSError?>`.
Struct `AutoreleasingUnsafeMutablePointer` as inline documentation comment says
is just mutable pointer to Objective-C pointer argument. It may take nil or
inout argument of referenced type.

To call such method in Swift you need to pass nil or create variable of type `NSError?`
and pass it by reference using `&amp;` operator (`&amp;err`) and you can get some
`Bool` value from a method as well as error object if returned - *How many times did
you just pass nil in Objective-C because you didn't want to handle errors because
of increased code complexity?! :> ... I thought that you answered "0 times,
I always handle errors."* :)

### throw and ErrorType in use
It was always the case that when something failed in such kind of method then
it returned false and filled passed error variable or just return true and
error was nil. Apple figured that out it could be improved. They added `throws`
in Swift 2.0.

In Swift 2.0 the method might be defined as

{% highlight swift %}
func removeFileAt(url: NSURL) throws -> Bool
{% endhighlight %}

It means, that it returns `Bool` or throws if something goes wrong. The method
declaration could be even simpler in this case, since we expect that method throws
which means something went wrong or returns `Bool` which is always true if
returned which means success - the return type can be dropped.

{% highlight swift %}
func removeFileAt(url: NSURL) throws
{% endhighlight %}

`throws` forces us to use `do-catch` around the code that can throw errors.

Let's pretend we've got secure store which is somewhere in an app directory and
we're using `SecureStore` class to store data inside. There is a method which
takes `NSData` object and path to a directory where this data will be stored.
It returns identifier of stored data if successfully saved or throws an error.
Here is possible declaration of such method:
{% highlight swift %}
func storeData(data: NSData, path: String) throws -> String
{% endhighlight %}

Let's say method just throws some error if something went wrong, `NSError` for
example. To execute the method it has to be around do-catch.

{% highlight swift %}
do {
    let dataIdentifier = try store.storeData(NSData(), path: "path/to/dir")
    // do something with the dataIdentifier
} catch {
    // or catch let error as NSError {
    // if you want to use the thrown error.
    // or any other type of error you expect here.
    print("some error")
}
{% endhighlight %}

But that's not all. `NSError` needs some error domain and code to be defined.
We can use `ErrorType` protocol and create own errors, e.g. using enums.

Let's define some enumeration which implements `ErrorType` protocol so it can
be throw by our method.

{% highlight swift %}
enum SecureStoreError: ErrorType {
    case DirectoryNotExist
    case NoRights
}
{% endhighlight %}

Handling could look like this:

{% highlight swift %}
do {
    try store.storeData(NSData(), path: "path/to/dir")
} catch SecureStoreError.DirectoryNotExist {
    print("dir not exist")
} catch SecureStoreError.NoRights {
    print("no rights")
} catch {
    print("not handled error")
}
{% endhighlight %}

You can omit `do-catch` by calling a method with `try!` instead of `try`.
You can do this when you're sure that your specific case of calling the method
will not throw errors. It disables error handling and if any error occurs it
will end with runtime error and crash of your code. Use it carefully!

But, how the method look inside? How to throw? What with the value that method
returns? Here is simple implementation of above method.

{% highlight swift %}
class SecureStore {

    func storeData(data: NSData, path: String) throws -> String {
        // Some logic goes here...
        // More logic...

        // Ooops, directory not exist, throw error
        throw SecureStoreError.DirectoryNotExist

        // the code here is executed when directory exist.

        // checking rights...
        // Oops, no write-rights, throw error
        throw SecureStoreError.NoRights

        // We've got rights

        // Saving logic goes here
        // Generates some identifier for stored data
        // This will be only returned if method didn't throw above.
        return NSUUID().UUIDString
    }
}
{% endhighlight %}

### Asynchronous code
**UPDATE 1**: As [Charlag Khan][charlag-khan-twitter]
rightly noted that I didn't cover handling errors in asynchronous code at
all - Thanks, this is also important thing worth to mention about. [Post on twitter](https://twitter.com/charlag_khan/status/632018006667870208)

So new error handling works well in synchronous code. Unfortunately we're not
able to use do-catch and handle errors the same way with asynchronous code because
the method returns/ends before asynchronous operation ends. There is no best
solution for this (at least for now), we can use some kind of `Result` enum which
will wrap errors in case of failure and returned value in a case of success.
Here is good example of our `SecureStore`.

{% highlight swift %}
// Errors for loading process.
enum SecureStoreLoadDataError: ErrorType {
    case NoFileForIdentifier
    case FileCorrupted
    case NotHandled
}

// Wrapper for errors and success value.
// Can be initialized with some T value or U error.
enum Result<T, U> {
    case Success(T)
    case Failure(U)
}

class SecureStore {
	// ...

	func loadData(identifier: String, completion: (result: Result<NSData, SecureStoreLoadDataError>) -> Void) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
            do {
                let data = try self.loadingLogicGoesHere(identifier)
                completion(result: Result.Success(data))

            } catch let error as SecureStoreLoadDataError {
                completion(result: Result.Failure(error))

            } catch {
                completion(result: Result.Failure(SecureStoreLoadDataError.NotHandled))
            }
        }
    }

    private func loadingLogicGoesHere(identifier: String) throws -> NSData {
        // Check if there is such file associated with identifier and throw
        // proper error if there is any.
        throw SecureStoreLoadDataError.NoFileForIdentifier

        // Check if we're able to load a content of a file or whether file is
        // corrupted and throw error if needed
        throw SecureStoreLoadDataError.FileCorrupted

        // All good, data loaded
        return NSData()
    }
}

// usage of such method
store.loadData("ABC") { result in
    switch result {
    case let .Success(d):
        print("good \(d)")

    case let .Failure(error):
        print("error \(error)")
    }
}
{% endhighlight %}

So, as you can see it reminds a bit Objective-C's way of handling errors in
asynchronous code. There is a private `loadingLogicGoesHere` method which is
do-catched inside and then the error is passed via `Result` enum to the block
which handles all the errors and does specific things for them. I tried to
figure out a better solution but do not have idea. I browse Apple Developer Forums
and found [this topic][adf-1] where person with nick **mmm** suggested a nice way
to solve this problem in Swift. I like it, we'll see whether this will be live
with next updates or not (The post is 2 months to date).

**UPDATE 2**: Charlag Khan replied on my tweet and wrote there is another way
we can tackle with asynchronous error handling.

He shared link to [Jeremy W. Sherman][jeremy-twitter] post about ["Swift throws with completion callbacks"][jeremy-post].

Jeremy figured out that if function call can throw we can pass the function as
parameter to a callback and execute it in the block and handle error there.
This let us drop this `Result` struct presented above and simplify `loadData` method.

{% highlight swift %}
// create typealias for function that will be passed to the completion callback.
typealias LoadDataResult = (identifier: String) throws -> NSData

// here is updated method.
func loadData(completion: LoadDataResult -> Void) {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
        completion(self.loadingLogicGoesHere)
    }
}

// let's call that method
store.loadData { (result: SecureStore.LoadDataResult) -> Void in
    do {
        // execute result function with identifier of data we want.
        let data = try result(identifier: "ABC")
        print("data \(data)")
    } catch let err as SecureStoreLoadDataError {
        print("err: \(err)")
    } catch {
        print("not handled")
    }
}
{% endhighlight %}

### Wrap up
`try`, `do-catch`, `throws` and `ErrorType` are nice additions to the Swift 2.0.
I believe it will makes our code easier to read, understand and debug without
thinking if error will be returned or not if there is some other thing returned
by the method like in example above. It creates nice separation between success
and failure scenarios, handling is clearer than with `NSError` floating around
the method in Objective-C.

It still needs some additional work to make it more consistent in terms of
handling synchronous and asynchronous code. Would be nice to not have to use
any additional transfer data enums like `Result`.

Declaring own error types and errors is very easy. We do not need to subclass `NSError`
or wrap it somehow, just create simple enum that implements `ErrorType` protocol
and it is ready to use.

[docs]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Reference/Swift_ErrorType_Protocol/index.html#//apple_ref/swift/intf/s:PSs9ErrorType
[charlag-khan-twitter]: https://twitter.com/Charlag_Khan
[adf-1]: https://forums.developer.apple.com/thread/4354
[jeremy-twitter]: https://twitter.com/jeremywsherman/
[jeremy-post]: https://jeremywsherman.com/blog/2015/06/17/using-swift-throws-with-completion-callbacks/
