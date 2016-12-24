---
layout: post
title: "Thoughts and impressions after creating and submitting 2048 game in 10 hours"
tags: ios, swift, thoughts
id: post-27
redirect_from: "/thoughts-and-impressions-game-submitted-in-10-hours/"
---
That was pretty effective Saturday. Writing game was an interesting process and
I played a lot 2048 that day. Here are my thoughts and impressions.

I spoke with my colleague about his job interview challenge he got from one of
employers from California and it was about writing 2048 game. Actually two guys
did it, one in 8 hours and my colleague in 17 hours - both with different
approaches in Objective-C.

I got inspired. The challenge might be good opportunity to check my Swift skills
and to make me more confident about it, because <em>I am thinking about relocating
from Poland to San Francisco Bay Area next year and will be searching for a
job - Want to help me? Or Hire?</em> :)

Anyway, I decided to spend Saturday working on the app. But on entire app, not
just a module to play like in the original challenge. And I planned to do it in
Swift because I'm in love with Swift these days and wanted to examine my
Swift skills :)

I did it in Swift 1.2 and Xcode 6.4 to be able to submit the app to the store.

### Repository and the app
The project is [available on GitHub][gh]. Feel free to contribute if you find
something that is worth to change/improve. I am always open to new ideas.

You can find the app on the App Store - [2048 - Flat Color Version][store]. Enjoy it!

### Planning UI
I spent about hour of Friday evening drawing some UI in [Sketch][sketch] app.
This is I think 3rd or 4th app that has been designed in Sketch by me - it is
really great, I recommend it not only for designing apps, it is also great
in quick prototyping and has many plugins hosted on GitHub. People care about
it so much - This is not an ad, just advice :)

The UI had to be simple, just colorful tiles on some darker background.
I think it looks nice - I am not UI/UX master, just software engineer with some
background in 2d/3d graphics :)

![image-1][img-1]

### Stop and think a while
Thinking about UI before implementing anything related to UI is a big
thing - it is a time-saver. If you're planning to do some UI component or entire
screen in the app, something complex, just draw it first, play it on screen,
maybe show to someone and then when you're done with prototyping you can start
coding. I think I saved a lot of time by drawing UI before starting coding.
I had some experience with previous projects when I was prototyping and coding
at the same time and it took much more time than it should.

### Assets
I've decided to export what I can to PDF files so I could work with vectors.
I recommend using PDFs - you can keep just one file of an asset instead of one
per screen scale/size. I wrote about it once in
[PDF vectors in Assets catalog][post-1] - go and read, I'll wait.

### Architecture
App is targeted to iOS 8 and newer so Launch Screen was a piece of cake as well
as adding icon and implementing first screen with "new game" button.

I read a bit about [MVVM][mvvm] some time ago and wanted to use it, at least to
some extent - It was more MVCVM (Model View Controller View Model). I didn't want
to use Reactive Cocoa and similar things, I just created and used View Models to
keep code that was responsible for processing/formatting content to be displayed
out of view controllers. The game's UI is simple so there are just few View Models,
but in bigger project I worked on it is working great - highly recommended.

### Data Structure
This is something I am proud of. I spoke with colleague after I finished the game
and he told me the approach is really nice. I was writing this in Swift and I
wanted to keep it really, really simple.

There are always 16 elements on the screen, Meaning, the board is 4 by 4 and it
has 16 slots for tiles. So I thought about the board like about matrix that has
always 16 slots and that values are appearing, disappearing and transitioning
between these slots. And I've created something like 2048-data-structure for it.

{% highlight swift %}
import Foundation
import CoreGraphics

func == (lhs: Position, rhs: Position) -> Bool {
    return lhs.x == rhs.x &amp;&amp; lhs.y == rhs.y
}

struct Position: Equatable {
    var x, y: Int

    var CGPointRepresentation: CGPoint {
        return CGPointMake(CGFloat(x), CGFloat(y))
    }
}

class Tile {
    let position: Position
    var value: Int?

    var upTile: Tile?
    var rightTile: Tile?
    var bottomTile: Tile?
    var leftTile: Tile?

    init(position: Position, value: Int? = nil) {
        self.position = position
        self.value = value
    }
}
{% endhighlight %}

Every tile has its own constant position and it always knows its neighbor tiles.
Doing this helped me a lot when I was writing logic for shifting tiles/values.

### Logic and its view representation
I wanted to keep game logic out of UI. It had to be very simple class that contains
all the logic and knows only about the data structure on the board and it does not
even know what `UIKit` is. I've implemented it this way that there is a class
named `GameLogicManager` which contains the whole logic and a `GameBoardRenderer`
class which is responsible for presenting the visual part of the game on screen.
It works well, at least in this case. I am wondering if this is known/popular and
good solution or not. Anyway, it fits nice to this project.

So, `GameLogicManager` calls delegate (`GameViewController`) and "saying"
*"move tile from position X to Y"*, *"move tile from X onto Y"*
(to combine with another tile) or just *"game over"* or *"game win"*.

Messages" were passed to `GameBoardRenderer` which has array with `TileView`
elements and reference to `GameBoardView` and was responsible for moving
(animating) them on the screen.

The method that is responsible for shifting tiles on the screen took 71 lines
with comments - 56 without comments - not too long. This was the hardest thing
to implement and I wanted to keep it well commented and implemented in very simple
way. And simple it is :) It handles shifting in 4 directions as game needs.
This method was a real pain in a butt and took almost 6 hours - I've tried few
approaches before I found best solution.

{% highlight swift %}
func shift(direction: ShiftDirection) {
    // Want to wait for ending one shifting before doing another which
    // can collide with currently performing.
    if updating == true { return }
    updating = true

    var performedShift = false
    for rowOrColumn in 0..<boardWidth {
        // Get all tiles in a row or column.
        var tilesToCheck = tiles.filter {
            return ((direction == .Right || direction == .Left) ? $0.position.y : $0.position.x) == rowOrColumn
        }

        // When shifting right or down array need to be reversed.
        if direction == .Right || direction == .Down {
            tilesToCheck = reverse(tilesToCheck)
        }

        var tileIndex = 0
        while tileIndex < tilesToCheck.count {
            let currentTile = tilesToCheck[tileIndex]
            // Find first tile with some value. When shifting right filter
            // seeks for tiles on the left, otherwise on the right from current tile.
            let filter: ((tile: Tile) -> Bool) = { tile in
                let position: Bool = {
                    switch direction {
                    case .Up: return tile.position.y > currentTile.position.y
                    case .Right: return tile.position.x < currentTile.position.x
                    case .Down: return tile.position.y < currentTile.position.y
                    case .Left: return tile.position.x > currentTile.position.x
                    }
                }()

                return position == true &amp;&amp; tile.value != nil
            }

            if let otherTile = tilesToCheck.filter(filter).first {
                // If value is same increase value of current tile and
                // remove value from other tile.
                if otherTile.value == currentTile.value {
                    moveOnSameTile(otherTile, onTile: currentTile)
                    // Notify about additional points because of adding up values
                    pointCount += currentTile.value!
                    // If current tile get's win tile value just end the game.
                    if currentTile.value! == winTileValue {
                        delegate?.gameLogicManagerDidWinGame()
                        return
                    }
                    delegate?.gameLogicManagerDidCountPoints(pointCount)
                    performedShift = true
                } else if currentTile.value == nil {
                    moveOnEmptyTile(otherTile, destinationTile: currentTile)
                    // if tile has been moved to another place repeat this step
                    // because maybe next tile has the same value and they
                    // should be added up.
                    tileIndex--
                    performedShift = true
                }
            }
            tileIndex++
        }
    }

    // Every shift method returns boolean value if shift has been performed on
    // some at least one tile or not.
    if performedShift {
        delegate?.gameLogicManagerDidAddTile(addRandomTile())
    }

    if isGameOver() == true { delegate?.gameLogicManagerDidGameOver() }
    updating = false
}
{% endhighlight %}


The rendering part was very simple and communication between logic manager
and renderer too - just look into repository for details. In the last three
hours I've finished the "Game Over" / "Game Win" screen, storing/loading high
score and submitted the app to the Store :)


### Conclusion
I am happy to finish the game in 10 hours. I think it is pretty good time and
doing this in Swift makes me more confident about my level of understanding
the language.

The challenge gave me also another thoughts about how important prototyping/problem
analysis before implementation is - It could be a big time-saver - It was something
like research, right?

IMO I could finish the shifting method a bit faster, like about hour faster, if
I would spend 30 minutes more on thinking about the problem with paper on desk
and pen in a hand, instead of testing solutions that I had in my head without
testing them on a paper.

[gh]: https://github.com/tomkowz/2048
[store]: https://itunes.apple.com/us/app/2048-flat-color-version/id1023137898?ls=1&mt=8
[sketch]: http://www.bohemiancoding.com/sketch/
[post-1]: http://szulctomasz.com/xcode-pdf-vectors-in-assets-catalog/
[mvvm]: https://en.wikipedia.org/wiki/Model_View_ViewModel

[img-1]: /uploads/{{page.id}}/1.png
