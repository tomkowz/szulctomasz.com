---
layout: post
title: "iOS 9: Getting single location update with requestLocation()"
tags: ios, ios9, swift, core location
post_id: post-22
redirect_from: "/ios-9-getting-single-location-update-with-requestlocation/"
---
Apple added new API to CoreLocation framework. One of the available methods
is `requestLocation()` which allow to get single location update.

This is nice for apps that need just a quick fix to work. You can call the
method and in few seconds get user's location and location manager automatically
stops.

{% highlight swift %}
import UIKit
import CoreLocation

class ViewController: UIViewController, CLLocationManagerDelegate {

    private var locationManager = CLLocationManager()

    override func viewDidLoad() {
        super.viewDidLoad()
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
    }

    @IBAction func getFixPressed(sender: AnyObject) {
        locationManager.requestLocation()
    }

    // MARK: CLLocationManagerDelegate
    func locationManager(manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        print(locations)
    }

    func locationManager(manager: CLLocationManager, didFailWithError error: NSError) {
        print(error)
    }
}
{% endhighlight %}

To use it you must implement `locationManager:didUpdateLocations:` and
`locationManager:didFailWithError:` methods of `CLLocationManagerDelegate`.
Failing method is called when location manager cannot get update or when
permissions are not granted or when location cannot be determined in some
time (managed by the system).

### Wrap-up
This is nice addition to Core Location. Unfortunately it is available only in
iOS 9. Code is cleaner and you can not worry about stopping location updates.
