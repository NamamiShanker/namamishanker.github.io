+++ 
draft = true
date = 2022-06-28T15:36:42+05:30
title = "Moving F2PY to Meson build system"
description = "We discuss implementing F2PY's compilation options in meson"
slug = ""
authors = ["Namami Shanker"]
tags = ["F2PY", "meson", "build"]
categories = []
externalLink = ""
series = ["GSoC'22"]
+++

# Meson as a backend build system for F2PY
Last week we discussed [F2PY's frontend](/posts/f2py_frontend) and how it is undergoing re-implementation using the `argparse` python library. This week we will look at the backend build system of F2PY, and how meson will be used to build the extension module.

## What is the use of a build system?

Simply explained, build system exist to automate compilation and linking of big programs. A program's source code might consist of several hundred C++ files, some C and a few Fortran files. It might also contain library dependencies which will be provided by some object files. A build system takes a simple configuration file, where user can specify the source files, libraries, and other dependencies. The build system will then compile and link the program, and provide the executable. 

#### But can't we do that using a simple bash script?

Aha, this is a good question. Compiling and building programs from scratch take a lot of time. Imagine you are a software developer in 1997 working in a small division in IBM. Your project contains ten thousand source C files. Building the entire program from scractch takes 2 hours. You saw a bug in one of the source files, you changed some code and started building it using the shell script to test it. Even though you made changes in only a single file, the shell script will delete the previous build (or move it to archives) and create the new build from scratch. It can't indetify the changes in source code and only rebuild the changed files.

Here the build systems come into play, they can create a dependency tree of your source code, and only rebuild the changed files. If your manager allows, you can create a CMake configuration file, include all the source files and dependencies, and explain how your want to build your project. For the first time, CMake will obviously take a lot of time to build the project (around the same 2 hours). But the next time you make changes, and rebuild using CMake, it will smartly detect the changes and only recompile and link what is necessary.

Build systems make life vastly easier for developers. A lot of them provide cross-platform building capabilities, will detect external libraries for you, install the software for you, run integration tests, generate documentation, and much more. Thats why every major developing software uses a populare build system for distribution.

## Why does F2PY need a build system

Most of the libraries written for Python are actually written in C and wrapped by Python C API (for ex - NumPy, Pandas, TensorFlow). This [RealPython article on how to build a Python C Extension Module](https://realpython.com/build-python-c-extension-module/) is a good place to start if you want to build your own Python library in C. But if your want to build a Python library using Fortran, [F2PY](https://numpy.org/doc/stable/f2py/index.html) is a go-to tool.

Python does not have any official supported API for Fortran. But then how does F2PY create Python extension modules from Fortran sources? The way it pull this off is by intelligently understanding the Fortran source code and generates a C wrapper for it. This C wrapper is in turn wrapped by Python C API and thus it can be understood and used by the Python interpreter.

The C wrapper and fortran source files need to be compiled and linked by a compiler family to produce a shared module (.so file). Here is where the build systems come in. We can use various build systems to do this job. [F2PY's documentation](https://numpy.org/doc/stable/f2py/buildtools/index.html#build-systems) explains how to use F2PY with some famous build systems.

## What would we need from a build system to support F2PY?

F2PY provides a great deal of freedom to the user of how they was to produce their build. If you go through its [module building manual](https://numpy.org/doc/stable/f2py/usage.html#building-a-module), you can find a ton of flags and variables you can set in the command line to create a build perfect for your needs. F2PY currently uses [NumPy's distutils](https://numpy.org/doc/stable/reference/distutils.html) build system. F2PY leverages unmatched flexibility provieded by `numpy.distutils` to provide user such choices.

Unfortunately, `distutils` will be deprecated which means we have to find a new build system. [CMake](https://cmake.org/) and [Meson](https://mesonbuild.com/) are great contenders, and we choose Meson because it is simple, has great documentation, and provides us enough features to maintain F2PY's all core building functionalities. This [github issue by Ralf Gommers](https://github.com/scipy/scipy/issues/13615) is an excellent source to get insights of why Meson is the better alternative for many Python libraries.

## Meson as F2PY's backend build system

For the next section, I recommend you go through this small article on [how to use Meson with F2PY](https://numpy.org/doc/stable/f2py/buildtools/meson.html). Like most other build systems, meson uses a config file: [meson.build](https://mesonbuild.com/Tutorial.html) to understand how the user wants the project to be built. You can go through [Building a module doc](https://numpy.org/doc/stable/f2py/usage.html#building-a-module) and see all the functionalities we need to cover with a build system. The table below explains what the NumPy's developer team as agreed upon now and how Meson will implement F2PY's building functionalities.

| Flag | Meson feature |
| -------- | -------- |
|`--fcompiler`| [Set FC Flag](https://mesonbuild.com/Reference-tables.html#compiler-and-linker-selection-variables)
|`--f77exec`, `--f90exec`| [Set FC Flag](https://mesonbuild.com/Reference-tables.html#compiler-and-linker-selection-variables) for now. Will use [custom_targets](https://mesonbuild.com/Custom-build-targets.html) after prototype is finished. `--fcompiler` takes precedence.
|`--compiler`| Set CC Flag |
|`--include-paths`, `-I`| `include_directories` in `py3.extension_module`|
|`--f77flags`, `--f90flags` | Pass [global args](https://mesonbuild.com/Adding-arguments.html#global-arguments) to fortran compiler |
|`--link-<resource>` | Add [dependency](https://mesonbuild.com/Dependencies.html#dependencies) in build file |
| Debug and optimization **defaults**| By default pass `-Ddebug=false` and `-Doptimization=3` to meson [buildtype](https://mesonbuild.com/Builtin-options.html#core-options)|
| `--debug` | Pass `-Ddebug=true` to meson |
| `--opt=`, `--arch=` | Pass [global args](https://mesonbuild.com/Adding-arguments.html#global-arguments) to C and Fortran compilers|
| `--noopt` | `-Doptimization=0` |
| `--noarch` | `-Doptimization=2` |
| `-L<dir> -l<libname>` | pass to linker argument `py3.extenion_module(..., link_args=['-L<dir>', '-l<libname>'`|
| `-D<macro>` | Pass to `c_args` |
| `-Uvar=value` | Pass to `c_args` as `-Dvar=value` |

Support for `--help-fcompiler` will be discussed later seperately. Its Meson implementation is rather difficult and requires more discussions. This covers our core operations provided by the F2PY CLI, and soon we will implement a program for generating meson file as described above.