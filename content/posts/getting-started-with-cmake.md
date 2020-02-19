+++
title = "Getting Started with CMake"
date = 2016-11-10 
ref = "getting-started-cmake"
tags = ["cpp", "conan", "cmake"]
categories = ["software engineering"]
+++

A successful project, in informatics or like in any other industry, starts with planning in advance what are the resources needed to accomplish it.
When you start to develop a new project, you don't just throw code in an editor and hack around it. This actually is the case of _hackathons_, where you have just a few hours to put things all together and show a MVP (minimum viable product). 

<!--more-->

In **well crafted** (production-grade) software development, you usually have a pipeline and some steps to follow to finally deliver a product, and this is true if you're in a team or if you're a lone wolf.
Generally, this pipeline is built upon well established work models, such as Waterfall, XP, Agile.
It begins with the design of components (usually done in UML) and _then_ the initial coding phase begins and starts to grew up with newer features, bug fixes and so on.

[**CMake**](https://www.cmake.org) is probably one of the most useful tools for C/C++ programming, because it's used for organizing small and large projects. You can define modules, functions, macros and partition the project in libraries and executables. It's useful because it deals with third-party libraries and their paths and produce Project Solutions for multiple IDEs and support multiple compilers. It can even produce Windows setup installers (based on NSIS or WIX) and deb, rpm packages for Linux. 
Yet some of you might argue its syntax isn't the best one, but the available commands and directives make it a good partner to team up with.

At the time of writing this article (read: November 2016), there are some IDEs which are shipped already with a full CMake support and they are CLion, Qt Creator and KDevelop. Visual Studio probably will support it with the release 15 (the '2017' edition), as came out as rumor from a recent [cppcast](http://cppcast.com/2016/10/kenny-kerr/) episode. This means, CMake is getting more and more adopted for its flexibility and powerfulness.

### One to rule them all 

Currently, from hundreds (if not thousands) of possible project setups, I settled up with a (__personal__) well-established organization:

```bash
/project/
-- CMakeLists.txt
-- cmake_modules/
-- docs/
-- modules/
-- applications/
```

The root `CMakeLists.txt` contains only the project name, the language specification (C or C++), the main version number and the includes of other CMake files contained in the subfolders. Remember that, just like for any other context, keeping the CMake files as small as possible helps the reusability and the modularity of the project itself.

An example of the root CMakeLists.txt file might the following one:

```cmake 
cmake_minimum_required(VERSION 3.6 FATAL_ERROR)

set(PROJECT_NAME "MyWonderfulProject")
set(PROJECT_VERSION 0.1.0)
project(${PROJECT_NAME} VERSION ${PROJECT_VERSION} LANGUAGES CXX)

add_subdirectory(cmake_modules)
add_subdirectory(docs)
add_subdirectory(modules)
add_subdirectory(applications)
```

In the `cmake_modules`, as the clever reader might understand, there's place for all the CMake-related configuration files, which define the project properties: an example of module can be the detection of platform and compiler being used, which C/C++ standard to enable, which compiler and linker flag to activate or how should the project be delivered, e.g. as a zip file or as an installer package.

The `docs` folder is totally optional and useful only if you support documentation generator systems like **Doxygen**: here can be stored all the basic definitions and properties for the documentation.

The `modules` folder keeps all the reusable functions and classes grouped as library targets, depending on their scope and usage. Each library might be compiled statically or dynamically, or as a header-only library when having a structure like [**shown here**](https://vittorioromeo.info/index/blog/2016_cpp_library_configuration_api.html), depending on the project and the CMake custom properties set. Each subfolder is then split into subfolders:

```bash
/modules/my_module/
-- CMakeLists.txt
-- include/
-- src/
-- tests/
```
The `include/` folder is used to store all the header files (*.h, *.hxx, *.hpp, whatever) that we want to expose outside to the client applications and that we want to make searchable in CMake. In `src/` folder we store the module's actual implementation files which should be accessible only by other files in the same module (for people coming from the Java world, this should remind them be the package concept). 
The tests help us to reduce implementation's defects and to keep their behaviour stable and reproducible. This can be even more effective using **TDD** (_test driven development_), crafting well designed pieces of software, always tested before actually implement them in production.

Finally, in `applications` we write our proper software application, which uses the previously built modules. Like for modules, we split the folder in includes, sources and tests.

### CMake for modules

Let's assume you develop two independent libraries by yourself in an unrelated time frame. Both of them have their own CMake scripts but you want to override them and use the libraries together in a new project. 

CMake has the not-so-known capability of downloading external projects and configure, build and install them inside your current project. The _magic_ happens with the function [**ExternalProject_Add()**](https://cmake.org/cmake/help/v3.6/module/ExternalProject.html), which performs all the aforementioned operations for us. I've already published a [**GitHub repository**](https://github.com/madduci/CppDevLibs) showing how it could be possible to compile libraries such as [Boost](http://boost.org/), [OpenSSL](https://openssl.org/) and [Google Test](https://github.com/google/googletest/) using this approach, in a cross-platform twist (currently, it's tested on Windows and Linux, thanks to [Travis CI](https://travis-ci.org/) and [AppVeyor](https://www.appveyor.com/) platforms).

This approach makes each C/C++ module really reusable like a CMake-plug, speeding up the whole workflow, as long as we use a centralized set of CMake definitions, making the code DRY and tested and defining a version number / release plan for each of those modules. This modularization is what projects like [**conan**](https://www.conan.io/) or the recent [**vcpkg**](https://github.com/Microsoft/vcpkg) are trying to address in the C++ community.

### CMake limitations and workarounds

CMake has some limitations, but its intrinsic composability and flexibility help to overcome to them with nice workarounds 

#### Build multi-architecture CMake projects

CMake by default can build one project at a time and for a specific architecture at a time. You cannot build code for the x86 architecture for both 32 and 64 bit at the same time, or you cannot build x86_64 code and arm64 one together. But there's a workaround for that: the trick comes directly from [**execute_process()**](https://cmake.org/cmake/help/v3.6/command/execute_process.html?highlight=execute_process) which is able to trigger other cmd/shell commands from within our CMake project, allowing us to create nested projects. This is useful when you want to create, for example, a single installer out of multiple builds. The clever reader might argue to create a new CMake project to handle just this case: sure, it's possible and it's totally fine to do that, but as drawback you have to duplicate the set of rules already defined for your project (e.g., version number, vendor, path to resources, install directory and so on).

Supposing you're working with Visual Studio 2015 and you want to build and package together 32 and 64 bit deliverables (a scenario that might be typical for a driver package):

```cmake
list(APPEND BUILD_ARCHITECTURES
    x86
    amd64
)
set(COMPILER_GENERATOR_NAME "Visual Studio 14 2015")
set(PROJECT_SOURCE "${CMAKE_SOURCE_DIR}/src")

# Configure project 
foreach(ARCHITECTURE ${BUILD_ARCHITECTURES})
    message("Preparing project for ${ARCHITECTURE}")
    configure_file(${PROJECT_SOURCE}/definitions.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/${ARCHITECTURE}/definitions.cmake @ONLY)

    set(BUILD_DIRECTORY ${CMAKE_BINARY_DIR}/${ARCHITECTURE})
    file(MAKE_DIRECTORY ${BUILD_DIRECTORY})

    if("${ARCHITECTURE}" MATCHES "amd64")
        set(COMPILER_GENERATOR_NAME "${COMPILER_GENERATOR_NAME} Win64")
    endif()

    # Prepare CMake project
    execute_process(
        COMMAND ${CMAKE_COMMAND} -G ${COMPILER_GENERATOR_NAME} -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} ${PROJECT_SOURCE}
        WORKING_DIRECTORY ${BUILD_DIRECTORY}
    )

    # Add install Target for the corresponding ARCHITECTURE
    install(CODE "execute_process(
            COMMAND \"${CMAKE_COMMAND}\" \"--build\" \".\" \"--target\" \"install\"
            WORKING_DIRECTORY ${BUILD_DIRECTORY})")
endforeach()
```

it will end up with a `build/` folder partitioned in `x86/` and `amd64/`.

#### Custom CPack templates

Another limitation involves the creation of installers using templates, like **NSIS**. The default template, even if it allows us to set some variables, it's limited to a few things. 
You can override the default NSIS template by supplying your own one with the variables and functions of your choice, which _still_ has to be named  `NSIS.template.in`, like the original one:

```cmake
set(CPACK_MODULE_PATH path/to/NSIS.template.in/folder/)
```

Thanks for reaching the end of this article! Next time we'll see in details how to establish a modularization and let CMake integrate them in one single project, generating the solution for us and testing it with different compilers. See you next time! 