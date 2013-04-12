---
layout: post
title: Runner add-on
---

# Runner add-on

I just released a new update plus an add-on for [Worqshop](http://worqshop.com), an IDE for the iPad. The add-on allows users to execute small Python and Ruby scripts from the iPad. Is it allowed by Apple? Yes, because the execution doesn't happen on the iPad itself but on the cloud.

I envisioned Worqshop to be an alternative development environment that can truly replace computers for software development. Well, it's not there yet, but I'm taking small incremental steps towards it. One of the steps is to provide a runtime environment where users can execute their scripts from the iPad. And this functionality is provided by the new Runner add-on.

### Price

The add-on is a 1-month non-renewing subscription for US $0.99. Some people might say: "Outrageous! It should come with the app for free!" I'm not making any profit from this and the server-side cloud costs money. I implemented the add-on as in-app purchase to prevent unauthorised users and jailbroken devices from accessing the Worqshop cloud. The price of US $0.99 is the lowest price tier allowed on the App Store for in-app purchase.

### Light Table

I've watched Bret Victor's [presentation](http://vimeo.com/36579366) and Chris Granger's video of the Light Table [Playground](http://www.chris-granger.com/2012/06/24/its-playtime/). I really like the concept and, reflecting back, that's how I learnt Python. When I'm stuck with a piece of code, I just open up a new Terminal window, run Python, and test the code there. The short feedback loop of executing and seeing the results helps me to understand the code better and allows me to build larger functions / programs. Another example, in the past when I used Eclipse and developed Eclipse plug-ins, one of the features that I liked was the [Eclipse Scrapbook Page](http://www.eclipsezone.com/eclipse/forums/t61137.html).

At the moment, the Runner add-on behaves like the Eclipse Scrapbook Page, i.e. run the script and display the output. There is no code introspection like the Light Table Playground. Nevertheless, I've experimented with Python introspection and I'm hopeful that I can implement it on the server-side code.

### Implementation

The Runner user scripts are executed on Amazon EC2 instances. The instances themselves are managed by a couple of Google App Engine apps. When a user runs a script, it is sent to a particular EC2 instance, executed, and the output is sent back to the iPad. The script execution is limited to 15 seconds. Using GAE as the intermediary allows me to utilise other IaaS providers, just in case AWS goes down :-)

### Alternatives

Some other alternatives on the App Store that allow you to run Python / Ruby code: [Python for iOS](http://pythonforios.com) and [CodeToGo](http://pinkeh.com/iphone/apps/codetogo/) that utilises [Ideone](http://pinkeh.com/iphone/apps/codetogo/). I was thinking to use Ideone APIs, but not happy with the price. I managed to embed the Python interpreter on Worqshop but I removed it to avoid problems with the App Store team.
