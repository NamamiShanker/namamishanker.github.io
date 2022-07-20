+++ 
draft = false
date = 2022-07-14T18:38:33+05:30
title = "Week 3 and 4: The new argparse F2PY"
description = "The new argparse F2PY is here"
slug = "argparse-f2py"
authors = ["Namami Shanker"]
tags = ["f2py", "argparse", "NumPy", "SciPy"]
categories = []
externalLink = ""
series = ["GSoC'22"]
+++
# Week 3 and 4: Argparse F2PY

The much awaited [F2PY](https://numpy.org/doc/stable/f2py/) with an [argparse](https://docs.python.org/3/library/argparse.html) frontend is here. It is in review as I write this blog but if you are feeling adventurous and can't wait use it, you can clone [my fork of NumPy and checkout the f2py_front branch](https://github.com/NamamiShanker/numpy/tree/f2py_front). Installation of this experimental NumPy version are detailed in  [these instructions](https://numpy.org/devdocs/user/building.html).(Of course I am joking, no one who gets excited over F2PY builds NumPy)

Lets dive into the changes that have taken place:-

### The `argparse` framework

Python has really good command line parsing frameworks such as [click](https://click.palletsprojects.com/en/8.1.x/), [Rich](https://rich.readthedocs.io/en/stable/introduction.html), [Typer](https://typer.tiangolo.com/), and of course, the built-in `argparse` module. `argparse` isn't usually the first choice for a developer when it comes to adding CLIs. Many developers prefer `click` and `typer` for this purpose as they provide higher degrees of freedom and more polished interfaces for flags and arguments, autocompletion, and better code structure as compared to `argparse`'s rigid requirements and boiler plate such as handling [Namespace](https://docs.python.org/3/library/argparse.html) with an `if-else` ladder and attaching correct functions.

However, when you are renovating a CLI from the early 2000s and need to maintain backward compatibility, there is only so much you can modernise in one go. Additionally, `numpy` does not wish to force users to have third-party dependencies. `argparse` provides us with enough features to implement all the features of F2PY's `sys.argv` based front-end. In fact, BECAUSE `argparse` forces user to handle Namespace, we were able to easily maintain the `numpy.distutils` compilation backwards compatibility. Moreover, the other projects would add heavy dependencies to NumPy for maintaining and using only a very small part of the codebase.

{{< notice info >}} NumPy has **zero** external Python module dependencies, while more than 10,000 out of 390,000 python packages use NumPy {{< /notice >}}

### F2PY's features

So my GSoC mentor, [Mr (soon to be Dr) Rohit Goswami](https://github.com/HaoZeke) started with building the frontend with `argparse` in [October'21](https://github.com/numpy/numpy/compare/main...HaoZeke:numpy:argparse_f2py). I wasn't a part of this project until a little before GSoC this year. The goal was to modernise F2PY's command line parsing interface of F2PY - [f2py2e](https://github.com/numpy/numpy/blob/main/numpy/f2py/f2py2e.py).

Now, let me remind you of the major capabilities of F2PY:-
1. Generating [signature file](https://numpy.org/devdocs/f2py/signature-file.html) for user to tweak the interface of between Python and the library user will build with F2PY.
2. Generate the C wrapper of Fortran code, which can be compiled and linked with Fortran code to produce desired python library.
3. [Building the python library](https://numpy.org/devdocs/f2py/buildtools/index.html). Basically, the automated 2nd step where F2PY generates a C wrapper to a Fortran source (or a pyf) file and uses `numpy.distutils` for compilation and linking. 

All the 3 above steps make sense in a linear flow. **Firstly** you create signature file of your fortran code using
```bash
f2py -m mypythonlib -h signature.pyf source.f 	# signature.pyf generated
```
and make sure the interace is correct. **Secondly** you can generate a C wrapper using the created signature file, 
```bash
f2py -m mypythonlib signature.pyf 	# Generate a mypythoblibmodule.c wrapper
```
but it is almost never required outside of debugging, unless the wrappers are to be explicitly modified and checked into version control. F2PY can reliably create the desired interface of the wrapper using your tweaked `.pyf` file. However, if you want to build your library using another build system, this C wrapper file is important. It needs to be bundled with other fotran sources and F2PY's internal C libaries which as described [here](https://numpy.org/devdocs/f2py/buildtools/index.html).
**Thirdly** You will build your module using
```bash
f2py -c -m mypythonlib signature.pyf source.f
```
This will create your Python library shared object file, which you can import and use.
```python=3.10
>>> import mypythonlib
>>> mypythonlib.sum([1, 2])
3.0
```

### How did`f2py2e` work?

If you read through [f2py2e](https://github.com/numpy/numpy/blob/main/numpy/f2py/f2py2e.py), you will find that it has two main functions - [run_main()](https://github.com/numpy/numpy/blob/cb28cd17d01a6a867517a3b8b3ea4fbb0370be10/numpy/f2py/f2py2e.py#L411) for handling wrapper and signature file generation [(Step 1 and 2)](#f2pys-features) and [run_compile()](https://github.com/numpy/numpy/blob/cb28cd17d01a6a867517a3b8b3ea4fbb0370be10/numpy/f2py/f2py2e.py#L505) for building extension modules using `numpy.distutils`[(step 3)](#f2pys-features). Now, the `f2py2e` command line parsing design was a product of its times. Both the `run_main()` and `run_compile()` functions have seperately implemented command line parsing in two different ways. Both use different default values of important information like the build directory and module name. `run_compile` runs the entire workflow by emulating what`run_main` does internally with `numpy.distutils`. `run_main` and `run_compile` are two divergent code-paths although they are parts of the same 3 step flow we discussed above.

This design led to a lot of problems while we tried to maintain backwards compatibility with `numpy.distutils`. If you read my implementation [f2pyarg here](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/f2py/f2pyarg.py#L600), you can see I am reconstructing command line arguments from Namespace because `distutils` needs it to call `f2py2e.run_main()`.

Probably in a year, `numpy.distutils` will be deprecated, which means F2PY won't need to maintain the convoluted flow for building modules using `numpy.distutils`. F2PY is shifting to [Meson](https://mesonbuild.com/), and by the time distutils is deprecated F2PY will be fully supported by the `meson` build system.

To read more about `f2py2e` you can visit my blog [Week 1: F2PY Frontend](/posts/f2py_frontend).

##### How `f2pyarg` does it?

[f2pyarg](https://github.com/NamamiShanker/numpy/blob/f2py_front/numpy/f2py/f2pyarg.py) is the ongoing reimplementation of F2PY's frontend. I started working on it during GSoC after Rohit had created a basic layout by November last year. My plan was to split the bulky `f2py2e` into two files - `f2pyarg` for dealing with the parser which would connect to a [service.py file](https://github.com/NamamiShanker/numpy/blob/f2py_front/numpy/f2py/service.py) responsible for dealing with F2PY internals. My goal here is to create a hierarchical file structure which can be further improved. This step seperate F2PY's functionalities with F2PY's parser. The user can now connect with user f2py functionalities by passing values instead of feeding command line arguments to a parser.

`f2pyarg.py`'s design is fairly straightforward and easy to understand. It has a main parser with 2 [argument groups](https://docs.python.org/3/library/argparse.html#argument-groups) - ([--hint-signature/-h](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/f2py/f2pyarg.py#L390) group and [-c](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/f2py/f2pyarg.py#L409) group). It has some helper functions to sanitize input arguments, and some command line flag reconstruction functions (ex. [get_f2py_flags_dist](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/f2py/f2pyarg.py#L600), [get_fortran_compiler_flags](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/f2py/f2pyarg.py#L636)) necessary for building with `numpy.distutils`.

The [process_args](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/f2py/f2pyarg.py#L696) is the main argument handling part of the frontend. First we have to parse positional arguments (the file paths etc). I faced a lot of problem here in implementing `skip:` and `only:` flags of original CLI `f2py2e`. Basically, F2PY provides these flag for user to pass some functions as arguments s/he would like to omit or select. But `argparse` framework can't accept `skip:` or `only:` as flags, they need to start with a hyphen, like `--skip:`. So these flags and their arguments were being sent to the positional arugment list along with source files. `f2pyarg` therefore has to segregate [this mixture of files and functions](https://github.com/NamamiShanker/numpy/blob/f2py_front/numpy/f2py/f2pyarg.py#L679-L694).

The files are then [segregated based on the extension](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/f2py/f2pyarg.py#L704). The flow goes by [sanitizing module name and signature file](https://github.com/NamamiShanker/numpy/blob/f2py_front/numpy/f2py/f2pyarg.py#L707-L709), [creating settings for F2PY's internal modules](https://github.com/NamamiShanker/numpy/blob/f2py_front/numpy/f2py/f2pyarg.py#L711-L745). I want to improve this settings mechanism in the future. The current way of [creating these large dictionaries](https://github.com/NamamiShanker/numpy/blob/f2py_front/numpy/f2py/f2pyarg.py#L711-L743) and passing them doesn't please me. If the user has chosen not to build the module right now (`-c` flag omitted), the [generate files](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/f2py/f2pyarg.py#L767) function simply creating signature file or wrapper.

If the user wants to build and passes the `-c` flag, we build the module using `numpy.distutils`. Now, the new `f2pyarg` implements `numpy.distutils` module building mechaninism in a messy way too. It [recreates the command line arguments](https://github.com/NamamiShanker/numpy/blob/f2py_front/numpy/f2py/f2pyarg.py#L600-L657) using the Namespace arguments, `numpy.distutils` [internally calls `f2py2e`](https://github.com/NamamiShanker/numpy/blob/f388c96cec2c5ec0b2c17d54b7928c484eb01148/numpy/distutils/command/build_src.py#L542) so we are still not free from it etc etc. But we couldn't improve it any further because of the time limitation. Furthermore, whatever new fancier implentation we make will be deprecated soon so its not worth it. We just cleaned up the original `run_compile` method as much as we could and stopped there to catch our breathes. My next part of the project, implementing backend build system with Meson will probably make everything much cleaner and maintainable.

If you are a seasoned Python programmer or a veteran CLI designer, I urge you to review and drop constructuve criticism on my implementation - [f2pyarg](https://github.com/NamamiShanker/numpy/blob/f2py_front/numpy/f2py/f2pyarg.py). 

## Moving forwards

I think F2PY's frontend will improve a lot once we deprecate `numpy.distutils` completely. `f2pyarg` still does wrapper generation and compilation seperately due to the `numpy.distutils` requirement. Shifting F2PY completely to `meson` and possibly `cmake` will improve code reusability and frontend code structure a lot. I am looking forward to make a good design in the upcoming months as I work on Meson Integration.

I am a beginner with a lot to learn about software design and programming. I will continue to maintain this code I have written and improve it with others' help. `f2pyarg` has come and its here to stay.