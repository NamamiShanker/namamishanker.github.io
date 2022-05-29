+++ 
draft = false
date = 2022-05-30T00:56:39+05:30
title = "GSoC'22 project introduction and tentative plan"
description = "GSoC'22 Project Introduction and tentative plan"
slug = ""
authors = ["Namami Shanker"]
tags = ["GSoC22", "F2PY", "Numpy"]
categories = ["GSoC22", "Blog"]
externalLink = ""
series = ["GSoC'22"]
+++

I applied for Google Summer of Code 2022 to work with NumPy's F2PY maintainence team and improve F2PY under Python Software Foundation. Breifly said, F2PY is a CLI that provides an easy connection between Python and Fortran languagues. I have previously dicussed F2PY on my website on the topics [Scipy, Fortran and F2PY](https://namamishanker.github.io/posts/scipy_fortran_f2py.md) and [F2PY tests](https://namamishanker.github.io/posts/f2py_tests.md).

During my time of application, F2PY's frontend was a handwritten command line parser with handwritten interfacing. The backend compilation was also being done by `numpy.distutils` with is set for deprecation. A project idea was raised by Mr. Rohit Goswami from F2PY's maintainence team to modernise the frontend with `argparse` and replace the backend with `meson` which can be found on [Scipy's GSoC'22 idea list](https://github.com/scipy/scipy/wiki/GSoC-2022-project-ideas).

This first idea in this idea list is precisely what I will be working on this summer. A detailed description of my proposal can be found at [GSoC'22 website](https://summerofcode.withgoogle.com/proposals/details/kirhPArq). Breifly, I will reimplement the frontend of F2PY with argparse library during the first half (which lasts till July end) of GSoC'22. During the second half (during August), I will be working on the backend of F2PY with meson. I also plan on modernising the test suite of F2PY with `pytest` and provide some critical developer's guide/documentation for F2PY.

I will post my weekly updates on this website regarding my progress on this project. A detailed tentative timeline of the action plan is given below. I recommend a thorough read of the proposal before starting. I will however give a brief overview of the project and the plan.

#### Project title - Restructuring the F2PY frontend and replacing the build system

## Predesign

### CLI Design

F2PY uses [f2py2e.py](https://github.com/HaoZeke/numpy/blob/f2py2eTests/numpy/f2py/f2py2e.py) as a CLI interface to parse user input. [f2pyarg.py](https://github.com/HaoZeke/numpy/blob/argparse_f2py/numpy/f2py/f2pyarg.py) is an ongoing re-implementation of this CLI using the “argparse” module, but it needs to be completed. I intend to complete this rewrite and provide a modernised, developer-friendly CLI. Additionally, I plan to deliver developer-guide documentation and a test suite covering the CLI.

### Test suite design

F2PY's current test suite predates “pytest” and does not use fixtures. I shall work on refactoring the existing test suite, using pytest features such as fixtures, setups and tear-downs.

### Backend design

F2PY currently uses “np.distutils” for compiling generated C files. Since it is to be deprecated, I intend to work on the addition of a flag that will switch the build backend to Meson for
F2Py modules. As discussed with the NumPy developer community and noted in the documentation, the goal is to transition gradually. F2PY will support both “np.distutils” and “meson” building options for testing, after which the former can be removed completely.

## Tentative project timeline

#### Week 1 - 2 (June 13 – June 25)
- Complete the existing f2pyarg.py. Implement missing functionalities like
compile, link etc.
- Refactor the “run_compile” method for the “np.distutils” backend.
- Discuss linking issues and to-be-deprecated flags with the mentor and
implement solutions.
#### Week 3 - 4 (June 27 – July 9)
- Start working on existing test_f2py2e.py. Implement remaining tests,
enhance existing failing tests and increase test coverage.
- Communicate with the mentor and maximise tests for f2py CLI.
#### Week 5 (July 11 – July 16)
- Add developer documentation for F2PY. Will contain files
“f2py-developer.rst” and “f2py-test.rst” explaining file structure, the
functionality of F2PY and its test suite, respectively.
- Start refactoring F2PY’s test suite in a modern fixture-based style.
#### Week 6 (July 18 – July 23)
- Finish refactoring the test suite.
- Fix bugs and update documents.
### First evaluation period (July 25 – July 29)
- Deliver the implemented f2py CLI, implemented tests and developer
guide.
- The F2PY CLI will now be more open to study and contributions by
developers.
#### Week 8 - 9 (Aug 1 – Aug 13)
- Create a “meson.build.src” file.
- Update “run_compile” method to add meson building option.
#### Week 10 (Aug 15 – Aug 20)
- Fix bugs and update documentation.
#### Week 11 -12 (Aug 22 – Sept 4)
- Pull request for code review and merge.
- Buffer time for any unexpected delays.
### Final evaluation period (Sept 5 - 12):
- Deliver the working implementation of the Meson build system for F2PY.
- Wrap up the project and submit the final evaluation of my mentor.
