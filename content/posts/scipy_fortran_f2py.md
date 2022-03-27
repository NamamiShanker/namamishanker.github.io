+++ 
draft = false
date = 2022-03-24T14:01:12+05:30
title = "Why are Numpy and Scipy so fast? And how can I create my superfast Python library?"
description = "Scipy is a thin wrapper on top of fast Fortran code. Learn how Scipy provides high level interface to the extremely fast and efficient BLAS and LAPACK libraries."
slug = ""
authors = ["Namami Shanker"]
tags = ["Fortran", "F2PY", "Scipy"]
categories = ["Informational", "Programming"]
externalLink = ""
series = ["Beginning with Scipy"]
+++


Numpy and Scipy are industry leading Python libraries for fast numerical computation, providing speed and accuracy matching MATLAB and highly optimized low-level numerical computation libraries. In this blog I'll explain to you how they work and how can you create your own superfast Python library.

## SciPy

The name SciPy is a well-known scientific computation industry. Every engineer, researcher, teacher or student who has used Python has used Scipy at least once or more in their career. SciPy is a collection of mathematical algorithms and convenience functions built on the NumPy extension of Python. Scipy is a thin wrapper on top of fast Fortran libraries and it provides a high-level interface to the extremely fast and efficient BLAS and LAPACK libraries written in Fortran.

### BLAS and LAPACK (Historical Context)

I thought I would take this oppurtunity to introduce you to the [BLAS](http://www.netlib.org/blas/) and [LAPACK](http://www.netlib.org/lapack/) libraries. You cannot find a good summary of these libraries with historical context online. I hope this article will be a short note of introduction to many beginners in the future. You can skip this section if you are not interested.

**B**asic **L**inear **A**lgebra **S**ubprogram (BLAS) is a collection of linear algebra routines written in Fortran. It is a very fast, effcient and well-tested library that has stood the test of time in the programming industry. BLAS was initially written by Charles Lawson and Richard Hanson (who, for some reason, have disappeared from the internet) in the 70s during their time in JPL. They realized it was necessary to standardize and get reusable software for all the platforms, so they embarked on a path to standardize a set of routines, get the community involvement, and create a standard library through community consensus.

They created a set of routines with an interface and a reference implementation. The key thing was not its implementation but the interface, which was simple to use, covered many options and was able to handle single, double, complex and double complex arithmetic. The reference implementation specified what the operation should do, and now people just had to implement that efficiently on their machine, and the people started to do that. 

As I told you, people started to write their own implementation of optimized BLAS. Two standardized versions of BLAS were released subsequently in 1987 and 1989. These were called BLAS Level 2 and Level 3, respectively, which expressed higher kinds of operations. Instead of vector operations, these expressed matrix-vector operations and matrix-matrix operations are critical to gaining performance on machines with a memory hierarchy architecture. They have main memory, Level 1 cache, Level 2 cache, vector registers, etc.

**L**inear **A**lgebra **PACK**age was created by [Dr. Jack Dongarra](https://en.wikipedia.org/wiki/Jack_Dongarra) in collaboration with [James Demmel](https://en.wikipedia.org/wiki/James_Demmel) and several other members on top of  Level 2 and Level 3 BLAS libraries. While BLAS is a collection of low-level matrix and vector arithmetic operations (“multiply a vector by a scalar”, “multiply two matrices and add to a third matrix”, etc ...), LAPACK is a collection of higher-level linear algebra operations. Things like matrix factorizations (LU, LLt, QR, SVD, Schur, etc.) that are used to do things like “find the eigenvalues of a matrix” or “find the singular values of a matrix”, or “solve a linear system”.

### Why is SciPy so fast?

Python is both a compiled and interpreted language. Its code is compiled to a *.pyc* file, which contains the bytecode. The bytecode is then interpreted and run on the machine. This process makes Python much slower than compiled languages like C, C++, Fortran, etc. Therefore, when it comes to scientific computing, it is better to write your program in a low-level language like Fortran, C, C++ and compile it to run directly on the CPU.

However, Numpy and Scipy are extremely fast, although they are python libraries. How fast? You can look at [this StackOverflow answer](https://stackoverflow.com/a/51675509/13700295). If you read the entire answer, you will see that Numpy reaches speed as fast as a highly optimized C code (and even faster in some cases).

You might wonder how Scipy and Numpy,  implemented in Python, are fast. The truth is that these libraries are not fully implemented in Python. They are a wrapper on top of the fast Fortran and C program executables, which handle the heavy computations. Scipy and Numpy provide a user-friendly interface to the BLAS and LAPACK libraries. If you download Scipy's source code and build it from scratch for user following [these instructions](https://docs.scipy.org/doc/scipy/dev/contributor/quickstart_ubuntu.html), you will notice output like this in your terminal:- 

![Compilation](https://i.imgur.com/v6rinJc.png)

This is just an output message from the compiler compiling C and Fortran code that ships with Scipy. These files were compiled to produce *.so* files, which is called a **shared object** file. 

![So files](https://i.imgur.com/vFX5MOG.png)

These shared object *.so* files which are dynamically linked during program execution contain can be called directly from your Python program. This is the reason why Scipy is so fast. Lets take a look at the speeds ourselves below.

### How to create your own superfast Python library?

Suppose you want to create your own super-fast Python library that computes the Fibonacci series. Obviously, you can write it in Python, but you can write your code in Fortran and call it in your Python library, making it faster.

Here comes in F2PY. F2PY is a commandline that provides a connection between Python and Fortran languages. It makes that job of calling Fortran code in Python much more straightforward. You can write your Fortran code and add special comments starting with `Cf2py` to tell F2PY how  you want Python interpreter to call your Fortran code. You can then compile your Fortran code to a shared object file and call it from your Python library. F2PY is a fantastic tool, and you can read more about it [here](https://numpy.org/doc/stable/f2py/).

Create a file `fib3.f` with the following code:-

{{< highlight fortran >}}
C FILE: FIB3.F
      SUBROUTINE FIB(A,N)
C
C     CALCULATE FIRST N FIBONACCI NUMBERS
C
      INTEGER N
      REAL*8 A(N)
Cf2py intent(in) n
Cf2py intent(out) a
Cf2py depend(n) a
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
C END FILE FIB3.F
{{< /highlight >}}

This contains the code for a subroutine `FIB` that calculates the first N Fibonacci numbers. Building the extension module can be now carried out in one command:

```
python -m numpy.f2py -c -m fib3 fib3.f
```
Do mind that you would need appropriate Fortran compilers for this. This will create a shared object file in your directory. You can now import this in your Python program with `import fib3` and call the function using `fib3.fib(n).

Create a file `fib1.py` with the following code:-

{{< highlight python >}}
import fib3
import time

start = time.time()

n: int = 1000

print(time.time() - start)
{{< /highlight >}}

This will print the time to calculate the first 1000 Fibonacci numbers.

To compare it with a simple Python program calculating Fibonacci numbers, we will create another file `fib2.py` with the following code:-

{{< highlight python >}}
#!/usr/bin/python

n: int = 1000
import time

start = time.time()

a: list = [0, 1]

a.extend(a[i - 1] + a[i - 2] for i in range(2, n + 1))

print(time.time() - start)
{{< /highlight >}}

Now, lets run both the files and compare the results:-

{{< highlight bash >}}
$ python fib1.py 
	1.2159347534179688e-05
$ python fib2.py 
	0.00019550323486328125
{{< /highlight >}}

Our Fortran calling Python code is much faster than the natively  implemented Python code, around 15 times faster.

You can now feel the gravity of why you want computations of your scientific library to be done in Low-Level Languages. Scipy ships with highly optimized Fortran and C code, which are then fed to F2PY to generate Python extension modules. For example, if you view the `scipy/interpolate/fitpack` directory in Scipy's source code, it is filled with Fortran subroutines. F2PY is used to build an extension module out of these subroutines, which you actually call in your Python code.

![Fortran code in Scipy repository](https://i.imgur.com/zEf3rUQ.png)