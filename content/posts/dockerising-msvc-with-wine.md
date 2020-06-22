+++
title = "Dockerising Visual C++ with wine"
date = 2020-06-21
ref = "dockerising-msvc-with-wine"
tags = ["compilers", "tools", "cpp", "wine"]
categories = ["tools", "docker"]
+++

I'm sure that at least once in your life as C or C++ software engineer/developer you had to target the Windows platform, preferring Visual Studio over MinGW or clang.

Without doubt and after many developers surveys, Visual Studio is one of the most efficient and appreciated IDEs available in 2020, thanks to the effort Microsoft put since the 2017 version and thanks to a friendly licensing (above all, a Community Edition).

Naturally, Visual Studio and the compiler toolset are made specifically for Windows and for Mac, but what if all you need is a _fast_ headless system to compile your code and verify that it works, without launching a Virtual Machine or relying to Windows machine?

Time ago I took the inspiration from other similar solutions and decided to make my own one: that's how my `docker-msvc-cpp`Docker Image was born.

This image includes a collection of C++ tools with a pretty simple task in mind: use it in CI/CD Pipelines. You can compile the code with Visual C++, setup a project with CMake, manage the dependencies with Conan, speedup the compilation times with Ninja and build MSI Installers with Wix.

Since everything goes under the Microsoft License, it cannot be redistributed freely, therefore, if interested, the reader can build her own image and use it.

Due to some internals, there are some caveats and minor issues, such as the adjustment of paths, set a CMake generation that isn't "Visual Studio", but you have the choice between "NMake Files" and "Ninja", since MSBuild isn't shipped with the image (but you can fork the project and tailor it to your needs). The system is configured with only a 64 bit compiler, targeting 64 bit systems, since there are nasty bugs popping out in 32 bit, so I've decided to remove it.

You can find all the details on the [GitHub repository](https://github.com/madduci/docker-msvc-cpp). Happy coding!