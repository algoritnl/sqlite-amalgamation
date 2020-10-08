[User Guide](User-Guide.md) | [Configuration Options](Configuration-Options.md) | [Step-by-Step Guide](Step-by-Step-Guide.md)
--------------------------- | ------------------------------------------------- | -------------------------------------------

# User Guide

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Building the Library](#building-the-library)
  - [The Library Name](#the-library-name)
- [Installing the Library](#installing-the-library)
- [Packaging the Library](#packaging-the-library)
- [Using the Library from another Project](#using-the-library-from-another-project)
- [Embedding the Library inside another Project](#embedding-the-library-inside-another-project)

## Introduction

According to the creators of SQLite:

> For most applications, the recommended method for building SQLite is to use the amalgamation code file, `sqlite3.c`,
> and its corresponding header file `sqlite3.h`. The sqlite3.c code file should compile and run on any unix, Windows
> system without any changes or special compiler options. Most applications can simply include the sqlite3.c file
> together with the other C code files that make up the application, compile them all together, and have working and
> well configured version of SQLite.
>
> _Most applications work great with SQLite in its default configuration and with no special compile-time configuration.
> Most developers should be able to completely ignore this document and simply build SQLite from the amalgamation
> without any special knowledge and without taking any special actions._

This repository provides a top-level `CMakeLists.txt` file for CMake to generate build system scripts that can build,
install and export the SQLite library and its interactive shell from the amalgamation code file. It can also be used to
embed the amalgamation in another project that uses CMake. A recent version of the amalgamation files is included.

## Prerequisites

A fairly recent version of the CMake build system generator is required. It uses features that were introduced in
version 3.15. Any standard compliant C compiler should be able to compile the amalgamation. In short:

- Standard C compiler + library
- CMake 3.15+
- Build system supported by CMake such as GNU make, Nmake, Ninja or an IDE such as Visual Studio, Eclipse CDT4, etc.

## Building the Library

It is recommended to build the library outside the source tree. Assuming that the source tree is in a subdirectory named
`sqlite-amalgamation`, then executing these commands on the command line will build the library in the subdirectory
`build`:

    $ cmake -E make_directory build
    $ cmake -S sqlite-amalgamation -B build
    $ cmake --build build

The above will build a static library without specifying the build type, which means that there are no compiler flags
that choose an optimization. For a shared library built for release, one would add the following options:
`-DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON` so that the second command becomes (with proper types):

    $ cmake -DCMAKE_BUILD_TYPE:STRING=Release -DBUILD_SHARED_LIBS:BOOL=ON -S sqlite-amalgamation -B build

For a full list of recognized configuration options, please consult [this document](Configuration-Options.md).

### The Library Name

Some options have an impact on the actual *name* of the library; the most obvious one being `BUILD_SHARED_LIBS` which
determines the extension of the library (**.a** vs. **.so** on Unix-likes and **.lib** vs. **.dll** on Windows). Others
have an impact of the *basename* of the library. Libraries built for Debug get a **-dbg** suffix, libraries built
for a single-threaded thread-model get an infix **-st** and libraries built for a multi-threaded thread-model get an
infix **-mt**.

**Static Library Names on Unix-like Systems**

Thread-model          | Non-Debug         | Debug
:-------------------- | :---------------- | :--------------------
`SQLITE_THREADSAFE=0` | `libsqlite3-st.a` | `libsqlite3-st-dbg.a`
`SQLITE_THREADSAFE=1` | `libsqlite3.a`    | `libsqlite3-dbg.a`
`SQLITE_THREADSAFE=2` | `libsqlite3-mt.a` | `libsqlite3-mt-dbg.a`

**Shared Library Names on Unix-like Systems**

Thread-model          | Non-Debug                 | Debug
:-------------------- | :------------------------ | :----------------------------
`SQLITE_THREADSAFE=0` | `libsqlite3-st.so.3.33.0` | `libsqlite3-st-dbg.so.3.33.0`
`SQLITE_THREADSAFE=1` | `libsqlite3.so.3.33.0`    | `libsqlite3-dbg.so.3.33.0`
`SQLITE_THREADSAFE=2` | `libsqlite3-mt.so.3.33.0` | `libsqlite3-mt-dbg.so.3.33.0`

Shared libraries on Unix-like systems have the version appended to their name, accompanied by a symlink without the
version.

## Installing the Library

The location where the library is installed can be controlled with the configuration option `CMAKE_INSTALL_PREFIX`.
The default is `/usr/local` on Unix-likes and `C:\\Program\ Files\\${PROJECT_NAME}` on Windows, according to the CMake
documentation. In orde to install to `${HOME}/.local` you can reconfigure the build tree and then build the pseudo
target `install`:

    $ cmake -DCMAKE_INSTALL_PREFIX:PATH=~/.local -S sqlite-amalgamation -B build
    $ cmake --build build --target install

None of the build scripts use `CMAKE_INSTALL_PREFIX` explicitly. One can choose the install location without having to
reconfigure the build tree, and therefore these two commands can be replaced with the single command:

    $ cmake --install build --prefix ~/.local

This will not only install the library and its headers, but also CMake files that help using the library from other
projects.

A typical list of files installed is:

    bin/sqlite3
    include/sqlite3.h
    include/sqlite3ext.h
    lib/libsqlite3.a
    lib/cmake/SQLite3/SQLite3Config.cmake
    lib/cmake/SQLite3/SQLite3ConfigVersion.cmake
    lib/cmake/SQLite3/SQLite3Targets.cmake
    lib/cmake/SQLite3/SQLite3Targets-release.cmake

## Packaging the Library

In order to distribute the library, it has to be packaged. There are a couple of generators available to create packages
such as zip files and tarballs. They need a config file as input that describes what has to go into the package. The
configuration step creates two config files: CPackConfig.cmake for targets and CPackSourceConfig.cmake for sources.
The `cpack` command uses these to create the packages. For instance if you want to create zip files for the targets:

    $ cpack -G ZIP -B "${PWD}/build" --config build/CPackConfig.cmake

And a compressed tarball for the sources:

    $ cpack -G TGZ -B "${PWD}/build" --config build/CPackSourceConfig.cmake

## Using the Library from another Project

For other projects to find the library, it needs to be installed in or extracted to a location where CMake's 
`find_package` command can find it. On a Unix-like system the config files get installed in
`<prefix>/lib/cmake/SQLite3/` and we have to make sure that this `<prefix>` is included in the search paths used by
CMake. CMake uses the standard system environment variable `PATH` if not explicitly turned off. It takes each entry,
strips of `/bin` and `/sbin`, and adds these to the list of search prefixes. Let's say that we've got the following
path:

    PATH=${HOME}/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

Then CMake will attempt each of the following prefixes:

    ${HOME}/.local
    /usr/local
    /usr
    /
    /snap

If this is not enough, then define a `CMAKE_PREFIX_PATH` environment variable or cache variable. Say we installed the
library in `/opt/sqlite3` but `/opt/sqlite3/bin` is not in the `PATH` environment variable, then we can add

    -DCMAKE_PREFIX_PATH=/opt/sqlite3

to the command line for the cmake command. For a more permanent solution, you can also use the environment variable with
the same name and set it in your `.bashrc` script.

Once we've made sure that the SQLite3Config.cmake file can be found by CMake, we can let other projects use it. In the
other project, we'll add a call to `find_package` to locate and load the SQLite3 library.

    if(NOT TARGET SQLite::SQLite3)
        find_package(SQLite3 REQUIRED CONFIG)
    endif()

This will define target `SQLite::SQLite3` if it didn't already exist, and you can link it to your executable:

    target_link_library(myApp PRIVATE SQLite::SQLite3)

## Embedding the Library inside another Project

This one is the easiest and most flexible alternative by far. Simply add the source tree as a subdirectory of the other
project or add the git repository as a submodule, and add the subdirectory to the top-level CMakeLists.txt of the main
project and add all the necessary configuration options (as cache variables), like this:

    # Configuration options recommended by the SQLite developers.
    set(SQLITE_THREADSAFE               0   CACHE STRING "" FORCE)
    set(SQLITE_DEFAULT_MEMSTATUS        0   CACHE STRING "" FORCE)
    set(SQLITE_DEFAULT_SYNCHRONOUS      2   CACHE STRING "" FORCE)
    set(SQLITE_DEFAULT_WAL_SYNCHRONOUS  1   CACHE STRING "" FORCE)
    set(SQLITE_DQS                      0   CACHE STRING "" FORCE)
    set(SQLITE_ENABLE_COLUMN_METADATA   OFF CACHE BOOL   "" FORCE)
    set(SQLITE_LIKE_DOESNT_MATCH_BLOBS  ON  CACHE BOOL   "" FORCE)
    set(SQLITE_MAX_EXPR_DEPTH           0   CACHE STRING "" FORCE)
    set(SQLITE_OMIT_DECLTYPE            ON  CACHE BOOL   "" FORCE)
    set(SQLITE_OMIT_DEPRECATED          ON  CACHE BOOL   "" FORCE)
    set(SQLITE_USE_ALLOCA               ON  CACHE BOOL   "" FORCE)

    # Features that we want enabled in our application.
    set(SQLITE_ENABLE_FTS4              ON  CACHE BOOL   "" FORCE)
    set(SQLITE_ENABLE_FTS5              ON  CACHE BOOL   "" FORCE)
    set(SQLITE_ENABLE_JSON1             ON  CACHE BOOL   "" FORCE)

    add_subdirectory(extern/sqlite-amalgamation)

    ...

    # Link the SQLite library to our application.
    target_link_libraries(MyApp PRIVATE SQLite::SQLite3)

