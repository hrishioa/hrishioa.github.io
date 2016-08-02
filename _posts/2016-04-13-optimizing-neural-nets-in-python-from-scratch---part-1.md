---
layout: post
title: "Optimizing Neural Nets in Python from Scratch"
date: 2016-04-13 22:26:17
image: '/assets/img/'
description: Part 1 - Looking at CPU Speedup
tags:
- Neural Nets
- ATLAS
- BLAS
- LAPACK
- Numpy
- Python
categories:
twitter_text:
---

### Background

When running basic matrix operations in Python, numpy is the most versatile and easily accessible library out there. For this reason, my first [implementation](https://github.com/hrishioa/NN_Scratch/blob/master/NeuralNet_Perceptron.ipynb) of a perceptron was done in Numpy. It wasn't until I started working on the [MNIST Database](http://yann.lecun.com/exdb/mnist/) that I realised it was painfully slow. The very first run, using batch gradient descent on a thousand runs took **1 day and 18 hours**. Yeah. It was here that I started looking for ways to improve performance. There are a wealth of articles and guides online about how to do this, but working from a Mac meant that only 5% of these were useful to me. And while there were a number of guides that would yield results when used together, I found myself lacking the depth of knowledge in machine-level libraries needed to compile, link and use them. I just wanted to speed up my neural net. Eventually I did, and here's a small compilation of the steps it took to achieve this speedup, and the many options available to do so.

## Part 1 - CPU Optimization

Speeding up Numpy on a CPU level requires understand how this speedup is obtained. In my particular case, the slowest operations I were doing were matrix operations, which lend themselves well to parallel implementations. They have become so commonplace that most modern CPUs (forget GPUs for now) have dedicated hardware components that speed up matrix operations. Part of this power comes from the [SIMD](https://en.wikipedia.org/wiki/SIMD) module, which can perform parallel operations on data sets while using the same instruction. Another part of this is the floating point unit found on most general-purpose CPUs, which optimizes floating point operations to the point of insanity. Here's an example:

~~~ python
import numpy as np
a = [np.random.rand(100,100) for i in xrange(0,1000)]
b = [np.random.rand(100,100) for i in xrange(0,1000)]
%time for i in xrange(0, 1000): np.dot(a[i], b[i])
~~~

~~~ shell
CPU times: user 97.5 ms, sys: 2.01 ms, total: 99.5 ms
Wall time: 101 ms
~~~

Keep in mind that this is still unoptimized. However, guess how long the following piece of code takes:

~~~ python
a = [np.random.randint(0,100,(100,100)) for i in xrange(0,1000)]
b = [np.random.randint(0,100,(100,100)) for i in xrange(0,1000)]
%time for i in xrange(0, 1000): np.dot(a[i], b[i])
~~~

It's integers, within a much smaller range than that of real numbers, and yet:

~~~ shell
CPU times: user 924 ms, sys: 7.74 ms, total: 931 ms
Wall time: 948 ms
~~~

Almost ten times the cost. Already it's worth noting that any operations you do would be better served on floating point, even without other optimizations. Even a band-aid fix works in a pinch:

~~~ python
a = [np.array(np.random.randint(0,100,(100,100)), dtype=float) for i in xrange(0,1000)]
b = [np.array(np.random.randint(0,100,(100,100)), dtype=float) for i in xrange(0,1000)]
%time for i in xrange(0, 1000): np.dot(a[i], b[i])
~~~

~~~ shell
CPU times: user 107 ms, sys: 1.28 ms, total: 109 ms
Wall time: 109 ms
~~~

There. Much better.

Now, that we understand how it works, let's look under the hood, for how numpy works:

~~~ python
np.config.show()
~~~
~~~shell
atlas_threads_info:
  NOT AVAILABLE
blas_opt_info:
    extra_link_args = ['-Wl,-framework', '-Wl,Accelerate']
    extra_compile_args = ['-msse3', '-I/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.Internal.sdk/System/Library/Frameworks/vecLib.framework/Headers']
    define_macros = [('NO_ATLAS_INFO', 3)]
atlas_blas_threads_info:
  NOT AVAILABLE
openblas_info:
  NOT AVAILABLE
lapack_opt_info:
    extra_link_args = ['-Wl,-framework', '-Wl,Accelerate']
    extra_compile_args = ['-msse3']
    define_macros = [('NO_ATLAS_INFO', 3)]
atlas_info:
  NOT AVAILABLE
lapack_mkl_info:
  NOT AVAILABLE
blas_mkl_info:
  NOT AVAILABLE
atlas_blas_info:
  NOT AVAILABLE
mkl_info:
  NOT AVAILABLE
~~~

There's a lot of stuff missing. I can't go into detail, but here's a quick rundown of what everything is:

* *ATLAS* stands for **Automatically Tuned Linear Algebra Software**. It\'s one of the fastest libraries around, but if you're looking to squeeze the last drop of performance out of your code, I'd recommend looking into detailed benchmarks for each library. ATLAS has been around for a while, and works well in most circumstances. I'd recommend it on OS X, while OpenBLAS has been known to perform better on linux systems, although some instances of [segfaults](https://github.com/xianyi/OpenBLAS/issues/229) have been observed when running large computations (> 60 GB).
* *BLAS* stands for **Basic Linear Algebra Subprograms**. This is the bare minimum, and you should be surprised if your implementation of numpy does not already come with some version of BLAS. Mac and Ubuntu, as far as I know come with default implementations of both ATLAS and BLAS, so this is the first thing you need.
* *LAPACK* is the **Linear Algebra PACKage**, and it works in tandem with BLAS.
* *OpenBLAS* is a relative newcomer, but it has been gaining on the others in terms of performance.
* Finally, *MKL* is the **Intel Math Kernel Library**, and it allows for massive speedups on Intel processors due to hardware optimizations.

Now, the easiest (and I mean easiest) thing you can do if you're a student is to install the [Enthought Canopy](https://www.enthought.com/products/canopy/) distribution on an Academic License. It comes with ATLAS and MKL enabled and compiled into numpy, and [Theano](http://deeplearning.net/software/theano/) recommends it as the easiest way of getting started. Once installed, `pip install theano` is all that's required to get started (but more on Theano another time). 

I wish I'd found Canopy sooner, which is partly the reason for this post. So if you are a student, stop reading, install Canopy and wait for Part II where we discuss GPU. If not, try importing numpy from python, using `import numpy as np`. If this fails, you're good. If not, and your `np.config.show()` does not include the libraries we discussed, you'll need to uninstall numpy and build it from scratch. You can uninstall using `pip uninstall numpy` (if you don't have pip, I wonder how you have numpy, but you can install pip from [here](https://bootstrap.pypa.io/get-pip.py).). Once this is done, build and link numpy with your choice of BLAS and LAPACK, then continue. [Here](https://github.com/CellCognition/cecog/wiki/Build-build-accelerated-numpy-using-ATLAS-on-Mac-OSX) are the instructions.

Now before we go further, let's look at how things change in terms of config:

~~~ python
np.__config__.show()
~~~
~~~shell
lapack_opt_info:
    libraries = ['mkl_lapack95_lp64', 'mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'mkl_mc', 'mkl_mc3', 'pthread']
    library_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../lib']
    define_macros = [('SCIPY_MKL_H', None)]
    include_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../include']
blas_opt_info:
    libraries = ['mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'mkl_mc', 'mkl_mc3', 'pthread']
    library_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../lib']
    define_macros = [('SCIPY_MKL_H', None)]
    include_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../include']
openblas_lapack_info:
  NOT AVAILABLE
lapack_mkl_info:
    libraries = ['mkl_lapack95_lp64', 'mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'mkl_mc', 'mkl_mc3', 'pthread']
    library_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../lib']
    define_macros = [('SCIPY_MKL_H', None)]
    include_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../include']
blas_mkl_info:
    libraries = ['mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'mkl_mc', 'mkl_mc3', 'pthread']
    library_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../lib']
    define_macros = [('SCIPY_MKL_H', None)]
    include_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../include']
mkl_info:
    libraries = ['mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'mkl_mc', 'mkl_mc3', 'pthread']
    library_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../lib']
    define_macros = [('SCIPY_MKL_H', None)]
    include_dirs = ['/Users/vagrant/src/buildsystem-git/build/master-env/Resources/Python.app/Contents/MacOS/../../../../include']
~~~

All right! Everything except OpenBLAS is installed and compiled against numpy, and if you're astute you'll notice that the 64-bit versions have been correctly identified and used. Now let's look at speeds:

~~~ python
a = [np.array(np.random.randint(0,100,(100,100)), dtype=float) for i in xrange(0,1000)]
b = [np.array(np.random.randint(0,100,(100,100)), dtype=float) for i in xrange(0,1000)]
%time for i in xrange(0, 1000): np.dot(a[i], b[i])
~~~

This time, you can see the speedup happen:

~~~ shell
CPU times: user 201 ms, sys: 1.09 ms, total: 208 ms
Wall time: 65 ms
~~~

The increased CPU times are due to the now-parallel workload imposed by the libraries we just installed. If you extend the workload, you'll notice all cores light up equally. Even on single-core machines, you should have speedup if you're on an architecture that has dedicated hardware optimization for matrix operations. 

You'll notice that the speedups we got by using these libraries, while significant, are still within the same order of magnitude. This is because we went from one to a few more cores, and still haven't hit the limits of parallel computing. But that's it for now. We'll discuss GPU speedups and how they work in Part II.