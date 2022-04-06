+++ 
draft = false
date = 2022-04-03T10:22:28+05:30
title = "F2PY Tests"
description = "Lets look into the test suite of F2PY, how to use it, and how you can add tests to it."
slug = ""
authors = ["Namami Shanker"]
tags = []
categories = []
externalLink = ""
series = []
+++

I have talked previously about [building fast libraries](https://namamishanker.github.io/posts/scipy_fortran_f2py/) for Python using Fortran. The key software that facilitates easy integration of Fortran code into Python is [F2PY](https://numpy.org/doc/stable/f2py/). The user can write fast Fortran code with F2PY specific comments and compile it using F2PY to produce a high-speed ready-to-use Python library.

## F2PY

F2PY is an open-source utilty tool that provides an easy connection between Python and Fortran languages. It was written in the late 90s by [Dr. Pearu Peterson](https://github.com/pearu). The first version of this software was released in 1999 when it was called Fortran to Python Interface Generator (FPIG). It remained a stand-alone software till 2007 when it was moved to the NumPy project. F2PY is currently maintained by the NumPy developer team and users can install numpy to use it.

### Current status

F2PY is an old software and it has not had any major releases since 2009. If you browse its [codebase in the current state](https://github.com/numpy/numpy/tree/eecd16210ffb487d4e862c71e86cc6f88f2e9aa2/numpy/f2py), you will find it a bit difficult to read, understand and contribute to. Recently, pushes have been made by [Rohit Goswami](https://github.com/HaoZeke/) to modernise its CLI and backend compilation process. Probably the F2PY you will get to use will be a more user-friendly and faster CLI.

### Codebase

If you explore the current directory [`numpy/f2py`](https://github.com/numpy/numpy/tree/main/numpy/f2py) you will open up a directory that looks likes this:-
{{< highlight go "linenos=table,hl_lines=7 9 21,linenostart=1" >}}
.
├── auxfuncs.py
├── capi_maps.py
├── cb_rules.py
├── cfuncs.py
├── common_rules.py
├── crackfortran.py
├── diagnose.py
├── f2py2e.py
├── f2py_testing.py
├── f90mod_rules.py
├── func2subr.py
├── __init__.py
├── __init__.pyi
├── __main__.py
├── rules.py
├── setup.cfg
├── setup.py
├── src
├── symbolic.py
├── tests
├── use_rules.py
└── __version__.py
{{< / highlight >}}

{{< notice info >}} [`crackfortran.py`](https://github.com/numpy/numpy/blob/main/numpy/f2py/crackfortran.py) file is the heart of f2py. It is responsible of reading Fortran code and extracting decleration information which is used to create `.pyf` files. [`f2py2e.py`](https://github.com/numpy/numpy/blob/main/numpy/f2py/f2py2e.py) stands for **F2PY Second edition**. It is the CLI program that sits on top of `crackfortran.py` that provides user interface. {{< /notice >}}

We will talk mostly about the test suite of F2PY in this blog which consists of the `tests` directory. 

### Testing f2py

The file tree of [tests](https://github.com/numpy/numpy/tree/main/numpy/f2py/tests) directory presents us with these:-

{{< highlight go "linenos=table,hl_lines=3 12,linenostart=1" >}}
./tests/
├── __init__.py
├── src
│   ├── abstract_interface
│   ├── array_from_pyobj
│   ├── // ... several test folders
│   └── string
├── test_abstract_interface.py
├── test_array_from_pyobj.py
├── // ... several test files
├── test_symbolic.py
└── util.py
{{< / highlight >}}

Files starting with `test_` contain tests for various aspects of f2py from parsing Fortran files to checking modules' documentation. `src` directory contains the Fortran source files upon which we do the testing. `util.py` is an interesting file. It contains superclass for class based tests in f2py which we will talk about later.

### Adding tests

If you wander through the `tests` folder snooping inside the files, you will find most tests are classes that inherit from `util.F2PyTest` superclass. This superclass is present in `util.py` file. 

{{< highlight python "linenos=table,linenostart=327" >}}
class F2PyTest:
    code = None
    sources = None
    options = []
    skip = []
    only = []
    suffix = ".f"
    module = None
    module_name = None

    def setup(self):
        if sys.platform == "win32":
	.
	.
	.
{{< / highlight >}}

This superclass contains many helper functions responsible for parsing and compiling test source files. Its child classes can override its `sources` data member to provide their own source files. This superclass will then compile the added source files upon object creation and their functions will be appended to `self.module` data member. Thus, the child classes will be able to access the functions specified in source file by calling `self.module.[fortran_function_name]`.

Anyone looking to add more tests can follow the same strategy. He/She can add new test classes to the appropriate file in `tests` directory inheriting from `util.F2PyTest` superclass and test the fortran function by calling `self.module.[fortran_function_name]`.

#### Example

Suppose you found [this bug](https://github.com/numpy/numpy/issues/14625) in f2py. You corrected it and now want to add tests for it. How do you do that?  

Well, first of all you would have to identify the correct file to add the test. This bug was related to incorrect parsing of Fortran code where subroutines ending with `endsubroutine` were not parsed and compiled. Upon importing such subroutines, the user would encounter a missing attribute error. Since `crackfortran.py` file is responsible for parsing, we add the test to `test_crackfortran.py` file. The testing source file should also be created in `numpy/f2py/tests/src/crackfortran/` directory. Similarly other tests can be added to already present `test_` files based on user's discretion.

First lets create a file `my_source_file.f` containing subroutines ending with `endsubroutine`. This file contains two self-explanatory Fortran subroutines :-

{{< highlight fortran "linenos=table,linenostart=1" >}}
        subroutine subb(k)
          real(8), intent(inout) :: k(:)
          k=k+1
        endsubroutine

        subroutine subc(w,k)
          real(8), intent(in) :: w(:)
          real(8), intent(out) :: k(size(w))
          k=w+1
        endsubroutine
{{< / highlight >}}

We will compile this Fortran code and test that it is working. 

Open `test_crackfortran.py` to add the test for the previously added Fortran file:-

{{< highlight python "linenos=table,linenostart=1" >}}
class TestNoSpace(util.F2PyTest):
    sources = [util.getpath("tests", "src", "crackfortran", "my_source_file.f")]

    def test_module(self):
        k = np.array([1, 2, 3], dtype=np.float64)
        w = np.array([1, 2, 3], dtype=np.float64)
        self.module.subb(k)
        assert np.allclose(k, w + 1)
        self.module.subc([w, k])
        assert np.allclose(k, w + 1)
{{< /highlight >}}

We override the `sources` data member to provide the source file. The source files are compiled and subroutines are attached to module datamember when the class object is created. The `test_module` function calls the subroutines and tests their results.

### Conclusion

The current test suite of F2PY is a bit old. Modern python programmers accustomed to `pytest` might find its lack of fixtures, setups and tear downs little unappealing. However, once you get the hang of it, its easy to understand and improve. It is not an exhaustive test suite, but it is improving everyday. A new test suite for testing the CLI program `f2py2e.py` is being developed with modern `pytest` style testing and you can have a look at it in [Rohit Goswami's fork](https://github.com/HaoZeke/numpy/blob/f2py2eTests/numpy/f2py/tests/test_f2py2e.py). F2PY and its test suite are sure to undergo heavy refactoring in the coming times.