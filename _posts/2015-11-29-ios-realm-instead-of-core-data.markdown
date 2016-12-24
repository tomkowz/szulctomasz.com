---
layout: post
title: "iOS: Realm instead of Core Data"
tags: ios, swift, realm, core data
id: post-41
excerpt: "Let's take a look how easy to use Realm is."
redirect_from: "/ios-realm-instead-of-coredata/"
---
I read many times "[Realm][realm]" word on the internet. I even had opportunity
to attend one of Realm's Swift Language User Group meetups in October. Finally,
I found free time to use their framework.

### The case
Currently I am on a diet. I need to loose some weight because I put on a bit
there in California - so much good food :) I browsed the iTunes Store looking
for an app that tracks water intake, and found few that IMO were looking bad or
UX was poor. I thought, if I decide to create such app for me I'll kill two
birds with one stone - I'll write the app for me that look like I want to, and
I'll use Realm framework instead of Core Data. So I did.

My first thoughts after I went through a documentation and started using Realm framework?
**Wow, this is damn good. Guys did a great job!**

<strong>Disclaimer</strong>: The following text covers just a bit of Realm framework.
It shows the basics. I recommend to go and read [documentation for more details][realm-docs].
I didn't want to show you everything about Realm here, because the post would
be as long as the documentation is, and I wanted to avoid this.
The documentation is really good. You'll have to read it too before start working on it.

My use case wasn't complicated, was rather simple. The app has only two model
classes: `Day` and `DrinkLogEntry`. Besides, I needed also functionalities like:
creating, updating, filtering and sorting stored data - As I mentioned, simple app.
Below I'll present some snippets from the app I created.

### Model
There is no xcdatamodel-like file for Realm. The model are just files with
classes that inherits from `Object` class.

 {% highlight swift %}
/**
 Represents day in a user's life. Day contains information about water
 drank by user and their daily goal.
*/
class Day: Object {

    dynamic var identifier: String!

    /// timestamp of the beginning of a day in UTC+0
    dynamic var timestamp: NSTimeInterval = 0

    /// amount of mililiters of water that user drank
    dynamic var waterDrank: Float = 0

    /// amount of mililiters of water that user have to drink
    dynamic var dailyGoal: Float = 0 // ml

    var drinkLogs = List<DrinkLogEntry>()

    convenience init(timestamp: NSTimeInterval) {
        self.init()
        self.timestamp = timestamp
        self.identifier = Day.convertTimestampIntoIdentifier(timestamp)
    }

    override class func primaryKey() -> String? {
        return "identifier"
    }

    override class func indexedProperties() -> [String] {
        return ["identifier"]
    }

    class func convertTimestampIntoIdentifier(timestamp: NSTimeInterval) -> String {
        return String(format: "%.0f", arguments: [timestamp])
    }
}
{% endhighlight %}

Any properties that have dynamic keyword will be parsed and will be part of a
data model. Realm also supports relationships. In example the Day class has
one-to-many relationship called drinkLogs. One-to-one relationship is just a
property with proper type.

Realm also supports migration between versions like Core Data. You can define
a block that will be executed when migration is needed, and you can perform all
the actions you need to properly go through migration.

### Indexed properties and primary keys
The Realm framework has nice features that Core Data does not have
(or I couldn't find them, or I just wanted to mention about them :) ). The first
one are indexed properties. You can declare array of properties that are indexed.
So searching should be faster with smaller number of properties. This is good
for performance.

Next thing are primary keys. You can declare which property is a primary key
for the model object. The key is used for updating more efficiently and
enforces uniqueness for each value.

In my use case both primary key and indexed property is "identifier" which
I used for searching and updating objects.

The object can have also ignored properties, that will not be persistent.

### Creating, updating and writing
You can use model object as is without persisting them and write them into Realm
store after the object is ready for that. This is the thing that I like more
than Core Data solution to play with temporary child context.

{% highlight swift %}
let day = Day(timestamp: timestamp)
day.dailyGoal = MenuSettings().dailyGoal
{% endhighlight %}

To write to Realm or to read from you have to create `Realm` instance.

{% highlight swift %}
let realm = try! Realm()
{% endhighlight %}

This is how you can add new object to database.

{% highlight swift %}
try! realm.write {
    realm.add(day)
}
{% endhighlight %}

I like the way how objects can be updated. Let's pretend that some object is
downloaded from the internet, it is mapped and added to the database.
An existing object in database should be updated instead of adding new if there is any.

{% highlight swift %}
func fetchAll(completion: [Day] -> Void) {
    /**
     Let's pretend that request returned json and data is mapped into
     model objects of Day class.

     Created objects are not stored in Realm database yet.
     Object's identifier is equal timestamp.
     */
    let day1 = Day(timestamp: 0)
    let day2 = Day(timestamp: 86400)
    let day3 = Day(timestamp: 172800)

    completion([day1, day2, day3])
}

func sync() {
    fetchAll { (days) -> Void in
        let realm = try! Realm()
        try! realm.write {
            /// If there is object with the same identifier it will be updated.
            realm.add(days, update: true)
        }
    }
}
{% endhighlight %}

This is a way better than manually performing queries on context and looking
for objects with the same identifier and then updating fields.

If update parameter will be false, and new object will have the same primary
key as existing object, the exception will be thrown.

There are other ways to update objects that I'll not cover in this article.

This is how you can get all `Day` objects.

{% highlight swift %}
let days = realm.objects(Day.self)
{% endhighlight %}

Filtering is very simple too.
{% highlight swift %}
realm.objects(Day.self).filter("identifier == %@", dayIdentifier)
{% endhighlight %}

Here are all days sorted ascending by timestamp.
{% highlight swift %}
let days = realm.objects(Day.self).sorted("timestamp", ascending: true)
{% endhighlight %}

Every time you perform `object()`, `sorted()`, `filter()` you'll get `Results<T>`
object that allow you to make additional filtering, sorting, etc - this is powerful
and easy to use.

### Conclusion
Realm - <strong>would I use it in next app</strong> that have even a more
complicated model? For sure <strong>YES</strong>. This is very easy to use
framework that can be quickly integrated with the app and provides great, powerful features.

P.S. The app is waiting for review :)

**Update 06/12/2015**

The app landed in the store today - [Water Intake - Drink more water and track it daily][store]

[realm]: http://realm.io
[realm-docs]: https://realm.io/docs/swift/latest/
[store]: https://itunes.apple.com/pl/app/water-intake-drink-more-water/id1062053347?mt=8
