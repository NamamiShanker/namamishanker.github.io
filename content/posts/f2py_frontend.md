---
title: "F2PY Frontend"
date: 2022-06-15T21:34:26+05:30
draft: true
---

# F2PY frontend: The Old and the New

We will look at the current F2PY frontend, and understand the ongoing re-implementation of it using the `argparse` python library.

## Introduction
F2PY is a command line tool that provides easy connection between Fortran languages and Python. It facilitates creating Python C/API extension modules that can be called and imported in Python. It gives you the [speed of Fortran and the flexibility of Python](https://namamishanker.github.io/posts/scipy_fortran_f2py/).

## The frontend of a command line tool

A command line works by parsing the input on the command line, and then calling correct functions based on the input. The input on the command line is generally a list of positional arguments (like files, folders, keys or data) and flags (like `--help` or `--version`). For example, if you type
```
git --version
```
then the command line tool will call the `git` program, and pass the `--version` flag to it as an argument. Internally, an array `argv` containing the argument is created and passed to `git` program. In this case `argv` will look like this: `['--version']`.
```
git add app.py
```
then the command line tool will call the `git` program, and pass the `add` and `app.py` to it as an argument. Internally, `git` will recognize `add` as an option, and `app.py` as a positional argument to `add` option. The `argv` array passed to `git` program will look like this: `['add', 'app.py']`.

## F2PY's frontend

Suppose you have a fortran source file `fib.f`:
```fortran
C FILE: FIB1.F
      SUBROUTINE FIB(A,N)
C
C     CALCULATE FIRST N FIBONACCI NUMBERS
C
      INTEGER N
      REAL*8 A(N)
      DO I=1,N
         IF (I.EQ.1) THEN
            A(I) = 0.0D0
         ELSEIF (I.EQ.2) THEN
            A(I) = 1.0D0
         ELSE 
            A(I) = A(I-1) + A(I-2)
         ENDIF
      ENDDO
      END
C END FILE FIB1.F
```

User can pass fortran source files and functions to F2PY and perform three major tasks:
1. Generate the [signature file](https://numpy.org/doc/stable/f2py/signature-file.html#fortran-c-routine-signatures) which can be used to generate the C/API extension module, done by `-h` flag.
```
f2py fib.f -h fibsign.pyf
```
It will produce a header file `fibsign.pyf` which the user can edit to optimize wrapper functions, to make them "smarter" and more "pythonic".

2. Generate the C/API extension module, either from a signature file or from a fortran source file, done by `-m` flag.
```
f2py fib.f -m fib
# OR
f2py fibsign.pyf -m fib
```
The first one will use create C wrapper from `fib.f` directly, `f2py` will intelligently sense input and output arguments and create the wrapper. The second one will use the signature file to create the modified wrapper.

3. Compile the C/API file and fortran source file to produce a shared library that can be imported in Python, done by `-c` flag.
```
f2py fib.f -c fib.c -m fib
```
Here `-m` flag provides the name of the module, without which the C file generate will be named like "untitled.cpython-310-x86_64-linux-gnu.so", and you would have to import it using `import untitled` (which is pretty funny).

## The current F2PY frontend

The current F2PY frontend can be found in [numpy/f2py/f2py2e.py](https://github.com/numpy/numpy/blob/main/numpy/f2py/f2py2e.py). This handwritten parser was written by [Dr. Pearu Peterson](https://github.com/pearu) way before `argparse` was introduced. It is not easy to understand for a beginner, so I will try to explain it in a few steps.

There are two important functions that run the entire program - [run_main()](https://github.com/numpy/numpy/blob/6032e0ab72be408497c14fd5b865bb25a4c800e7/numpy/f2py/f2py2e.py#L411) and [run_compile()](https://github.com/numpy/numpy/blob/6032e0ab72be408497c14fd5b865bb25a4c800e7/numpy/f2py/f2py2e.py#L411).

| Flag | Activates function |
|:----:|:----------------:|
| `-c` | run_compile() |
| `-m`, `-h` | run_main() |

#### run_main()

As I said above, `f2py` performs three major tasks. This function handles the first two tasks - generating the signature file and generating the C/API extension module. It is called by the `f2py` if the  `argv` array contains the doesn't contain the `-c` flag. It uses [scaninputline()](https://github.com/numpy/numpy/blob/6032e0ab72be408497c14fd5b865bb25a4c800e7/numpy/f2py/f2py2e.py#L179) to parse the `argv` array to get fortran source files and functions, and set a lot of internal variables accordingly (visible in [this line](https://github.com/numpy/numpy/blob/6032e0ab72be408497c14fd5b865bb25a4c800e7/numpy/f2py/f2py2e.py#L181)). 

`run_compile` then calls [crackfortran.py](https://github.com/numpy/numpy/blob/6032e0ab72be408497c14fd5b865bb25a4c800e7/numpy/f2py/crackfortran.py) using [callcrackfortran()](https://github.com/numpy/numpy/blob/6032e0ab72be408497c14fd5b865bb25a4c800e7/numpy/f2py/f2py2e.py#L330) function. If you have studied about compiler, you might know about [abstract syntax trees (AST)](https://en.wikipedia.org/wiki/Abstract_syntax_tree). An AST is a tree structure that represents the source code. Here, `crackfortran` generates a similar struture out of the fortran source code, which is then used by [buildmodules](https://github.com/numpy/numpy/blob/2d4452477245f1332b607c2899a12f6e398a8675/numpy/f2py/f2py2e.py#L366) to either generate the signature file or the C wrapper.

#### run_compile()

This is a really tricky function to understand. It is called by the `f2py` if the `argv` array contains the `-c` flag. It parses the command line again without the use of `scaninputline()`, basically filtering out all the possible flags and options, and then searching for the fortran, `.pyf`, or object files. It then passes the source files and related flags to numpy.disutils where C wrapper is generated and shared library is compiled and linked to produce a Python extension module.

This function needs a lot of work. All the flag filtering is needs to be done by another function, which should be common to both `run_compile()` and `run_main()`. This function also uses regex to parse the command line which is hard to understand. The code is not self-explanatory or modular. It is essentially a bohemoth that does the entire job from parsing the command line to creating extension module in a single shot. 

## The new F2PY frontend
The NumPy team is aiming to make F2PY to more user and developer friendly. F2PY will be soon shifting to a modern `argparse` based CLI. An ongoing implementation can be found in [Rohit Goswami's fork](https://github.com/numpy/numpy/blob/argparse_f2py/numpy/f2py/f2pyarg.py) (the core function to handle flags is remaining). The new frontend will leverage [subparser](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.add_subparsers) functionality of `argparse` to handle the three major tasks. 
#### The main parser
[The main parser](https://github.com/numpy/numpy/blob/argparse_f2py/numpy/f2py/f2pyarg.py#L226) will accept fortran source files and functions to generate C wrapper for and general flags related to module name (`-m`), documentation generation(`--rest-doc`), incuding other header files (`--include-paths`), verbosity (`--verbose`, `--quiet`) etc.

[The build helper subparser](https://github.com/numpy/numpy/blob/ade978b9f7de5cb42fb4e2573972b128158aa41e/numpy/f2py/f2pyarg.py#L263`) will activated by `-c` flag to generate the shared object file which can be imported in Python. It will handle all the build related options like fortran compiler and its flags (`--fcompiler`, `--f77exec`, `f77flags`), architecture optimization (`--arch`), linking (`--link-<name>`)etc. 

[The wrapper generation subparser](https://github.com/numpy/numpy/blob/ade978b9f7de5cb42fb4e2573972b128158aa41e/numpy/f2py/f2pyarg.py#L264) will activated by the `-h` flag to generate the signature file.

## Why do we need a new `argparse` implementation?
One may raise a question about why we need a new `argparse` implementation. If the CLI has been working fine for more than a decade, why not leave it as it is?
I am still a newbie to `f2py` and fortran, but there are a lot of ways we can improve F2PY, like adding the derived types support, improving syntax tree mechanism. The current development team will pass on F2PY to new developers, and it will be worthwhile to modernise and make its functionalities granular. A better codebase and documentation will facilitate easier development and maintenance. Making it more modular will increase flexibility and will allow other programs to use specific functionalities of F2PY exclusively (like how SciPy uses `--build-dir` flag to generate shared object file from its `pyf` files). It will also be much easier to enhace the functionality of the CLI. A developer could just go add a new flag to the appropriate subparser and link to the handling function. Here are the three parsers and functionalities they provide:

Modernising F2PY is a difficult task as its essentially legacy code. Howevery it is essential for the future development and maintainence of this incredible tool.