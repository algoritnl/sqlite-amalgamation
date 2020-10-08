[User Guide](User-Guide.md) | [Configuration Options](Configuration-Options.md) | [Step-by-Step Guide](Step-by-Step-Guide.md)
--------------------------- | ------------------------------------------------- | -------------------------------------------

# Step-by-Step Guide

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup of the Build Environment](#setup-of-the-build-environment)
- [Configuration of the Build Tree](#configuration-of-the-build-tree)
- [Building the Targets](#building-the-targets)
- [Testing the Targets](#testing-the-targets)
- [Additional Information](#additional-information)
- [Configuration Options](#configuration-options)
- [Installing](#installing)
- [Packaging](#packaging)
  - [Oh, one more last thing...](#oh-one-more-last-thing)

## Prerequisites

This repository uses the CMake build system to build, install and export the
SQLite library and its interactive shell. In order to use the CMake build
system, the build host needs to have a couple of packages installed. First and
foremost the CMake tooling itself. Version 3.15 is the minimum requirement in
order to use certain features of the build system.

The SQLite library is written in the C language, so a C compiler is next on the
list, with a standard C library. The building of the library has been tested
with the GNU Compiler Collection (GCC), versions 7.5 and 9.3, and with Clang
10.0 on Ubuntu 18.04 LTS, and with Visual Studio Community 2019 on Windows 10
Pro, version 2004. It also builds successfully on GitHub Actions for
ubuntu-latest (Ubuntu 18.04.5 LTS) and windows-latest (Microsoft Windows Server
2019 10.0.17763 Datacenter).

## Setup of the Build Environment

The build environment, where the build artefacts are stored, is created
preferably outside the source repository. In order to create a directory for
the build, in this example called `build-static`, you can use the platform
independent method:

    $ cmake -E make_directory build-static

On a Unix-like system this is equivalent to `mkdir -p build-static`, i.e. it can
make parent directories as needed.

## Configuration of the Build Tree

Next, we'll invoke a CMake Generator to write the input files for the native
build system, such as Unix Makefiles. Assuming that the Git repository has been
cloned to subdirectory `sqlite-amalgamantion`, the platform independent method
is:

    $ cmake -S sqlite-amalgamation -B build-static

This will inspect the build host and generate input files for the build system
in directory `build-static`. The command will write its progress on standard
output, which looks like this:

    -- The C compiler identification is GNU 9.3.0
    -- Check for working C compiler: /usr/bin/cc
    -- Check for working C compiler: /usr/bin/cc -- works
    <multiple lines omitted for brevity>
    -- Configuring done
    -- Generating done
    -- Build files have been written to: /home/john/work/build-static

## Building the Targets

When the makefiles have been generated, we're ready to perform the actual build.
For Unix makefiles we could do this:

    $ cd build-static && make

But of course, cmake has a nice platform independent alternative that hides
the knowledge of which makefile command (nmake, ccmake, Ninja, Xcode, etc...) is
used under the hood:

    $ cmake --build build-static

And on top of that, you won't end up in the build directory as the current
working directory. (Ok, there's `make -C build-static`. I'll give you that. But
the output isn't as nice.) Note that this builds the implicit `--target all`.
The output is short and sweet and should look like this:

    Scanning dependencies of target SQLite3
    [ 25%] Building C object CMakeFiles/SQLite3.dir/source/sqlite3.c.o
    [ 50%] Linking C static library libsqlite3.a
    [ 50%] Built target SQLite3
    Scanning dependencies of target SQLite3Shell
    [ 75%] Building C object CMakeFiles/SQLite3Shell.dir/source/shell.c.o
    [100%] Linking C executable sqlite3
    [100%] Built target SQLite3Shell

This created a static library `libsqlite3.a` and an interactive shell `sqlite3`.
If the Threads library had not been found in the Configuration step, it would
have created a single-thread library `libsqlite3-st.a` instead.

## Testing the Targets

In order to verify that the build was successful, we run the simple test that is
defined in the CMake build script as the pseudo target `test`:

    $ cmake --build build-static --target test

This test is very basic. The checks that the interactive shell reports the
correct version number; it should match the the definition `SQLITE_VERSION` in
`sqlite3.h`.

    Running tests...
    Test project /home/john/work/build-static
        Start 1: CheckVersion
    1/1 Test #1: CheckVersion .....................   Passed    0.00 sec

    100% tests passed, 0 tests failed out of 1

    Total Test time (real) =   0.00 sec

Of course, before actual deployment more rigorous tests must be performed.

## Additional Information

In the first steps we've built a static library, but sometimes it is desired to
have a shared library (in Windows terminology: dynamic-link library, or DLL) for
which we have a special switch, the command line parameter `BUILD_SHARED_LIBS`
which takes a boolean value: `1`/`0`, `ON`/`OFF`, `YES`/`NO`, `TRUE`/`FALSE`,
any one of these will do. We forgot to specify the build type above, so we got
sort of a Release target with no optimization flags. Supported build types are
`Debug`, `Release`, `RelWithDebInfo` and `MinSizeRel`. Let's build a shared
library for debugging:

    $ cmake -E make_directory build-shared-dbg
    $ cmake -DCMAKE_BUILD_TYPE=Debug -DBUILD_SHARED_LIBS=YES -S sqlite-amalgamation -B build-shared-dbg
    $ cmake --build build-shared-dbg

Note that the name of the build directory is arbitrary. It does not have an
impact on the build outcome. The output of the build step shows that a shared
library `libsqlite3-dbg.so` with debug info is built this time:

    Scanning dependencies of target SQLite3
    [ 25%] Building C object CMakeFiles/SQLite3.dir/source/sqlite3.c.o
    [ 50%] Linking C shared library libsqlite3-dbg.so
    [ 50%] Built target SQLite3
    Scanning dependencies of target SQLite3Shell
    [ 75%] Building C object CMakeFiles/SQLite3Shell.dir/source/shell.c.o
    [100%] Linking C executable sqlite3
    [100%] Built target SQLite3Shell

And finally run the little sanity test:

    $ cmake --build build-shared-dbg --target test
    Running tests...
    Test project /home/john/work/build-shared-dbg
        Start 1: CheckVersion
    1/1 Test #1: CheckVersion .....................   Passed    0.00 sec

    100% tests passed, 0 tests failed out of 1

    Total Test time (real) =   0.00 sec

Pfew! The **rpath** in the shell executable, which helps it locate the shared
libraries, was set correctly.

## Configuration Options

So far we've encountered the following options:

- [`CMAKE_BUILD_TYPE`](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html):
  This is a built-in option to select the build type.  Accepted values include:
  `Debug`, `Release`, `RelWithDebInfo` and `MinSizeRel`. The default value is
  None. (You can't set `None` explicitly, just omit the option.)

- [`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/latest/variable/BUILD_SHARED_LIBS.html):
  If present and **true**, this will cause the library to be built shared. The
  default is **false**.

If the interactive shell is not needed, you can skip its construction:

- `SQLITE_BUILD_SHELL`: This is **true** by default, but can be set to **false** in
  which case the interactive shell `sqlite3` will not be built. This implies
  that the sanity test is not available. The Testing step is a no-op.

A full list of supported options are described [here](Configuration-Options.md).

## Installing

Couldn't be easier:

    $ cmake --install build-shared-dbg --prefix ~/.local

This will install all build targets to directory `.local` in the user's
home directory.

    -- Install configuration: "Debug"
    -- Installing: /home/john/.local/lib/libsqlite3-dbg.so.3.33.0
    -- Installing: /home/john/.local/lib/libsqlite3-dbg.so
    -- Installing: /home/john/.local/include/sqlite3.h
    -- Installing: /home/john/.local/include/sqlite3ext.h
    -- Installing: /home/john/.local/bin/sqlite3
    -- Set runtime path of "/home/john/.local/bin/sqlite3" to "$ORIGIN/../lib"
    -- Installing: /home/john/.local/lib/cmake/SQLite3/SQLite3Targets.cmake
    -- Installing: /home/john/.local/lib/cmake/SQLite3/SQLite3Targets-debug.cmake
    -- Installing: /home/john/.local/lib/cmake/SQLite3/SQLite3Config.cmake
    -- Installing: /home/john/.local/lib/cmake/SQLite3/SQLite3ConfigVersion.cmake

If `--prefix ~/.local` is omitted then it will try to install to the default
prefix, which is `/usr/local` but you'll need **root** access for that.

Beware that the step is irreversible. There is **NO** uninstall or clean. If you
want to remove the files, you'll have to do it manually.

Notice that the actual shared lib has a version number in its name, and the one
reported in the Build step is a symbolic link to it.

Furthermore there are a couple of .cmake files that help find the SQLite3
package for your other projects. These projects can import the SQLite3 package
by including the following statement in their CMake script:

    find_package(SQLite3 3.33.0 REQUIRED CONFIG)

Without the keyword `CONFIG` it may stumble over CMake's own `FindSQLite3.cmake`
and try to use a pre-installed version of the package instead of the one
tailored to your needs.

If the package is found then `SQLite3_FOUND` is set to `TRUE`, `SQLite3_VERSION`
is set to the version of the package that was found and target `SQLite::SQLite3`
is available for linking with `target_link_libraries()`.

    target_link_libraries(MyAwesomeApp PRIVATE SQLite::SQLite3)

Via its target properties, this will also make sure that the include directory
for `sqlite3.h` and `sqlite3ext.h` is added to `target_include_directories()`.

## Packaging

If you want to give the installed targets to somebody else, you can create a
package for you, such as a [tarball](https://en.wikipedia.org/wiki/Tar_%28computing%29)
or a [zipfile](https://en.wikipedia.org/wiki/Zip_%28file_format%29).

You can let cmake help you with this, it offers a pseudo target **package**:

    $ cmake --build build-static --target package

The default configuration uses 3 generators (STGZ, TGZ and TZ) to create
packages:

    [ 50%] Built target SQLite3
    [100%] Built target SQLite3Shell
    Run CPack packaging tool...
    CPack: Create package using STGZ
    CPack: Install projects
    CPack: - Run preinstall target for: SQLite3
    CPack: - Install project: SQLite3 []
    CPack: Create package
    CPack: - package: /home/john/work/build-static/SQLite3-3.33.0-Linux.sh generated.
    CPack: Create package using TGZ
    CPack: Install projects
    CPack: - Run preinstall target for: SQLite3
    CPack: - Install project: SQLite3 []
    CPack: Create package
    CPack: - package: /home/john/work/build-static/SQLite3-3.33.0-Linux.tar.gz generated.
    CPack: Create package using TZ
    CPack: Install projects
    CPack: - Run preinstall target for: SQLite3
    CPack: - Install project: SQLite3 []
    CPack: Create package
    CPack: - package: /home/john/work/build-static/SQLite3-3.33.0-Linux.tar.Z generated.

TGZ is a generator that creates a tarball that is compressed with the GNU Gzip
command. TZ is similar but compressed with an older compress utility.
Its compression is not so good. Then there is the STGZ generator which creates
a shell script that contains an encoded tarball.

In most cases, you don't need these default packages. TGZ on its own may be
useful and of course ZIP. So let's reconfigure the workspace to generate those
instead:

    $ cmake -DCPACK_GENERATOR="TGZ;ZIP" -S sam -B build-static

And generate the gzip-compressed tarball and zipfile:

    $ cmake --build build-static --target package

Output:

    [ 50%] Built target SQLite3
    [100%] Built target SQLite3Shell
    Run CPack packaging tool...
    CPack: Create package using TGZ
    CPack: Install projects
    CPack: - Run preinstall target for: SQLite3
    CPack: - Install project: SQLite3 []
    CPack: Create package
    CPack: - package: /home/john/work/build-static/SQLite3-3.33.0-Linux.tar.gz generated.
    CPack: Create package using ZIP
    CPack: Install projects
    CPack: - Run preinstall target for: SQLite3
    CPack: - Install project: SQLite3 []
    CPack: Create package
    CPack: - package: /home/john/work/build-static/SQLite3-3.33.0-Linux.zip generated.

So, what does this zipfile contain?

    Archive:  build-static/SQLite3-3.33.0-Linux.zip
     Length   Method    Size  Cmpr    Date    Time   CRC-32   Name
    --------  ------  ------- ---- ---------- ----- --------  ----
           0  Stored        0   0% 2020-09-26 02:53 00000000  SQLite3-3.33.0-Linux/include/
      580572  Defl:N   150395  74% 2020-09-26 02:51 ec38316d  SQLite3-3.33.0-Linux/include/sqlite3.h
       35269  Defl:N     6178  83% 2020-09-26 02:51 6b0f09ba  SQLite3-3.33.0-Linux/include/sqlite3ext.h
           0  Stored        0   0% 2020-09-26 02:53 00000000  SQLite3-3.33.0-Linux/bin/
     1353472  Defl:N   496422  63% 2020-09-26 02:52 a9c10b12  SQLite3-3.33.0-Linux/bin/sqlite3
           0  Stored        0   0% 2020-09-26 02:53 00000000  SQLite3-3.33.0-Linux/lib/
     1270122  Defl:N   413274  68% 2020-09-26 02:52 efc0a26f  SQLite3-3.33.0-Linux/lib/libsqlite3.a
           0  Stored        0   0% 2020-09-26 02:53 00000000  SQLite3-3.33.0-Linux/lib/cmake/
           0  Stored        0   0% 2020-09-26 02:53 00000000  SQLite3-3.33.0-Linux/lib/cmake/SQLite3/
        3812  Defl:N     1395  63% 2020-09-26 02:52 f0563573  SQLite3-3.33.0-Linux/lib/cmake/SQLite3/SQLite3Targets.cmake
         804  Defl:N      352  56% 2020-09-26 02:53 f5047b6e  SQLite3-3.33.0-Linux/lib/cmake/SQLite3/SQLite3Targets-noconfig.cmake
         543  Defl:N      284  48% 2020-09-26 02:52 74b046b8  SQLite3-3.33.0-Linux/lib/cmake/SQLite3/SQLite3Config.cmake
        1383  Defl:N      610  56% 2020-09-26 02:52 f8e2be88  SQLite3-3.33.0-Linux/lib/cmake/SQLite3/SQLite3ConfigVersion.cmake
    --------          -------  ---                            -------
     3245977          1068910  67%                            13 files

That's exactly the same directory structure as if you'd have run:

    $ cmake --install build-static --prefix SQLite3-3.33.0-Linux

If you had packaged the shared lib build, then even the symbolic link would have
been zipped and unzipped correctly. After unzipping you can copy or move the
contents to wherever you like, such as `~/.local`.

### Oh, one more last thing...

Did you know you can also package the source tree?

    $ cmake -DCPACK_SOURCE_GENERATOR=ZIP -S sqlite-amalgamantion -B build-static
    $ cmake --build build-static --target package_source

This will put the whole repository (_without_ the **.git** subdirectory) in the
zipfile. Of course, cloning the github repository is far more flexible than
distributing a zip file, but still... At least you have the option if the
recipient cannot use git.
