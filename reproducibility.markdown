---
layout: page
title: Reproducibility
permalink: /reproducibility/
nav_order: 6
---
# Reproducibility
Random numbers are useful for many things in statistics but computers are deterministic machines so random number generators (RNGs) can only generate numbers that "look" random, i.e. have an even distribution. Each time a RNG is invoked it deterministically extracts a number from its initial state (i.e. seed) then deterministically updates its state so that it will be different on the next call. Therefore setting a seed is not thread-safe; without external memory synchronization, it's plausible two threads could generate the same or different random numbers depending on if one thread sees the update to the internal RNG state made by the other thread. Sometimes the multi-threading speedup outweighs the risks of race conditions, but when reproducibility is important you should use a single thread. Most neuroimaging tools (including ANTS, FSL, Freesurfer) can be configured to use 1 thread by setting the following environment variables:
```sh
export ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS=1
export OMP_NUM_THREADS=1
export OMP_THREAD_LIMIT=1
export MKL_NUM_THREADS=1
export OPENBLAS_NUM_THREADS=1
```
BIDS apps that use ANTS, FSL, etc also usually provide a command line-option to run deterministically on one thread. For example, fMRIprep and sMRIprep have a [`--skull-strip-fixed-seed` argument](https://fmriprep.org/en/stable/usage.html#Specific%20options%20for%20ANTs%20registrations) which ensures run-to-run replicability when used with `--omp-nthreads 1` and matching `--random-seed <int>`.

When writing multithreaded R code, reproducibility can be achieved by setting the `RNGkind` to `L'Ecuyer-CMRG`:
```R
RNGkind("L'Ecuyer-CMRG") # Without this line setting a seed won't work in multithreaded code
set.seed(1)
parallel::mclapply(1:2, function(n) rnorm(n), mc.cores = 2)
```
Python, on the other hand, does not need a special kind of RNG:
```py
from multiprocessing.pool import Pool
import numpy as np

np.random.seed(1)
p = Pool(2)
print(list(p.map(np.random.normal, range(2))))
```

## Variance beyond RNGs
Another cause of variance in scientific computing has to do with the limited precision of floating-point numbers. Since humans use a base-10 system, fractions that don't use a prime factor of the base (e.g. 1/3, 1/7, and so on) can't be expressed cleanly; in decimal form, these would be repeating decimals. On the other hand, computers use binary (base-2) and since the only prime factor is 2, you can only cleanly express fractions whose denominator has only 2 as a prime factor. For a computer, `0.1 + 0.2` often doesn't equal `0.3` exactly because when you perform math with re peating decimals you have precision errors. For examples of `0.1 + 0.2` in a variety of languages, see: [0.30000000000000004.com](https://0.30000000000000004.com). Note that these results are dependant on a number of factors, however, including CPU architecture, compiler or underlying system libraries, the use of single or double precision, and multi-threading (via race conditions that change the sequence of floating-point operations). Strategies to mitigate the error such as [compensated summation](https://en.wikipedia.org/wiki/Kahan_summation_algorithm) exist, but at the cost of performance.

## Discussion
Being able to reproduce results exactly is important in science, but as [Brian Avants discusses](https://github.com/ANTsX/ANTsR/issues/210#issuecomment-377511054), forcing run-to-run reproducibility simply "hides the uncertainty intrinsic to the problem." Moreover, if different random numbers cause large differences in the output of your application then there are other issues you should worry about. Furthermore, there is a bias-variance tradeoff. As a method can either be too simple (high bias, low variance) or too complex (low bias, high variance), but not more complex and less complex at the same time, the tradeoff in complexity means there is a give and take between the bias and variance of a particular method. For example, when registering images sampling the grid in a perfectly regular manor results in estimation bias. [Randomness can help reduce this bias](http://bigwww.epfl.ch/preprints/thevenaz0602p.pdf), but at the cost of variance.
