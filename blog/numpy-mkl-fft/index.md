.. title: Numpy plus the Intel MKL for fast fourier transforms
.. slug: numpy-mkl-fft
.. date: 2018-01-03 17:59:00 UTC+00:00
.. tags:
.. category:
.. link:
.. description:
.. type: text
.. author: Will Furnass

One of the nice things about using (Ana)conda's default builds of Numpy, Scipy and Scikit-Learn is that they are [compiled against and depend on Intel's Math Kernel Library](https://docs.anaconda.com/mkl-optimizations/) (MKL),
which provides multi-threaded implementations of functions for linear algebra that satisfy the [BLAS](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms)
and [LAPACK](https://en.wikipedia.org/wiki/LAPACK) APIs.  
This Numpy build is therefore able to distribute the work involved in say a matrix multiplication operation between CPU cores without having to spawn multiple processes and migrate/duplicate data between the address space of those proceses.  We can see that multiple CPU cores are being used by comparing the 'real' (wall-clock) time of a simple matrix multiplication example to the 'user' time, which is the amount of time our code spend running on all CPU cores.  


```
$ conda create -n default_env python=3.6 numpy
$ source activate default_env
$ time python -c 'import numpy as np; x = np.random.random((5000, 5000)); x @ x'                                       
                                                                                                                                                                                                                  
real    0m2.505s
user    0m6.640s
sys     0m0.067s
```

As you can see, the 'real' time is much less than the 'user' time, which would only be possible if multiple cores were used.

The MKL also provides **Fast Fourier Transform** FFT functions but be warned that by default the Anaconda build of Numpy is not able to use multiple threads for FFT functions!

```
$ time python -c 'import numpy as np; shape = 1 * [50000000]; x = np.random.choice(a=1, size=shape); np.fft.ifftshift(np.fft.fftn(np.fft.fftshift(x)))' 

real    0m4.821s
user    0m4.419s
sys     0m0.400s

$ source deactivate
```

However, the Intel Python Distribution's build of Numpy does use multiple threads for FFts.  
The IPD is a separate set of conda packages where things like numpy, scipy, scikit-learn etc have been 
optimised to make better use of Intel libraries such as the MKL and Data Analytics Acceleration Library (DAAL).

```
$ conda create -n ipd_env python=3.6 numpy
$ source activate ipd_env
$ time python -c 'import numpy as np; shape = 1 * [50000000]; x = np.random.choice(a=1, size=shape); np.fft.ifftshift(np.fft.fftn(np.fft.fftshift(x)))'

real    0m1.305s
user    0m3.545s
sys     0m0.899s

$ source deactivate
```

Note that the wall-clock time for the IPD example is less than the
total time spent executing user code on all CPU cores i.e. multiple
cores are being used.
