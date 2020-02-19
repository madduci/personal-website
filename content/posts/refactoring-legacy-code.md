+++ 
draft = true
date = 2020-02-15
title = "Refactoring C++ legacy projects - the onion method"
description = "Some tips on refactoring legacy projects"
slug = "" 
ref = "refactoring-cpp-legacy-projects-onion"
tags = ["cpp", "conan", "cmake"]
categories = ["software engineering"]
externalLink = ""
series = []
+++


At least once in a developer's lifetime the day he/she has to deal with huge, dense and complicated projects, which has grown up over years wildly and at the end the good old guy who was responsible of the build environment has left, or worse, the dependencies needed aren't updated anymore because of their complexity as well and none wants to deal with them. 

<!--more-->

### What is a legacy project?

A legacy project has at least one of the following characteristics:

* it targets each platform with platform's specific configurations (eg. VS Solutions, Makefiles, etc.)
* it includes dependencies (as compiled libraries as well as original source code) in the source folder
* it containes a lot of hard coded paths to the dependencies and additional resources
* it lacks of tests
* it requires that tests have to executed manually
* it depends heavily on specific third party libraries in all of its build targets
* cannot be built automatically end-to-end

Probably the last two points might be less relevant and it depends strictly on which kind of third library is being used, but in general, a good separation between business logic and client logic should be always pursued.
If your project has at least one of the above properties, it's time to think about a refactoring process **as soon as you can**.

### Peeling the onion: starting from the surface

A refactoring process should be always approached carefully and gently, because things are going to break up and you have always to be sure of cleaning up everything and make it _a better place_ than you've found.

Just like peeling an onion, you start from the external surface: before touching it, the project works (hopefully!), the artifacts are shipped to clients after the QA and release process, the building tools are set and the required external libraries are in place.

#### Focus on artifacts

If the artifacts are stored and shipped just as a compressed archive or as a list of files which has to be copied somewhere in an operating system folder, then think about introducing a proper packaging step. If you're shipping something for Linux systems, spend time with deb/rpm/aur package systems: first, achieve the goal, then write down a script to automate the process.
If you're shipping for Windows, spend time with installers: you might pick up one between [**NSIS**](http://nsis.sourceforge.net/Main%5FPage), [**WiX**](http://wixtoolset.org/) or [**InnoSetup**](http://www.jrsoftware.org/isinfo.php): clients will have less pain to set your software up, if you have partly automated it during the installation process. Again, start doing it manually and then write a script to make the installer generator automatically. 

Until now, the project has still its own integrity and we haven't touched anything yet. If the packaging works as expected, you might integrate it in your build process as its final step.

#### Focus on dependencies

Working with third party libraries is simple, said no one ever. Unless you use header-only libraries, which are easier to maintain but increase the compile-time in big projects, dealing with ABI changes, building changes (OpenSSL anyone?), compiler flags and runtime used isn't really funny at all, especially when you have to build them up by yourself. 
If the dependencies are just downloaded from internet, extracted and compile on your machine by manually giving build instructions, again, you should focus on what we want to achieve: clean manangement and automatization whenever is possible.
Can you optimize the above steps with a script? Then write it down, for each required dependency. At the end, write a _master_ script which calls all the scripts defined for each third-party library.

## TODO:

- tests
- cmake
- conan
- jenkins
- separate bl from client logic
