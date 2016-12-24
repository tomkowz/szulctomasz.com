---
layout: post
title: "Xcode: Moving view in storyboard and keeping it on the same level in hierarchy"
tags: ios, xcode
id: post-34
redirect_from: "/xcode-moving-view-in-storyboard-and-keeping-it-on-the-same-level-in-hierarchy/"
---
I am playing a bit with storyboards and was thinking if there is a way to move
view without changing its level in views hierarchy. And, there is one.

**Following case:** I have a view controller which has its view which is container for other views
obviously. I added another view (called <em>board</em>) which cover entire space
and added 2 inside this <em>board</em> view. I added a label to the view controller's
main view and wanted to keep it on this level when moving it around. The natural
thing is that when you start moving this label, it causes that one of views below
changes its color to blue which indicates that when you drop the label it will
be added as subview of the blue-highlighted view.

You can avoid this by pressing Command (⌘) key during dragging and dropping.

Here is a screen recording that presents how it works:

<iframe width="560" height="315" src="https://www.youtube.com/embed/tGENAJ8wA54" frameborder="0" allowfullscreen></iframe>

Before I found this trick I always selected the view and modified X and Y
position of the view using the Size Inspector. The new way I found is better and faster.
