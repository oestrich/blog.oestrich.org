---
layout: post
categories:
- autotools
- linux
- c
title: Setting Up Autotools
---

For a while I have wanted to start learning C, but I have always been daunted by compiling a project that is larger than 1 source file. I started reading [21st Century C][century-c] and it finally got me past the hurdle of autotools for compiling C projects. These are the scripts I ended up with to start a simple project.

#### Folder structure for an empty C project with autotools

```
.
├── AUTHORS
├── autogen.sh
├── ChangeLog
├── configure.ac
├── COPYING
├── INSTALL
├── Makefile.am
├── NEWS
├── README
└── src
    ├── main.c
    └── Makefile.am
```

`AUTHORS`, `ChangeLog`, `COPYING`, `INSTALL`, `NEWS`, and `README` are required by autotools before it will generate the `configure` script. Right now I have them as empty files to move along.

##### autogen.sh

``` bash
#!/bin/sh
# Run this to generate all the initial makefiles, etc.
set -e

autoreconf -i -f
```

I run this script after checking out the project on a new machine. Autotools will spit out all of the required files to let it compile. Your project folder will have a lot of new files that can be ignored.

##### configure.ac

``` bash
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([prog], [1], [eric@oestrich.org])
AM_INIT_AUTOMAKE
AC_CONFIG_SRCDIR([src/main.c])
AC_CONFIG_HEADERS([src/config.h])

# Checks for programs.
AC_PROG_CC
AC_PROG_CC_C99

# Checks for libraries.

# Checks for header files.

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.

AC_CONFIG_FILES([
                 Makefile
                 src/Makefile
                 ])
AC_OUTPUT
```

This file generates the `configure` script. It expands from macros into the bash script. The best way I found to expand this file is looking at other projects and see what they use. For instance, checking to see if `glib-2.0` is available and can be compiled against:

```bash
AC_CHECK_LIB([glib-2.0], [g_free])
```

##### Makefile.am

``` bash
SUBDIRS=src
```

This sets up automake to look in the `src` subfolder. Most of the work will happen in there.

##### src/Makefile.am

``` bash
bin_PROGRAMS = prog
prog_CFLAGS = # `pkg-config --cflags glib-2.0`
prog_LDFLAGS = # `pkg-config --libs glib-2.0`
prog_SOURCES = main.c # list out all source files
```

`bin_PROGRAMS` tells automake what binary files will be generated. `prog_CFLAGS` and `prog_LDFLAGS` sets up the `CFLAGS` and `LDFLAGS` appropriately. This is where you will add `-I` and `-l` flags. `prog_SOURCES` are what sources gets compiled into `prog`. The last three lines are named after the binary file that is generated.

## All together

Once these files are in place you will be able to clone the repo, and within a few simple commands be compiling your project. Below is the full set of commands it takes.

``` bash
$ git clone $GIT_REPO git_repo
$ cd git_repo
$ ./autogen.sh
$ ./configure
$ make
$ src/prog
```

With this set up I have been able to get past the big hurdle of compiling multiple C source files, and start learning C itself. I hope this helps others get over the hurdle as well.

[century-c]: http://shop.oreilly.com/product/0636920033677.do
