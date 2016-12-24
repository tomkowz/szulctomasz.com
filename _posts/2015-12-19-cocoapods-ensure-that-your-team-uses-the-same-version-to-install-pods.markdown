---
layout: post
title: "CocoaPods: Ensure that your team uses the same version to install pods"
tags: cocoapods
id: post-44
excerpt: "A trick that help you and your team in cocoapods install and updating process."
redirect_from: "/cocoapods-ensure-that-your-team-uses-the-same-cocoapods-version-to-install-pods/"
---
This post is an extended answer to [The Trick to Working with CocoaPods on a Team][natasha-post] post.
Jump to it, read, and go back here.

**Update 19/Dec/2015 6:41 PM**: Okay, Natasha helped me realize that my solution is a bit overkill.
I used the Gemfile in the project and I like it :) Instead of writing script
to check if CocoaPods exists and install them if not, I just created a script
with two commands she mentioned. So it look like this:

{% highlight ruby %}
bundle install
bundle exec pod install
{% endhighlight %}

And the Gemfile is just this

{% highlight ruby %}
source 'https://rubygems.org'

gem 'cocoapods', '~> 0.38.2'
{% endhighlight %}

So, yeah, this is what Natasha mentioned in her post :) Thanks Natasha!


**Original post**:
~~I think the better way to ensure that teammates use the same CocoaPods is not
to modify the Podfile, but to write simple bash script.~~
(I made a mistake. Natasha is using Gemfile, so Podfile is not modified).
~~Also, instead~~ Instead of manually call `bundle install` and `bundle exec pod install` (She uses Gemfile for that) I would like to let the script do a proper work for me - The Gemfile might be better if we have more gems to control.

Here is my suggestion for this.

{% highlight bash %}
#!/bin/bash

REQUIRED_COCOAPODS='0.38.2'

check_cocoapods_existance=$(gem list -i cocoapods --version $REQUIRED_COCOAPODS)

if [ $check_cocoapods_existance == "false" ]; then
	echo "CocoaPods $REQUIRED_COCOAPODS does not exist."
	echo "Installing CocoaPods $REQUIRED_COCOAPODS..."
	sudo gem install cocoapods --version $REQUIRED_COCOAPODS
fi

if [ $check_cocoapods_existance == "true" ]; then
	echo "Installing Pods..."
	pod "_"$REQUIRED_COCOAPODS"_" install
fi
{% endhighlight %}

1. The script do check whether required CocoaPods version is installed.
2. If CocoaPods in specified version does not exist, install it.
3. If CocoaPods is installed, install pods using this version.

Now you can save the script, call it setup.sh, and in terminal just execute `./setup.sh`.

### Conclusion
Never used such way to ensure CocoaPods integrity, and it was only me who
integrated and updated pods of a project. So I believe, I'll be using it with
my team since now.

[natasha-post]: http://natashatherobot.com/cocoapods-on-a-team/
