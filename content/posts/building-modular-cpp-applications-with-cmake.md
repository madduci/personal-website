+++
title = "Building modular C++ applications with CMake"
date = 2016-12-11 
tags = ["cpp", "conan", "cmake"]
categories = ["software engineering"]
ref = "writing-modular-cpp-apps-cmake"
+++

[**Last time**]({{< ref "/posts/getting-started-with-cmake.md" >}} ) we've seen how to get started with CMake and discussed about its advantages, especially when we want to compile our code with different compilers, but what if we want to build and deliver our modules?

<!--more-->

### TL;DR

* Demo project at [GitHub](https://github.com/a-coding-journey/foolibrary-cmake)
* Use [`ExternalProject_Add`](https://cmake.org/cmake/help/v3.7/module/ExternalProject.html) to import dependencies in CMake
* Use [**generate_export_header(TARGET)**](https://cmake.org/cmake/help/v3.7/module/GenerateExportHeader.html) to export libraries definitions
* Learn [**conan**](https://conan.io) as alternative to CMake-only management

### The project

I've prepared an example (hosted on [GitHub](https://github.com/a-coding-journey/foolibrary-cmake)), so if you're in hurry, you can directly skip the whole article and get the code.

First of all, the intent should be clear: CMake must tell the compiler to spit out a library and an interface definition, enabling us to include these files in another project later and use them.

### Project structure and root CMakeLists file

The `foolibrary-cmake` project is structured in an easy way:

```
foolibrary-cmake/
-- CMakeLists.txt
-- cmake_modules/
   - Project.cmake
   - cpack.cmake
   - test.cmake
-- include/
   - my_class.h
-- src/
   - my_class.cpp
-- test/
   - my_class_test.cpp
```

The root `CMakeLists.txt` file is our entrypoint: here the project name, project version and final target are defined. It includes additional *.cmake files stored in the `cmake_modules/` folder, to set/get informations about the configuration of the building process.

The my_class files are written just for show how does CMake for modules works:

_include/my_class.h_

```cpp
#ifndef _MY_CLASS_H_
#define _MY_CLASS_H_

#include "fooproject_export.h"
//A Coding Journey
namespace acj
{
  class FOOPROJECT_EXPORT MyClass
  {
    public:
      MyClass() = default;
        
      void greet();

      int answer() const;
  };
}
#endif
```

_src/my_class.cpp_

```cpp 
#include "my_class.h"
#include <iostream>

using namespace acj;

void MyClass::greet()
{
  std::cout << "Hello reader! This is MyClass!\n";
}

int MyClass::answer() const
{
  return 42;
}
```

You've noticed that **FOOPROJECT_EXPORT** symbol, right? Actually, not all the platforms are equal and some of them need to export explicitly symbols to let us be able to use them in other contexts: this is the case of Windows, when we compile shared libraries and produce DLL files. We can automatize the export of those symbols by generating the export header (the **fooproject_export.h** file) during CMake's configuration steps: the only "price to pay" is the include of that header file and the specification of the export definition itself.

The root CMakeLists.txt file defines the source file and the installation steps to produce a clean installation:

```cmake
project("fooproject" CXX)
set(PROJECT_VERSION 1.0.0)
# Including Library settings
include(cmake_modules/Project.cmake)
# Include header files
include_directories(include)
# Define target
add_library(${PROJECT_NAME} ${BUILD_TYPE} src/my_class.cpp)
generate_export_header(${PROJECT_NAME})
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 14)
set_property(TARGET ${PROJECT_NAME} PROPERTY VERSION ${PROJECT_VERSION})
set_property(TARGET ${PROJECT_NAME} PROPERTY SOVERSION ${PROJECT_VERSION})
```

As you might expect, the generation of exporting symbols happens through the CMake function [**generate_export_header(TARGET)**](https://cmake.org/cmake/help/v3.7/module/GenerateExportHeader.html), which grabs the target's name (in this case, ${PROJECT_NAME}, which is "fooproject", as specified in the first line) and creates the export file **fooproject_export.h**, depending on the compiler being used.

The definition of the real target is done with the function **add_library()**, setting the target name, in this case "fooproject", and the source files to be used to produce the target. Additional properties, such as library version and standard to be used are set once the target is defined.

Additionally to the main target, we want to define test targets, which help us in discovering program defects and errors:

```cmake
# Define test
if(BUILD_TESTS)
  add_executable(${PROJECT_NAME}_test test/my_class_test.cpp)
  add_test(class_test ${PROJECT_NAME}_test)
  add_dependencies(${PROJECT_NAME}_test Doctest)
  target_link_libraries(${PROJECT_NAME}_test ${PROJECT_NAME})
endif(BUILD_TESTS)
```

In this case we produce target named "fooproject_test" as executable and set as dependencies the Doctest test

### External dependency

The `cmake_modules/` folder, which shouldn't be meant as the folder containing CMake's find modules folder, stores all the properties needed to be set in order to build the project. In particular, the `Project.cmake` file defines the options that user can set via command line or using the CMake GUI, such as the possibility of building a static or a shared library or if tests should be compiled or skipped. This file includes then other files, such as `cpack.cmake` and `test.cmake`, which describe how the project should be delivered as package (e.g. defining version number, compression format) and which test suite library should be used (in this case, [**Doctest**](https://github.com/onqtam/doctest)).

In this case, we use the function [`ExternalProject_Add`](https://cmake.org/cmake/help/v3.7/module/ExternalProject.html) to trigger the download of Doctest from its repository and define the operations to be performed (download, check the hash and unpack - the extraction of archive is performed internally):

```cmake
# Enable ctest
enable_testing()

# Use ExternalProject_Add facility
include(ExternalProject)

# Define settings
set(DOCTEST_VERSION "1.1.3")
set(DOCTEST_ROOT ${CMAKE_BINARY_DIR}/doctest)

# Download doctest
ExternalProject_Add(
    Doctest
    PREFIX            ${DOCTEST_ROOT}
    TMP_DIR           ${DOCTEST_ROOT}/temp
    STAMP_DIR         ${DOCTEST_ROOT}/stamp
    #--Download step--------------
    DOWNLOAD_DIR      ${DOCTEST_ROOT}/download
    URL               "https://github.com/onqtam/doctest/archive/${DOCTEST_VERSION}.zip"
    URL_HASH          SHA1=cdcb33d3d311f1eeca81e03bfeb50cbbcfe9d6c3
    #--Configure step-------------
    SOURCE_DIR        ${DOCTEST_ROOT}/source
    CONFIGURE_COMMAND ""
    #--Build step-------------
    BUILD_COMMAND     ""
    BUILD_IN_SOURCE   1
    #--Install step---------------
    INSTALL_COMMAND   ""
    INSTALL_DIR       ""
)

include_directories(${DOCTEST_ROOT}/source)
```

Setting **BUILD_COMMAND** and **INSTALL_COMMAND** to an empty string inhibits the execution of the internal build command (e.g. in Linux is make) and the install command (e.g., make install), as long as Doctest doesn't need any build or installation process (it's an head-only library we don't want to deliver later).

As a result of this, the project **Doctest** is downloaded and used in our project. This mechanism allow us to treat external source code as dependency and on specific requirements, it can be compiled and used as soon as possible in our defined targets.

### Alternative: conan

The presented approach works fine and it's well enclosed in the CMake environment itself, dealing everything with one single language and defining clear rules. The main drawback is certainly the complexity that this approach introduces, especially when dealing with extended library configurations (e.g. Boost, OpenSSL, Apache Thrift), but it clearly depends on the configuration system being used by such projects. 

An alternative to use **ExternalProject_Add()**, which decreases the lines of codes of our CMake files, but introduces Python as scripting language, is [**conan**](https://conan.io), a timid approach to tackle the dependency madness which affects the C/C++ ecosystem since ever. The great potential of conan relies on the availability of precompiled libraries, which speeds up the development, skipping the compilation time, needed in case of **ExternalProject_Add()**. If a library is not compiled for your platform, then you can build it using the conan client and that's it.

The integration with CMake works perfectly, as long as you write just few magic lines:

```cmake
project(MyAmazingProject)
cmake_minimum_required(VERSION 3.0)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake) # Required by conan
conan_basic_setup() # Required by conan

add_executable(my_exec my_file.cpp)
target_link_libraries(my_exec ${CONAN_LIBS}) # Required by conan
```

The documentation covers the basic use cases to get you introduced to the commands and on how our project can be configured using a conan configuration file, but it lacks of some advanced usages, such as the redistribution of libraries on systems where conan is not installed, but it can be addressed using CMake.

Another advantage of conan is the possibility of building our projects for multiple target platforms by using the [conan tools](http://docs.conan.io/en/latest/packaging/package_tools.html), skipping the effort of writing scripts invoking CMake with different compilers.