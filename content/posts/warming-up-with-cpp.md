+++
title = "Warming up with C++"
date = 2016-10-24 
ref = "warming-up-with-cpp"
tags = ["cpp", "conan", "cmake"]
categories = ["software engineering"]
+++

In my last [**post**]({{< ref "/posts/know-your-tools.md" >}}) I've put in evidence the necessity of learning to _learn_ your development environment. Think about it as a full ecosystem and that each element (compiler/IDE/build scripts) live all together and they contribute to the production of a final deliverable.

<!--more-->

I will assume in this blog series that you use Windows or Linux (sorry, I've no Apple computer to test with, but it _should_ work) which will be fine for the majority of posts, except some particular edge cases we'll in future. 

### Virtualize if you can

My suggestion is to work with virtual machines as much as possible, as it makes you flexible to take snapshots and revert back in case of failure or misconfiguration. You can use [**Virtualbox**](https://www.virtualbox.org/) since it's free or go premium products if you have a license, such as [**VMware**](https://my.vmware.com/web/vmware/downloads) or [**HyperV**](https://www.microsoft.com/en-us/cloud-platform/virtualization) on Windows Server. Another possibility could be [**Docker**](https://www.docker.com/) and for the special purposes of this blog, I've prepared a couple of images sometime ago and they're available on [**Docker Hub**](https://hub.docker.com/r/madduci/docker-ubuntu-cpp/): they already are packed with some of the below listed software packages.

### The Ingredients

The fundamental software you should install are without doubt:

- [**CMake**](https://cmake.org/), for managing your project(s) in a flexible way, enabling, depending on the project itself, the portability of a project from a platform (e.g., Windows) to another platform (e.g., Linux, MacOS), thanks to the built-in smart compiler detector.
- A compiler, which depends on the platform of your choice. They are examinated later in this post.
- An IDE or an editor of your choice. There are some nice cross-platform IDEs such as [**Visual Studio Code**](https://code.visualstudio.com/Download), [**Qt Creator**](https://www.qt.io/ide/), [**CLion**](https://www.jetbrains.com/clion/) or [**KDevelop**](https://www.kdevelop.org/), just to mention the most famous ones.
- A static code analyzer, which helps to detect and prevent some mistakes introduced by humans when coding. They're not to be trusted 100%, but they offer some hints on some common mistakes. You can use [**cppcheck**](http://cppcheck.sourceforge.net/), [**clang static analyzer**](http://clang-analyzer.llvm.org/) if you use the clang compiler or [**PVS-Studio**](http://www.viva64.com/en/pvs-studio).
- Memory analyzers, debuggers, to analyze the runtime behaviour of the developed software. Some IDEs (such as Visual Studio) already have them bundled in their tools, some other require external setups, such as **valgrind** or **gdb** under Linux (both available in your distro's repositories) or [**Dr. Memory**](http://www.drmemory.org/) which is cross-platform.

### Compilers

On Windows you might go for [**Visual C++**](https://www.visualstudio.com), which comes with an IDE, or, if you want to be platform-agnostic, you would like to use [**clang**](http://clang.llvm.org/). 

The compiler's choice is the most important one, as long as all the future required libraries/dependences and the all the final libraries/executables we'll produce will not be compatible with other compilers (e.g. something compiled with Visual C++ is not compatible with something compiled with gcc).

This choice is influenced by the hosting platform. As said above, if you plan to use Visual C++ compiler, you are tighting yourself with Windows. To make it less problematic, again, try to use virtualized environments.

In this blog series I will try to write as much not-platform specific code as possible. We're lucky as long as C++ offers preprocessor guards:

```cpp
#ifdef WIN32
//here goes Windows-compatible code
#else
//here goes non Windows-compatible code
#endif
```

and CMake has a detector as well:

```cmake
if(WIN32 AND MSVC)
#CMake directives for Windows + Visual C++
else()
#Other directives for other combinations
endif()
```