---
layout: post
title: "iOS: Searching with UISearchController"
tags: ios, swift, uisearchcontroller
id: post-14
redirect_from: "/ios-searching-with-uisearchcontroller/"
---
iOS 8 brought a lot of new APIs for developers. One of them is new API used for searching. Before iOS 8 we're using `UISearchDisplayController`. Since iOS 8
there is new [UISearchController][uisearchcontroller] available - let's play
with it for a moment and implement in [Quotes][quotes] app.

**If you're not familiar with [Quotes][quotes] yet, please read this [brief introduction][previous-post].**

The new API requires to implement `UISearchBarDelegate` for dealing with
`UISearchBar`, `UISearchControllerDelegate` for the new search controller
(all of them are optionals) to react when search controller will/did present
or dismiss and finally `UISearchResultsUpdating` protocol which has method
that is called when search bar text changes or when search bar becomes first
responder.

![image-1][img-1]

Before iOS 8 it was possible to drag and drop *"Search Bar and Search Display Controller"* on a table view and the controller was ready to use, except writing filtering and telling when to search. Unfortunately, or fortunately
(I like this new one) this property has been deprecated and probably will be
obsolete in iOS 10.

{% highlight swift %}
extension UIViewController {
    @available(iOS, introduced=3.0, deprecated=8.0)
    var searchDisplayController: UISearchDisplayController? { get }
}
{% endhighlight %}

In the past I dealt with `UISearchDisplayController` and used the same `UITableViewDelegate` and `UITableViewDataSource` protocols implementation for
search controller and main table view from the class where table view was
contained. There was also code responsible for entries filtering. The code was
in one view controller and was very messy. I didn't work out better solution or
flow - I think there are few. Using `UISearchController` code can be easily
decoupled and easy to maintain.

The first thing to think about is what will be displayed in main table view and
what in filtered one. Let's assume that in Quote app we want to display quotes
in both main and filtered table. So the same cell can be used. Let's detach it
to separated xib and create the class.

{% highlight swift %}
import UIKit

class QuoteTableViewCell: UITableViewCell {
    static let Identifier: String = "QuoteTableViewCell"

    @IBOutlet private var quoteLabel: UILabel!
    @IBOutlet private var authorLabel: UILabel!

    var viewModel: QuoteViewModel! {
        didSet {
            quoteLabel?.text = viewModel.content
            authorLabel?.text = viewModel.author
        }
    }
}
{% endhighlight %}

Cells and table views are the same. The good practice is to share common code
to avoid repeating. Let's create `QuoteListBaseTableViewController` which holds
base informations about table view and cells - this class will be also used as a
search results controller. And `QuoteListViewController` class - it inherits
from the base, contains search bar and main table with quotes.

{% highlight swift %}
class QuoteListBaseTableViewController: UITableViewController {
    var quotes = [Quote]()

    override func viewDidLoad() {
        super.viewDidLoad()
        // register cell nib
        let nib = UINib(nibName: QuoteTableViewCell.Identifier, bundle: nil)
        tableView.registerNib(nib, forCellReuseIdentifier: QuoteTableViewCell.Identifier)

        // set that table view has dynamically self-sized cells
        tableView.estimatedRowHeight = 40
        tableView.rowHeight = UITableViewAutomaticDimension
    }

    // Table view code is the same in the base version.
    // The same number of elements because we want to display all we've got.
    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return quotes.count
    }

    // Created cells will be the same, there is no reason for not keeping it here :)
    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCellWithIdentifier(QuoteTableViewCell.Identifier, forIndexPath: indexPath) as! QuoteTableViewCell
        cell.viewModel = QuoteViewModel(quote: quotes[indexPath.row])
        return cell
    }

    override func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
        tableView.deselectRowAtIndexPath(indexPath, animated: true)
    }
}
{% endhighlight %}

Okay. Now is the time to create search controller, results table controller and
results updater which is responsible for filtering content.

{% highlight swift %}
class QuoteListViewController: QuoteListBaseTableViewController, UISearchBarDelegate, UISearchControllerDelegate {
...
private var searchController: UISearchController!
private var resultsTableController: QuoteListBaseTableViewController!
private var resultsUpdater = QuoteResultsUpdater()

// additional code for better user experience
private var searchControllerWasActive = false
private var searchControllerSearchFieldWasFirstResponder = false

// when qoutes changes in our controller we want to update results updater to use fresh data.
override var quotes: [Quote] {
    didSet {
        resultsUpdater.quotes = quotes
    }
}
...
}
{% endhighlight %}

And here search controller is configured:
{% highlight swift %}
override func viewDidLoad() {
    super.viewDidLoad()
    configureSearchController()
}

private func configureSearchController() {
    // Create results table controller and search controller
    resultsTableController = QuoteListBaseTableViewController()

    // Create search controller and assign updater to it.
    searchController = UISearchController(searchResultsController: resultsTableController)
    searchController.searchResultsUpdater = resultsUpdater

    // set other properties, UI, delegates
    searchController.searchBar.searchBarStyle = UISearchBarStyle.Minimal
    tableView.tableHeaderView = searchController.searchBar
    resultsTableController.tableView.delegate = self
    searchController.dimsBackgroundDuringPresentation = false
    searchController.searchBar.delegate = self
    self.definesPresentationContext = true
}
{% endhighlight %}

I kept entire implementation in view controller with main table view and realized
it was not necessary and finally detached results updater to separated file to
keep view controller as clean and simple as possible.

Implementation of results updater is easy. There is only one protocol and one
method to implement. Method is called when text changes in search bar. It is
responsible for parsing search text and creating predicates for filtering.
At the end it changes content of search results controller and calls `reloadData()` on its table view.

{% highlight swift %}
import Model
import UIKit

class QuoteResultsUpdater: NSObject, UISearchResultsUpdating {
    var quotes = [Quote]()

    func updateSearchResultsForSearchController(searchController: UISearchController) {
        let searchText = searchController.searchBar.text?.stringByTrimmingCharactersInSet(NSCharacterSet.whitespaceCharacterSet()) ?? ""
        var searchItems = [String]()
        if searchText != "" {
            searchItems = searchText.componentsSeparatedByString(" ")
        }

        var andMatchPredicates = [NSCompoundPredicate]()
        for searchItem in searchItems {
            var subPredicates = [NSPredicate]()
            subPredicates.append(NSPredicate(format: "content CONTAINS[c] %@", searchItem))
            subPredicates.append(NSPredicate(format: "author CONTAINS[c] %@", searchItem))
            let orMatchPredicate = NSCompoundPredicate.orPredicateWithSubpredicates(subPredicates)
            andMatchPredicates.append(orMatchPredicate)
        }

        let finalCompoundPredicate = NSCompoundPredicate.andPredicateWithSubpredicates(andMatchPredicates)
        let filteredQuotes = (quotes as NSArray).filteredArrayUsingPredicate(finalCompoundPredicate) as! [Quote]

        let tableController = searchController.searchResultsController as! QuoteListBaseTableViewController
        tableController.quotes = filteredQuotes
        tableController.tableView.reloadData()
    }
}
{% endhighlight %}

I personally like the new API.

[uisearchcontroller]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISearchController/
[quotes]: https://github.com/tomkowz/Quotes
[previous-post]: http://szulctomasz.com/starting-project-in-swift-2-for-ios-9-8-features-testing/

[img-1]: /uploads/{{page.id}}/1.png
