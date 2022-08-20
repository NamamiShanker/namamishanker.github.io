---
title: "Week 8-9: Meson backend for F2PY"
date: 2022-08-20T13:26:08+05:30
draft: false
slug : ""
authors : ["Namami Shanker"]
tags : ["F2PY", "argparse", "Meson", "cli"]
categories : ["F2PY", "Programming", "Meson"]
externalLink : ""
series : ["GSoC'22"]
---

# Week 8-9: Meson backend for F2PY

## F2PY and its current status

I have talked a lot about [F2PY](https://numpy.org/doc/stable/f2py/) during past 2 months as my Google Summer of Code progressed. Simply explained, F2PY is a command line tool that provides easy connection between Fortran languages and Python. It facilitates creating Python C/API extension modules from Fortran files that can be called and imported in Python. It gives you the [speed of Fortran and the flexibility of Python](https://namamishanker.github.io/posts/scipy_fortran_f2py/). The way F2PY does this is **by creating a C wrapper for your Fortran code** with Python-C API which is interpretable by Python. The Fortran sources and C wrapper can be built together by any build tools (CMake, Meson etc) to produce a shared library file (.so) allowing you to import your Fortran subroutines as if you were importing any other Python module. During the period of 13th June to 31st July, I modernised the frontend of F2PY with [argparse](https://docs.python.org/3/library/argparse.html). From August, I started working on integrating a Meson build system for building shared library from C wrapper and Fortran source code.

Though F2PY provides [options to build the Python module](https://numpy.org/doc/stable/f2py/usage.html#building-a-module), building the module is not the main selling point of F2PY. It is efficiently generating C wrappers for all kinds of features exclusive to Fortran (eg- [derived types](https://fortran-lang.org/learn/quickstart/derived_types), [kind parameter](https://stackoverflow.com/questions/838310/fortran-90-kind-parameter#:~:text=The%20KIND%20of%20a%20variable,required%20by%20the%20Fortran%20standard.)) all through F2PY's internal logic. F2PY currently uses [NumPy's Distutils](https://numpy.org/doc/stable/reference/distutils.html) as a build tool but `distutils` will soon be deprecated, consequently deprecating NumPy's `distutils`. Thus, F2PY needs to move on to other build systems like [CMake](https://cmake.org/) or [Meson](https://mesonbuild.com/), and it is first going for Meson.

## Build systems for F2PY

As I said, module building options present in F2PY are not a core F2PY functionality. These options are simple settings which help the user specify how they want their build to be produced. For example, the user can enable debug mode, specify optimization levels, or specify a required custom header file etc. I have written an extensive article exploring build options of F2PY and how they can be implmented with Meson here: [Week 2: Moving F2PY to Meson build system](https://namamishanker.github.io/posts/f2py-meson-backend/). Today, we will discuss how I changed the existing structure for integrating build system dependency into F2PY for building Python modules.

The general flow of generating a Python module with F2PY is as follows:

![](https://i.imgur.com/Fjsz8DG.png)

In the above diagram, we can use any build system. F2PY's docs even provides extensive articles on how to [build your Python modules using different build systems](https://numpy.org/doc/stable/f2py/buildtools/index.html).

To add builtin support for a build system all you need to do is parse F2PY's compilation flags, and provide corresponding configuration to your build system, and voila, you can now build using your favourite tool. Therefore, any build system can be **integrated** in the above flow by providing it correct configuration, source files, and C wrapper to produce the output module.

I have followed the same principle for integrating Meson into F2PY. Let's dive into the changes and discuss the details.


### A general superclass structure

First and formost, I have implemented a class which can be inherited by "build-system" specific classes. The object of this superclass can be constructed by providing compilation specific information as mentioned below:-

```python
class Backend(ABC):
	"""
 	Superclass for backend compilation plugins to extend.
 
 	"""

	def __init__(self, module_name, fortran_compiler: str, c_compiler: str,
        f77exec: Path, f90exec: Path, f77_flags: list[str],
        f90_flags: list[str], include_paths: list[Path],
        include_dirs: list[Path], external_resources: list[str],
        linker_libpath: list[Path], linker_libname: list[str],
        define_macros: list[tuple[str, str]], undef_macros: list[str],
        debug: bool, opt_flags: list[str], arch_flags: list[str],
        no_opt: bool, no_arch: bool) -> None:
		"""
		The class is initialized with f2py compile options.
		The parameters are mappings of f2py compilation flags.

		Parameters
		----------
		module_name : str
			The name of the module to be compiled. (-m)
		fortran_compiler : str
			Name of the Fortran compiler to use. (--fcompiler)
		c_compiler : str
			Name of the C compiler to use. (--ccompiler)
		f77exec : Pathlike
			Path to the fortran compiler for Fortran 77 files (--f77exec)
		f90exec : Pathlike
			Path to the fortran compiler for Fortran 90 and above files (--f90exec)
		.
		.
		.

	def numpy_install_path(self) -> Path:
		"""
		Returns the install path for numpy.
		"""
		return Path(numpy.__file__).parent

	def numpy_get_include(self) -> Path:
		"""
		Returns the include paths for numpy.
		"""
		return Path(numpy.get_include())

	def f2py_get_include(self) -> Path:
		"""
		Returns the include paths for f2py.
		"""
		return Path(f2py.get_include())

	@abstractmethod
	def compile(self, fortran_sources: Path, c_wrapper: Path, build_dir: Path) -> None:
		"""Compile the wrapper."""
		pass

```
As you can see, this superclass elementarily [receives values of F2PY flags directly from the frontend CLI](https://github.com/NamamiShanker/numpy/blob/db578560a5309a1038b0a7ba309b527155f178c0/numpy/f2py/f2pyarg.py#L867) and it is the responsibility of `compile` function to implement a logic to parse the information provided in the flags and build Python module. This superclass provides some useful paths containing important NumPy and F2PY C library which would be linked to your Python module during building. But more importantly, it provides an **abstract method** `compile` which should be implemented by children classes for their specific build system.

The approach is simple and intuitive - provide sources files, C wrapper and compilation specific information and build the Python module. The `compile` method will handle all the compile specific einformation provided to F2PY.

### The Meson subclass structure

So now, time for us to create a subclass which will inherit the above discussed `Backend` superclass and provide build producing capabilies using Meson. Now the most important feature to implement would be generating a [meson.build](https://mesonbuild.com/Meson-sample.html) file. We provide build configuration through `meson.build` to Meson which in turn produces a `build.ninja` file which the **Ninja build system** uses to build the module. Meson itself is not a build tool, but an domain specific language to decalare the build system like CMake. This article [Design rationale for Meson](https://mesonbuild.com/Design-rationale.html#overview-of-the-solution) is provides excellent insight into understanding what Meson actually is.

Lets now define our problems and goals: **We want to dynamically generate a Meson configuration (a `meson.build` file) from F2PY's compilation flags, set the correct environment, and run Meson** to create the Python extension module.

Our problem has a seperate subproblem - **Generating `meson.build` file**. The most intuitive way I could go about solving this design issue was by creating a seperate class for generating the build file.

#### Generating `meson.build` file

The old age problem of dynamically generating text has a simple solution for our case - [formatting a string templating](https://realpython.com/python-string-formatting/). by quite a few libraries in Python - [Jinja2](https://jinja.palletsprojects.com/en/3.1.x/), [Mako](https://www.makotemplates.org/) etc. NumPy itself comes bundled with [Tempita](https://pypi.org/project/Tempita/), a very old string templating library. However, we chose to use [Python's builtin template string](https://docs.python.org/3/library/string.html#template-strings) instead of relying on the mentioned libraries.

{{< notice info >}} Python's in-built Template strings provided easy simpler substitutions. Jinja is a very advanced templating library and would be an overkill for our purposes. On the other hand Tempita is no longer maintained. {{< /notice >}}

We have created a [meson.build.src](https://github.com/NamamiShanker/numpy/blob/meson_backend/numpy/f2py/backends/src/meson.build.src) file which contains template for generating actual `meson.build` files. The templating is handled by a seperate class whose source code you can read here: [MesonTemplate class](https://github.com/NamamiShanker/numpy/blob/meson_backend/numpy/f2py/backends/meson_backend.py#L9). It has a [substitutions](https://github.com/NamamiShanker/numpy/blob/41b6949abdd843be0f570fba60675e2c2063c770/numpy/f2py/backends/meson_backend.py#L21) dictionary as data member which is populated by the various methods present in the classes. The [generate_meson_build](https://github.com/NamamiShanker/numpy/blob/41b6949abdd843be0f570fba60675e2c2063c770/numpy/f2py/backends/meson_backend.py#L49) calls these methods one by one and the final `substitutions` dictionary is passed to the `Template` object. This pipeline sort of structre was inspired by **monad design pattern**.

#### Generating build

So now that we have solved the problem of generating Mesob build file, lets try to use run Meson with our dynamically generated Meson build file. This can be done in three steps:

1. Set `FC` and `CC` environment variables for telling Meson which compilers to use.
2. Generate `meson.build` file using [MesonTemplate](https://github.com/NamamiShanker/numpy/blob/db578560a5309a1038b0a7ba309b527155f178c0/numpy/f2py/backends/meson_backend.py#L10) class in the build directory.
3. Change working directory to `build_dir` and setup Meson build directory specifying optimization and debug levels.
4. Compile the Meson build directory and produce the Python module - a shared library file with `.so` extension.
5. Move the `.so` file to root so that the user can import it.

### How will we our frontend `f2pyarg` communicate with the Backend class?

We have created a folder **[backends](https://github.com/NamamiShanker/numpy/tree/meson_backend/numpy/f2py/backends)** in the `f2py` directory within Numpy. This directory will contain our `Backend` superclass as well as build-system specific subclasses. The tree of the folder looks as following: 
```
backends/
├── backend.py
├── __init__.py
├── meson_backend.py
└── src
    └── meson.build.src
```

The source code of `__init__.py` is shown below:

```python
from .backend import Backend
from .meson_backend import MesonBackend

backends = {
	'meson': MesonBackend
#	'cmake': CMakeBackend
}
```

Following the flow-diagram shown above, we need to pass the generated C wrapper and Fortran sources files after instantiating our Backend class. The Backend can be called and instantiated from `f2pyarg` simply as follow:

```
c_wrapper = generate_files(f77_files + f90_files, module_name, sign_file)
if c_wrapper and args.c:
	backend: Backend = backends.get(args.backend.value)(module_name=module_name, ...args)
	backend.compile(f77_files + f90_files, c_wrapper, build_dir)
```
This approach is more scalable than currently existing approach of compiling with `numpy.distutils` where passing `-c` flag causes F2PY to transform into a frontend `distutils`. Meson building APIs can be exposed seperately by writing them in `f2py/service.py`.

## Retrospection of the changes

We introduced Meson to F2PY because the current build tool `distutils` was going to be deprecated. Here I found the oppurtunity to simplify how F2PY uses build system. If you try to understand my flow and the existing `distutils` building structure, I think you would find this approach much intuitive, manageable and scalable with respect to adding more build tools. Still a lot of work is left and we just yet can't sit down to admire the lovely, dark and deep woods.

The next step would be to support `--f77exec, --f90exec, --f90flags, --f77flags` such that Meson can handle seperate compilers for Fortran 77 and Fortran 90 code. It would be followed by writing an extensive test suite and benchmarking the results to verify Meson backend is robust and ready for public usage.