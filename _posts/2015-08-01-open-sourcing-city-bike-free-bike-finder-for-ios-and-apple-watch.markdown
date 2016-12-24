---
layout: post
title: "Open Sourcing City Bike, Free Bike Finder app for iOS and Apple Watch"
tags: ios, swift, citybike, open source, watchos
id: post-24
redirect_from: "/open-sourcing-city-bike-finder/"
---
I was working on [City Bike][store] app for some time recently. Today I am
excited to announce that I am making it open source project and the code
is [available on GitHub][gh].

The project is written in Swift 1.2 in Xcode 6.4 because of ability to submit
app to the App Store. The app is powered by [citybik.es API][api] - App uses
version 2 of the API, and there is version 3 on the way.

I written the app to learn how to develop apps on iOS and watchOS devices. I've
learned a lot, I had few rejections and had to fix some issues but now I see
that at least Apple tests manually apps on the Apple Watch - which is great of
course. I think they care about Apple Watch, at least for now when platform
is growing and started not that long ago.

### Why I made it open source?

1. I think sharing knowledge is very important and people can learn from other
projects, from reading the code and participating in software development.
Contribution to this project is great idea if you want to learn something new
about Swift and watchOS
2. This is great opportunity for me to learn something new from other people.
3. I'd like to move the app over to Swift 2 and watchOS 2.
4. There are some ideas I would like to realize in this project.
There are some issues to fix but also new features to implement - [Issues][issues].
5. Deployment target for the project is set to iOS 8.0 so app supports two latest
system version. It makes development process easier.

I made it free because I want people to use it. I made it open source because I
would like to make a great and useful app that people will use. There are many
bike networks over the world and will be more and maybe when owners of these
networks see that a lot of people is using such app they will share its data
more willingly - Here is the [video about necessity of sharing data about networks][video].
Now getting such data from some networks is difficult.

[store]: https://itunes.apple.com/pl/app/city-bike-find-free-bike/id1016094872?mt=8
[gh]: https://github.com/tomkowz/citybike-app
[api]: http://api.citybik.es/v2/
[issues]: https://github.com/tomkowz/citybike-app/issues
[video]: http://bambuser.com/v/2998385
