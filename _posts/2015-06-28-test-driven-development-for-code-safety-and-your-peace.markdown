---
layout: post
title: "Test Driven Development for code safety and your peace"
tags: ios, tdd
id: post-6
redirect_from: "/test-driven-development-for-code-safety-and-your-peace/"
---
Today I spent time working on one framework I would like to share end of this
week. It took me 8 hours and this were very peace, calm and nice hours of funny
coding today. Why you might ask? Because of unit testing.

I started doing the framework and after one hour I realized that I should write
unit tests for the framework in the same time when writing framework's code -
writing code for not UI-related framework is a lot easier than with some UI
specific code. And I started. I wrote tests for the code that already exists
and then started writing tests with regular code.

After 8 hours of development I realized too that there is 10 classes and
structs, 2 protocols and 2 enums. I started working on Demo app to present
how the framework works and I found there is some issue, probably in the
framework...

The process of fixing was very nice and quick and I am happy about that.
I found this bug after 8 hours of coding at 01:00am and I usually want to go
sleep at this time, so I didn't want to check every case manually to confirm
that nothing brake because of fix to the recent bug.

### What I did
1. I wrote the test for this not working case in my demo app - this provided me that this situation will be tested in the future. If some change brake the logic, I'll be notified about it quickly.
2. I fixed the bug.
3. Run the tests again - all passed, so I am happy ;)
4. Demo works perfectly.
5. I was calm all the time because other 44 tests checked that entire previous implementation works as it should.

### Conclusion
I know that writing tests may be a long process when testing code is close to
the view controllers and UI and you probably would say: "Ohh, I don't really
want to test it, It will be pain in the butt". You probably should, but what
should be tested for sure is low level code in the app which is responsible for
the app logic. The faster you start writing tests the easier development and
fixes will be ;)
