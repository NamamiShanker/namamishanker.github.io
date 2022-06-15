---
title: "F2PY Frontend"
date: 2022-06-15T21:34:26+05:30
draft: true
---

# F2PY frontend

We will look at the current F2PY frontend, and ongoing re-implementation of it using the `argparse` python library.

## Introduction
F2PY is a command line tool that provides easy connection between Fortran languages and Python. It facilitates creating Python C/API extension modules that can be called and imported in Python. It gives you the [speed of Fortran and the flexibility of Python](https://namamishanker.github.io/posts/scipy_fortran_f2py/).

## The frontend of a command line tool

A command line works by parsing the input on the command line, and then calling correct functions based on the input. The input on the command line is generally a list of positional arguments (like files, folders, keys or data) and flags (like `--help` or `--version`). For example, if you type
```
git --version
```
then the command line tool will call the `git` program, and pass the `--version` flag to it as an argument. Or if you type
```
git add app.py
```
then the command line tool will call the `git` program, and pass the `add` and `app.py` to it as an argument. Internally, `git` will recognize `add` as an option, and `app.py` as a positional argument to `add` option.

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

User can pass fortran source files and functions to F2PY and perform two major tasks:
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
Here `-m` flag provides the name of the module, without which the C file generate will "untitled".