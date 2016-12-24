---
layout: post
title: "iOS: 3D Touch, impressions and thoughts"
tags: ios
id: post-37
excerpt: "Few thoughts about 3D Touch."
redirect_from: "/3d-touch-impressions-and-thoughts/"
---
Hey, we've got 3D Touch in new iPhones and iOS 9. This is a great piece of
technology. Let's take a look what it brings to us developers and users.
Here are my thoughts and listed things that we should care about.

### Initial problems
I wanted to start doing some "getting started" articles about 3D Touch in apps
but there is one problem with this - I and no one is able to simulate 3D Touch
in simulators and Xcode 7. Not possible at all.

Apple Says:

> With Xcode 7.0 you must develop on a device that supports 3D Touch. Simulator
in Xcode 7.0 does not support 3D Touch.

Sad, but true. I thought they'll figure out something to simulate it. Probably
they'll solve it in next Xcode release, I hope.

Another thing is that new iPhone is not available here in Poland - will be
soon - maybe in 2 weeks.

There is an option to get one -<em> I'll be in San Francisco and Palo Alto next
month, for entire month, starting 30 Sep, Yay!</em> And will probably buy one
new shiny iPhone. Not sure if 6s or 6s plus.

*I'll back for sure with more articles about 3D Touch when I get a device.*

### Peek and pop
This is the first feature that Apple introduced with 3D Touch - or should I say two features.

- A Peek lets user preview item without leaving current context. An item indicates
that it supports peek by displaying small rectangular view in a response to a light press.
- The view that displays on peek gesture should be large enough so user's finger
doesn't obscure the content and he can decide whether to press deeper to show the pop.
- A Pop is a detailed view of the item which is presented when user presses a
little deeper on the peek view.
- Even though the peek should give user enough information, you should always
let user transition to the pop. The pop should be the same view that user get
when he taps the item.
- Do not display buttons inside peek view. If user lifts finger in order to
tap a button the peek will disappear.
- Peek can provide quick actions related to item when user swipe up within the
peek view. When you want to provide some actions in the peek view, do it with
quick actions. You can add some quick actions to the peek view so user can
swipe up and select one of displayed actions.
- If you provide some touch-and-hold actions for an item, it is good practice
to provide the same actions within the peek that replaces this touch-and-hold gesture.
- If you want to use somewhere quick actions, peek and pop, remember to do a
check whether 3D Touch is enabled.
- Not every device supports peek and pop and 3D Touch may be off. Do not use
peek as only way to show item-specific actions. Try with some touch-and-hold
gesture and show a view with actions that user can choose when doing the gesture.

### Quick Actions
Those are the shortcuts presented when user presses app's icon deeply.

- Actions consists of title, icon and subtitle.
- They can display updated informations when app updates.
- Your app should provide at least one task in a Home screen, so user can perform
gesture on your app too and get some little nice shortcut.
- App can provide four quick actions at most.
- Do not use quick actions as a way to notify about update in the app or changes
or something like that. Notifications are proper way to do that.
- Quick action's name should be succinct and should have subtitle if necessary
and some icon. User should know what the action does. If you provide subtitle
the title will be one line long, and if not fit, the system will truncate it.
If subtitle is not provided an action's title will be wrapped to second line.

### Conclusion
3D Touch is very nice feature that provides a new way of interacting with a device.
There is also a Taptic Engine inside the device so we can get a feedback from
device during pressing the screen - Really nice, like it and can't wait to try.

I am sad that Apple didn't provide a way to simulate the gesture in the latest
Xcode 7 beta 2 but hope they fix the problem fast - or they increase incomes
because I am sure people want to play with it and need to improve their app
so they simply buy new device :D

I can't wait new apps based on 3D Touch like games, apps to sketch or draw,
some other kind of apps I can't even imagine. Maybe we can see fully working
scale app which can check weight of fruits or other items placed on the screen
or something like that? :>
