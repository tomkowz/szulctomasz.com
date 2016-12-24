---
layout: post
title: "Redesigned blog and changed to Jekyll"
tags: general
id: post-48
excerpt: "Migrated the blog from Wordpress to Jekyll."
---
I thought about this migration for a bit last few months, but never had
a time to perform it. I observed what people are using, and wanted to decrease
amount of money I spend for hosting and to make my life easier.

After quick research and thinking what I need, I decided to go with [Jekyll][jekyll].

*If you're planning to use static site generator, read this post and maybe you'll
pick Jekyll too.*

#### Why Jekyll you ask?
I know there are several static site generators like this one but I picked
Jekyll because this is the only one official tool supported by Github.
Other tools works with Github too, like [Middleman][middleman]. I just... wanted
to have really easy migration and do not spent more time than needed.

- Hosting page on Github will allow me to drop my current hosting.
- I don't like Wordpress articles editor. I always wanted to use markdown and write
articles in [Atom][atom] which is great app btw! I can recommend it to everyone - Dropped
*Sublime Text*, *Mou* and *TextWrangler* for it. Now I can use markdown and this
is really great, articles look clear during writing and easy to maintain.
- I can host blog on github, so I have source control management.
- I do not need a big solution like Wordpress - I want to keep my blog simple.
- I played a lot with Python last month, working on RESTful service and frontend using [Flask][flask].
It uses [Jinja2][jinja2], and Jekyll also uses Jinja2 templating system, which
I like - It is easy to do frontend.
- It is easier for me to create new pages, like page with list of all articles on
the blog, or... okay, I have only just this one for the moment :)
- Syntax highlighting is better than the Crayon on Wordpress.
- There is no html tags floating around in the article, e.g. inline or not inline
syntax highlighting code, or links, or images - oh, I hate this!

And there is more things that I don't want to think about now.

![atom-screenshot][img-1]

#### Migration process
It took me ~10-12 hours to move 47 articles with images and links, and to convert
everything to markdown, and update a bit default Jekyll's theme.

After I moved everything manually I found there is a [migration tool][migration-tool] that allow
you move your wordpress data to Jekyll. My bad, but now I do know very well
how to create posts on Jekyll ;)

Jekyll is nicely documented so I found answers for probably every of my questions
in [official documentation][jekyll-docs].


#### Migration is done
That's it. Blog based on Jekyll is up and running.
Good, now I just need to write posts on the blog.

[atom]: https://github.com/atom/atom
[flask]: http://flask.pocoo.org
[jekyll]: https://github.com/jekyll/jekyll
[jekyll-docs]: https://jekyllrb.com/docs/home/
[jinja2]: http://jinja2.readthedocs.org/en/latest/
[middleman]: https://github.com/middleman/middleman
[migration-tool]: https://import.jekyllrb.com/docs/wordpressdotcom/

[img-1]: /uploads/{{page.id}}/1.png
