---
layout: post
title: "Swift 2: @NSManaged can be used for autogenerated methods"
tags: ios, swift, xcode, core data
post_id: post-29
redirect_from: "/swift-2-nsmanaged-for-methods/"
---
Another addition to Swift 2 in Xcode 7 beta 5. We can use `@NSManaged` to declare
autogenerated methods when using to-many relationships.

I see it very useful. Let's say you've got `Library` and `Book` entities. `Library`
has to-many `books` relationship. With new Xcode and `@NSManaged` you can declare
autogenerated methods in the `Library` entity (manually).

This is how it look like:
{% highlight swift %}
class Library: NSManagedObject {
    @NSManaged func addBooksObject(book: Book)
    @NSManaged func removeBooksObject(book: Book)
    @NSManaged func addBooks(books: Set<Book>)
    @NSManaged func removeBooks(books: Set<Book>)
}
{% endhighlight %}

This is nice, I appreciate that we're able to do this. I remember I had to write
such methods from scratch recently.

I noticed an issue here. Yes we can declare such methods, but what was the
problem to make them generated anyway and put them in to the
*Entity+CoreDataProperties.swift* files? When generating Objective-C subclasses even
in a Swift project the methods are presented there. When generating in Swift
language - NO. Opened [rdar://22177139][rdar-1] to keep an eye on this.

This is from entity class generated in Objective-C language.

{% highlight objc %}
@interface Library (CoreDataGeneratedAccessors)
- (void)addBooksObject:(Book *)value;
- (void)removeBooksObject:(Book *)value;
- (void)addBooks:(NSSet<Book *> *)values;
- (void)removeBooks:(NSSet<Book *> *)values;

@end
{% endhighlight %}

There is a lot more methods to declare manually when you mark relationship to be ordered.
{% highlight objc %}
- (void)insertObject:(Book *)value inBooksAtIndex:(NSUInteger)idx;
- (void)removeObjectFromBooksAtIndex:(NSUInteger)idx;
- (void)insertBooks:(NSArray<Book *> *)value atIndexes:(NSIndexSet *)indexes;
- (void)removeBooksAtIndexes:(NSIndexSet *)indexes;
- (void)replaceObjectInBooksAtIndex:(NSUInteger)idx withObject:(Book *)value;
- (void)replaceBooksAtIndexes:(NSIndexSet *)indexes withBooks:(NSArray<Book *> *)values;
- (void)addBooksObject:(Book *)value;
- (void)removeBooksObject:(Book *)value;
- (void)addBooks:(NSOrderedSet<Book *> *)values;
- (void)removeBooks:(NSOrderedSet<Book *> *)values;
{% endhighlight %}

Do not want to do this manually... Hope they fix it soon :)

Another issue that I am waiting to be fixed by Apple is the one with ordered to-many
relationship and those auto-generated methods. This bug exists for a very long time.
Don't know if there are bug reports for this one. I think it is from the time when
Core Data and this functionality has been released for the first time. Let's say
I want to add some Book to a Library:

{% highlight swift %}       
let ctx = self.managedObjectContext
let library = NSEntityDescription.insertNewObjectForEntityForName("Library", inManagedObjectContext: ctx) as! Library
let book1 = NSEntityDescription.insertNewObjectForEntityForName("Book", inManagedObjectContext: ctx) as! Book
library.addBooksObject(book1)
{% endhighlight %}

Not possible.

{% highlight plain %}
2015-08-06 23:14:18.541 NewNSManagedExample[54727:3677632] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** -[NSSet intersectsSet:]: set argument is not an NSSet'
*** First throw call stack:
(
	0   CoreFoundation                      0x00ea83b4 __exceptionPreprocess + 180
	1   libobjc.A.dylib                     0x005cde02 objc_exception_throw + 50
	2   CoreFoundation                      0x00dfc574 -[NSSet intersectsSet:] + 260
	3   Foundation                          0x00214756 NSKeyValueWillChangeBySetMutation + 153
	4   Foundation                          0x0017c4c7 NSKeyValueWillChange + 394
	5   Foundation                          0x0021466a -[NSObject(NSKeyValueObserverNotification) willChangeValueForKey:withSetMutation:usingObjects:] + 630
	6   CoreData                            0x00a981c6 _sharedIMPL_addObjectToSet_core + 182
	7   CoreData                            0x00a99189 __generateAccessor_block_invoke_2 + 41
	8   NewNSManagedExample                 0x000f0e80 _TFC19NewNSManagedExample11AppDelegate11applicationfS0_FTCSo13UIApplication29didFinishLaunchingWithOptionsGSqGVSs10DictionaryCSo8NSObjectPSs9AnyObject____Sb + 720
	9   NewNSManagedExample                 0x000f10c7 _TToFC19NewNSManagedExample11AppDelegate11applicationfS0_FTCSo13UIApplication29didFinishLaunchingWithOptionsGSqGVSs10DictionaryCSo8NSObjectPSs9AnyObject____Sb + 199
	10  UIKit                               0x0122e1c6 -[UIApplication _handleDelegateCallbacksWithOptions:isSuspended:restoreState:] + 337
	11  UIKit                               0x0122f56c -[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 3727
	12  UIKit                               0x01236929 -[UIApplication _runWithMainScene:transitionContext:completion:] + 1976
	13  UIKit                               0x01259af6 __84-[UIApplication _handleApplicationActivationWithScene:transitionContext:completion:]_block_invoke3142 + 68
	14  UIKit                               0x012336a6 -[UIApplication workspaceDidEndTransaction:] + 163
	15  FrontBoardServices                  0x03ff9ccc __37-[FBSWorkspace clientEndTransaction:]_block_invoke_2 + 71
	16  FrontBoardServices                  0x03ff97a3 __40-[FBSWorkspace _performDelegateCallOut:]_block_invoke + 54
	17  FrontBoardServices                  0x040171cb -[FBSSerialQueue _performNext] + 184
	18  FrontBoardServices                  0x04017602 -[FBSSerialQueue _performNextFromRunLoopSource] + 52
	19  FrontBoardServices                  0x040168fe FBSSerialQueueRunLoopSourceHandler + 33
	20  CoreFoundation                      0x00dc27af __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 15
	21  CoreFoundation                      0x00db843b __CFRunLoopDoSources0 + 523
	22  CoreFoundation                      0x00db7858 __CFRunLoopRun + 1032
	23  CoreFoundation                      0x00db7196 CFRunLoopRunSpecific + 470
	24  CoreFoundation                      0x00db6fab CFRunLoopRunInMode + 123
	25  UIKit                               0x01232f8f -[UIApplication _run] + 540
	26  UIKit                               0x01238724 UIApplicationMain + 160
	27  NewNSManagedExample                 0x000f24dc main + 140
	28  libdyld.dylib                       0x039afa21 start + 1
)
libc++abi.dylib: terminating with uncaught exception of type NSException
{% endhighlight %}

The only workaround for this is to re-implement those methods.
Opened [rdar://22177512][rdar-2] - Hope they fix it finally.

[rdar-1]: http://www.openradar.me/22177139
[rdar-2]: http://www.openradar.me/22177512
