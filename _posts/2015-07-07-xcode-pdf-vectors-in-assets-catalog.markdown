---
layout: post
title: "Xcode: PDF vectors in Assets catalog"
tags: xcode
id: post-10
redirect_from: "/xcode-pdf-vectors-in-assets-catalog/"
---
Recently I realized that it is possible to work easier with Assets catalog in
Xcode 6/7. As I started searching the Internet I found that the feature I'm
using now is available since Xcode 6 has been released - PDF vectors. Don't
know how could I missed that.

In a project I recently finished and waiting for review approval I found that
this is possible to use PDF in Assets catalog - And this is awesome.

You just put PDF in Assets and mark it as vector graphic and there is only one
"socket" for putting asset called **All**, instead of using **@1x, @2x, @3x**.
Xcode will rasterize this PDF to the correct resolutions and scales at build time.

I am using [Sketch][sketch] to draw UI and here it is how my export settings
looked like before and after using PDFs:

![image-1][img-1]

After assets are imported to Assets catalog the only thing to do is to mark it as vector so Xcode knows what to do with this PDF.

![image-2][img-2]

And the best part is how it looked before and now with PDFs.

![image-3][img-3]

[sketch]: http://www.bohemiancoding.com/sketch/

[img-1]: /uploads/{{page.id}}/1.png
[img-2]: /uploads/{{page.id}}/2.png
[img-3]: /uploads/{{page.id}}/3.png
