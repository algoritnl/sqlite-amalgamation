# SQLite

For detailed information on SQLite, please visit the official SQLite
[homepage](https://sqlite.org/index.html).

## Derived Work

This is an unofficial repository that adds custom CMake build system generator
scripts to the official SQLite amalgamated source code releases. The source code
in this repository is extracted from the official zip-file downloaded from the
SQLite website, without modifications. Please consult the
[User Guide](docs/User-Guide.md) on how to use this repository.

**The rights to this derived work are covered by the
[Unlicense](UNLICENSE.md).**

## Table of Contents

- [What is SQLite?](#what-is-sqlite)
- [The Amalgamation](#the-amalgamation)
- [Obtaining the Source](#obtaining-the-source)

## What Is SQLite?

Small. Fast. Reliable. Choose any three.

SQLite is a C-language library that implements a small, fast, self-contained,
high-reliability, full-featured, SQL database engine. SQLite is the most used
database engine in the world. SQLite is built into all mobile phones and most
computers and comes bundled inside countless other applications that people use
every day.

The SQLite file format is stable, cross-platform, and backwards compatible and
the developers pledge to keep it that way through at least the year 2050.
SQLite database files are commonly used as containers to transfer rich content
between systems and as a long-term archival format for data. There are over 1
trillion (1e12) SQLite databases in active use.

**SQLite source code is in the [public-domain](source/LICENSE.md) and is free to
everyone to use for any purpose.**

## The Amalgamation

SQLite is built from over one hundred files of C code and script spread across
multiple directories. The implementation of SQLite is pure ANSI-C, but many of
the C-language source code files are either generated or transformed by
auxiliary C programs and AWK, SED, and TCL scripts prior to being incorporated
into the finished SQLite library. Building the necessary C programs and
transforming and/or creating the C-language source code for SQLite is a complex
process.

To simplify matters, SQLite is also available as a pre-packaged amalgamation
source code file: **sqlite3.c**. The amalgamation is a single file of ANSI-C
code that implements the entire SQLite library. The amalgamation is much easier
to deal with. Everything is contained within a single code file, so it is easy
to drop into the source tree of a larger C or C++ program. All the code
generation and transformation steps have already been carried out so there are
no auxiliary C programs to configure and compile and no scripts to run. And,
because the entire library is contained in a single translation unit, compilers
are able to do more advanced optimizations resulting in a 5% to 10% performance
improvement. For these reasons, the amalgamation source file ("**sqlite3.c**")
is recommended for all applications.

&emsp;_The use of the amalgamation is recommended for all applications._

Building SQLite directly from individual source code files is certainly
possible, but it is not recommended. For some specialized applications, it might
be necessary to modify the build process in ways that cannot be done using just
the prebuilt amalgamation source file downloaded from the website. For those
situations, it is recommended that a customized amalgamation be built and used.
In other words, even if a project requires building SQLite beginning with
individual source files, it is still recommended that an amalgamation source
file be used as an intermediate step.

## Obtaining the Source

The leading SQLite source repository is available from here:<br/>
https://www.sqlite.org/cgi/src (Dallas)

Mirrors:<br>
https://www2.sqlite.org/cgi/src (Newark)<br>
https://www3.sqlite.org/cgi/src (San Francisco)

The official GitHub mirror:<br/>
https://github.com/sqlite/sqlite
