---
layout: post
title: Miscellaneous thoughts on iOS Development
---

## Modules and Dependencies

It's a good idea to group common and reusable code into a library. It minimises code duplication, saves time, helps in debugging, and simplifies code maintenance. The following [tutorial](http://www.blog.montgomerie.net/easy-xcode-static-library-subprojects-and-submodules) shows how to create a library and structure your Xcode project into subprojects. In addition, we also need a way to manage the dependencies. We could use Git submodules as described in the aforementioned tutorial or we could use [CocoaPods](http://cocoapods.org). Both approaches are fine as long as you set the version numbers explicitly for the dependencies. For example, do not set your dependency to the *master* branch of a project. Instead, set it to a well known version, e.g. 2.5.2. You do not want your app to break because of a late night code change by the project maintainer.

## Code Style and Formatting

It's important to have a consistent code style guide. Preferably, it's company wide in scope. It allows new people to move into existing projects easily. It allows people to switch between projects easily and it simplifies collaboration of two or more people in a big project. However, this is a subjective topic and people tend to have strong opinions on this. There are so many Objective-C style guides, for example, there are guides from [Google](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml), [Apple](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html), [Adium](https://trac.adium.im/wiki/CodingStyle), [Zarra Studios](http://www.cimgf.com/zds-code-style-guide/), and so forth. Related to the code style is code formatting. Again, people have strong opinions on code formatting, for example, tabs or spaces, 4 or 8 spaces, bracket on the same line or next line, and so forth. Some people use code formatting tool like [Uncrustify](http://uncrustify.sourceforge.net) for Xcode projects, for example, [Uncrustify Automator Services](https://github.com/tonyarnold/Xcode-4-Uncrustify-Automator-Services). Some people use Git pre commit hooks to format the code before it's being committed to a repository. To iterate, it's important to have a consistent code style guide. Having said that, if it becomes a blocker in the development process, it's also useful to see this in a larger context. After all, apps on the App Store are not judged by whether or not they use  tabs/spaces.

## Prototyping

For a quick prototyping, just use [Storyboard](https://developer.apple.com/library/ios/#releasenotes/Miscellaneous/RN-AdoptingStoryboards/) and Interface Builder. The tools are useful and they get the job done in a short amount of time. They allow you to quickly prototype the UI, get something functional running, and test the app with users. In addition, Storyboard allows you to design your table views and cells statically (no code is required). Unfortunately, it's not available in Interface Builder. Do not write code for the UI for an initial prototype. The code might not survive the first user acceptance testing. In addition, we could also use one of the many iOS prototyping apps: [Briefs](http://giveabrief.com), [POP](http://popapp.in), or [Prototypr](http://prototypr.com). Or one of the web apps: [Fluid](https://www.fluidui.com) or [Flinto](https://www.flinto.com).

## Testing

With iOS 7 and apps that can auto update, application testing is becoming more and more important. New projects created with the new Xcode 5 come with unit tests by default. In addition, Apple has provided new tools and frameworks that put further emphasis in testing and continuous integration: the new XCTest framework, Xcode 5, Xcode bots, and Xcode Server in Mac OS X. Thus, new apps need to be developed with testing in mind.

## Version Control

There is only one answer here: use a version control, period. Feel free to use Git or Subversion (the only supported version control systems by Xcode), but most of the time, Git is the right choice. It allows you to collaborate with other people, track changes to your code, and review the changes. Xcode provides you with the code comparison editor and history view. Also, please write meaningful and helpful commit messages as explained by this [blog post](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).

## Documentation

Transferring knowledge by talking and hands-on sessions does not scale and it does not work in the long term. It's really important to document the code for the sake of your future self and the next project maintainer. If the project is big and the design needs to be explained, write a separate text document explaining the rationales and the design decisions. Write comments to explain an intricate code segment and when you change the code, edit the comments accordingly. Write code comments judiciously. Remember that comments can become stale and lines of comments in the code are lines that need to be maintained.
