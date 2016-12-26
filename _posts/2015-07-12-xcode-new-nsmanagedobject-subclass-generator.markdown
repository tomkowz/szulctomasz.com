---
layout: post
title: "Xcode: New NSManagedObject subclass generator"
tags: ios, swift, core data, xcode
post_id: post-16
redirect_from: "/xcode-new-nsmanagedobject-subclass-generator/"
---
Xcode 7 includes new generator of NSManagedObject subclasses and it works for
both Swift and Objective-C classes.

I was playing with Core Data and I noticed that Xcode 7 generates subclasses
for entities in a different manner and this is cool. It does similar work like [mogenerator][mogenerator] but instead of creating "human" and "machine" classes
like mogenerator does it creates class for entity where class logic is and `CoreDataProperties` extension which contains declaration of attributes
of the entity.

*Quote.swift*
{% highlight swift %}
@objc(Quote)
class Quote: NSManagedObject {
    // Logic goes here
}
{% endhighlight %}

*Quote+CoreDataProperties.swift*
{% highlight swift %}
import Foundation
import CoreData

extension Quote {

    @NSManaged var author: String?
    @NSManaged var content: String?
    @NSManaged var objectId: String?
    @NSManaged var readCount: NSNumber?

}
{% endhighlight %}

## Glad
I like that extension is generated and properties are there, it ensure that
regenerating subclass will not overwrite class logic - Finally!

## Sad
The generator or better Core Data model editor should have more "advanced"
options - I meant, hey, there is no option to select if property is optional
or not. It generates properties and marks them optionals by default - I don't
like it because in my case I want non-optional properties for Quote class - [rdar://21785457][rdar-1] opened.

It should be generated as below:

{% highlight swift %}
import Foundation
import CoreData

extension Quote {

    @NSManaged var author: String
    @NSManaged var content: String
    @NSManaged var objectId: String
    @NSManaged var readCount: NSNumber

}
{% endhighlight %}

What I am sad about is also that even <em>readCount</em> has specified default
value set to 0 it also generate it as optional.

The second thing I noticed is that regenerating class after e.g. model has been
updated causes that Xcode asks whether you want to replace existing extension
of entity class file with new properties declarations. If you confirm it will
replace extension which is okay, but it also adds extra reference to this
extension to the project navigator there are two references pointing to the
same file - [rdar://21785404][rdar-2] opened.


## Conclusion
This is nice that we've got finally generator that separates properties
declaration from entity logic. I think this new generator is a good tool for
first time generation of newly added entity. Looking for my sad things I
recommend to update the extension manually every time when it changes and when
you've got custom code inside extension file like: non-default access modifiers
or properties marked non-optionals.

[mogenerator]: http://rentzsch.github.io/mogenerator/
[rdar-1]: http://www.openradar.me/21785457
[rdar-2]: http://www.openradar.me/21785404
