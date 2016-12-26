---
layout: post
title: "Finding contours on paper sheet scans"
tags: osx, machine learning, swift
post_id: post-51
excerpt: "First step to handwritten digits recognition - collecting data from paper sheet scans"
---

*This article covers what data do we need to collect to learn neural network to recognize
handwritten digits and how to collect that data using OS X app written in Swift.
This can be however easily ported to iOS, so if you're not familiar with OS X
you can find many useful informations anyway.*

---

Severals months back I decided to get deeper understanding of what machine
learning is, how to create neural networks and how to recognize handwritten digits.

The Machine Learning (ML) domain is broad topic and is getting more and
more popular each year. Big companies that has some photos hosting or photos
sharing services uses neural networks to recognize people faces to create albums
where you can find photos of just one person. Some services uses machine learning
to learn when to show you an ad on their page, to make sure you'll be willingly
clicking it or at least reading. Siri uses it too. There is a lot of use cases
nowadays where neural networks exist.

Today, I am several weeks after I finished [Machine Learning by Stanford University][course]
course. This helped me understand basic stuff of ML and neural networks.

Before I start working on such neural network, I wanted to fetch
data of digits written by me first and use it for feeding neural network (NN)
and test it later also using my digits.

To learn NN how your digits look like you need to feed it with a lot of examples
of a single digit.

### Collecting data
I started of collecting 100 examples of each digit (0-9). The next step is
to get this data out of the paper sheet scans, cut the digits out of the scans and store in
files that share the same dimensions. Later on these small images representing
one digit will be converted to more useful data for the NN. The image will be
represented by a MxM matrix that contains information about every pixel of
these images with one digit. The individual item in a matrix is an argument of
a function that NN will use to learn. So, having 20x20 pixels images will end
up of having function that will take 400 parameters. The bigger the picture is,
the more details it has, the longer time will it take to learn.

Getting 100 examples of each digits is a lot of writing.. As a software dev I
am not typing too often using a pen, so, it is quite time consuming and my hand
is hurting now :) However, to get more examples you can take the scanned digits
and e.g. rotate them to create new examples.

![preparing-data][picture-1]

### Contours detection - idea
To do a simple contours recognition and prepare images for later use we
need to think about 3 things - **size of a slice** (group of pixels) of a scan
you'll be examinating to have or not to have an interesting information, **threshold**
that will decide whether examinated data is the interesting one or not
(everything above threshold is not interesting), and **size of an output images**
which is important for your future neural network.

Okay, we've got all this parameters but what next? Now, algorithm need to loop over
all pixels divided by slices of width (square is fine for this) you chose,
and if there is at least one interesting pixel in this slice the algorithm stops examinating this slice and start checking neighborhood for other interesting slices. After it
found first interesting slice it has to group all neighbor slices because it
will be a contour we're looking for. Following sketch presents roughly what's
going on.

![detecting-1][detecting-1]

Slices with solid line borders are ones with no contours inside.
At some point the algorithm will find a slice with dotted borders **(1)**.
Then it creates a group and start examinating neighborhood for another dotted-border slices
and group them together **(2)**. After there is no interesting slices to collect,
it stops searching for this specific contour and stores it for later.
Every time slice is checked the algorithm marks it as visited to not visit it again.

Having group of slices you can calculate an area that will cover all of them.
This area is the area of the contour on the image. The area is used to cut the
contour from the original image and later saved on disk.

Having such group of slices you can also draw preview of what algorithm found
and whether it correctly calculated area and whether it missed some slices or not.
This stuff turned out to be really useful during development.

### App
I decided to go with OS X app this time. I did it because of the Mac performance.
I wanted to detect contours quickly and have them saved on my Mac's disk.

All this led me to following UI
![app-screnshot][app-screenshot]

I'll not cover code of the view controller or UI logic in this post. Only
snippets of algorithm will be covered.

### Loading image
What we need to do first is to provide path to the image file on disk and load
it as CGImage for later processing.

{% highlight swift %}
let fileSystemRep = (path as NSString).fileSystemRepresentation
if let dataProvider = CGDataProviderCreateWithFilename(fileSystemRep) {
    if let imageSource = CGImageSourceCreateWithDataProvider(dataProvider, nil) {
        if let img = CGImageSourceCreateImageAtIndex(imageSource, 0, nil) {
            return img
        }
    }
}
{% endhighlight %}

### Getting pixel data
After image is loaded we have to obtain pixel data of the image that we can
search through later. This can be easily achieved using Core Graphics functions.

{% highlight swift %}
let provider = CGImageGetDataProvider(cgImage)!
let data = CGDataProviderCopyData(provider)
self.dataPtr = CFDataGetBytePtr(data)
self.length = CFDataGetLength(data)

let bytesPerRow = CGImageGetBytesPerRow(cgImage)
self.bytesPerComponent = bytesPerRow / CGImageGetWidth(cgImage)
{% endhighlight %}

Now, **dataPtr** is an 1 dimensional array with pixel data that you can access
by index as normal array. You have to know width and height of an image to calculate
the position of the pixel in array.

### Processing pixel data
*The easiest type of image to process and get contours from is the black-and-white
image. If you have color one you should consider converting it to b&w first, or
your algorithm will have to do that.*

The easiest way to find an interesting slice is to check whether it has dark pixel.
So, as we have RGBA pixel data now, we need to get light of all the pixels and use
such data with light information only. All we need to do is to convert RGB pixel
to HSL and take only L.

{% highlight swift %}
for i in 0.stride(to: totalBytes, by: 4) {
    let r = Int(originalData.byte(i + 0))
    let g = Int(originalData.byte(i + 1))
    let b = Int(originalData.byte(i + 2))

    let maxVal = max(r, max(g, b))
    let minVal = min(r, min(g, b))

    let lightInt: Int = (maxVal + minVal) / 2
    var light = UInt8(max(0, min(lightInt, 255)))

    light = light < threshold ? 0 : 1

    CFDataAppendBytes(newData, &light, 1)
}
{% endhighlight %}

Here the algorithm uses **threshold** parameter we discussed above. If light value
of a pixel is below the threshold you can set 0, otherwise set 1 (or whatever you want).

We can now feed our contour finder with this processed data.

### Finding contour
We know image size, we have light pixels data, so we can search through by creating
slices of an image and examine groups of pixels. If there is pixel data with 0 light
value, it means this slice has a contour and we should group it.

{% highlight swift %}
func isContourAt(x x: Int, y: Int) -> Bool {
    return data.byte((y * Int(imageSize.width)) + x) == 0
}
{% endhighlight %}

{% highlight swift %}
for col in 0.stride(to: Int(imageSize.width - sliceSize.width), by: Int(sliceSize.width)) {
    for row in 0.stride(to: Int(imageSize.height - sliceSize.height), by: Int(sliceSize.height)) {

        // create slice for examinated position
        let rect = CGRect(x: col, y: row, width: Int(sliceSize.width), height: Int(sliceSize.height))

        let startingSlice = Slice(rect: rect, delegate: self)

        // create temporary contour and try to reveal it
        var sliceGroup = SliceGroup()
        searchNeighborhood(&sliceGroup, slice: startingSlice)

        // if examinated area contains contours you can store it
        if sliceGroup.slices.count > 0 {
            let contour = Contour(rect: sliceGroup.rect, sliceRects: sliceGroup.sliceRects)
            delegate?.finderDidFind(contour, originalImageSize: imageSize)
        }
    }
}
{% endhighlight %}

Having one such slice we start searching recursively until we do not find any
slices with contour inside.

{% highlight swift %}
// Do not proceed with checked slice
if visited.isVisited(sl.rect) {
    return
}

// Mark this slice as checked
visited.markAsVisited(sl.rect)

// Do not check neighborhood of this slice if it has no contour.
if sl.hasContour() == false {
    return
}

// Part of a contour found in this slice, store it and search further.
contour.add(sl)

// Calculate boundary of the neighborhood
let frCol = max(0, sl.rect.minX - sliceSize.width)
let toCol = min(sl.rect.minX + 2 * sliceSize.width, imageSize.width - sliceSize.width)

let frRow = max(0, sl.rect.minY - sliceSize.height)
let toRow = min(sl.rect.minY + 2 * sliceSize.height, imageSize.height - sliceSize.height)

// Iterate through neighborhood slices
for col in frCol.stride(to: toCol, by: sliceSize.width) {
    for row in frRow.stride(to: toRow, by: sliceSize.height) {
        // do not check the center slice (the same as `sl`)
        if row == sl.rect.minX && col == sl.rect.minY {
            continue
        }

        let rect = CGRect(x: col, y: row, width: sliceSize.width, height: sliceSize.height)
        let nearSlice = Slice(rect: rect, delegate: self)

        // Check its neighborhood
        searchNeighborhood(&contour, slice: nearSlice)
    }
}
{% endhighlight %}

*In the same time you're examinating slices you have to remember to mark them as visited
to not make your algorithm running forever. Also, remember to calculate the common area
for all grouped slices.*

### Cutting and resizing contours
After contour is found algorithm uses **output size** parameter defined above
to resize contour and save it on disk.

Cutting part is easy and look like below
{% highlight swift %}
let rect = contour.rect
if let subImage = CGImageCreateWithImageInRect(image, rect) {
    let sourceImage = NSImage(CGImage: subImage, size: rect.size)
    return ImageResizer.resize(sourceImage, toSize: outputSize)
}
{% endhighlight %}

Image we cut might have different size that defined output size.
Resizing it consists of finding longer edge and resizing it to desired one by
keeping ratio of cutted piece of a scan.

{% highlight swift %}
// source - cutted image with contour
// resultSize - output size

if source.size.height > source.size.width {
    let ratio = source.size.height / source.size.width
    drawRect.size.height = resultSize.height
    drawRect.size.width = drawRect.size.height / ratio
} else {
    let ratio = source.size.width / source.size.height
    drawRect.size.width = resultSize.width
    drawRect.size.height = drawRect.size.width / ratio
}
{% endhighlight %}

Later the contour have to be centered on the output image.
{% highlight swift %}
drawRect.origin.x = (resultSize.width - drawRect.size.width) * 0.5
drawRect.origin.y = (resultSize.height - drawRect.size.height) * 0.5
{% endhighlight %}

Then algorithm creates image with white background and draws contour on it.

### Preview of what algorithm found
As I mentioned above it is useful to see what algorithm sees. Having all the
information above contours and slices the preview can be drawn.

{% highlight swift %}
CGContextSetLineWidth(context, 1.0)

CGContextTranslateCTM(context, 0, CGFloat(imageSize.height));
CGContextScaleCTM(context, 1.0, -1.0)

for contour in contours {
    // draw contour's rect
    CGContextSetStrokeColorWithColor(context, NSColor.blueColor().CGColor)
    for slRect in contour.sliceRects {
        // draw slice rect
        CGContextStrokeRect(context, slRect)
    }

    CGContextSetStrokeColorWithColor(context, NSColor.redColor().CGColor)
    CGContextStrokeRect(context, contour.rect)
}
{% endhighlight %}

The interesting part of generating such preview is that we can see how algorithm
accuracy increases when slice width gets smaller.

Following animation presents how algorithm accuracy look like on 540x562 image with
100 ones on it. Width of a slice is 20 and goes down to 2. The smaller it gets you
can see that slices forms digit one.

![animation][animation]

Not always the smallest size of a slice is the best option to follow.
The drawbacks are the algorithm gets slower and slower and can even grab some
dots or other small stains from a scan and interpret them as contours. We could have another
parameter that would say whether contour should be omitted or not.

### Performance
At some point of development the algorithm ended up in 30 seconds to detect contours on 540x562 image
with width slice of 5px. As [Piotr Tobolski][li-piotr] pointed out it was too slow for such
a small number of operations - about ~600,000 operations (preparing data and examining pixels).

The thing I didn't remember of was to change **Optimization Level** of
**Swift Compiler Code Generation** from *None* **[-Onone]** to *Fast* **[-O]**.
After I changed it to *Fast* I started getting 1.22 seconds for the same operations.

### Wrap-up
This is the first post of series about machine learning. As I finished the
online course that took me few months I would like to start
learning more in this area and start experimenting.

The algorithm accuracy for iPhone photos works pretty well but I believe scans
would work even betters. The photos are more grey-and-black than white-and-black.
You can play with *threshold*, edit photo in Preview.app to make it more white-and-black
or just scan it and it should be ready to use.

Here we covered how you can get a handwritten
digits data to feed the neural network later. If you'd like to learn what machine
learning is and how to start, you can go to the [coursera course][course] and
after you finished it, you can take a look on [Neural Networks and Deep Learning][book]
book that covers a lot of details about neural networks for digits recognition.

[book]: http://neuralnetworksanddeeplearning.com/index.html
[course]: https://www.coursera.org/learn/machine-learning
[picture-1]: /uploads/{{page.post_id}}/picture-1.png
[app-screenshot]: /uploads/{{page.post_id}}/app-screenshot.png
[detecting-1]: /uploads/{{page.post_id}}/detecting-1.png
[animation]: /uploads/{{page.post_id}}/animation.gif
[li-piotr]: https://pl.linkedin.com/in/piotrtobolski
