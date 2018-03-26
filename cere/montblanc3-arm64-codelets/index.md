---
title: MontBlanc 3 -- ARMv8 (arm64) standalone codelet repository
---

This is a repository for a set of codelets for ARMv8 extracted from
representative regions and applications from the Montblanc 3 project.

For any question on this repository please contact Pablo Oliveira at University
of Versailles (UVSQ) at pablo DOT oliveira AT uvsq DOT fr.

## Manifest

This repository contains the following codelets

Codelet                                Region                                 Line  Invocation(s) 
-------------------------------------- -------------------------------------- ----  -------------
[lulesh](./lulesh.tar.xz)              CalcFBHourglassForceForElems_Domain    810   16,261
[eikonal](./eikonal.tar.xz)            UpdateScheme:updateScheme_updateEv     204   42
[npb3.0-is](./npb3.0-IS.tar.xz)        is_rank                                475   11
[blackscholes](./blackscholes.tar.xz)  bs_thread                              368   10

## Origin and Licences

The codelets have been extracted from the following applications.
Please refer to each original program for the licence information.

[Lulesh](https://codesign.llnl.gov/lulesh.php) is the Livermore Unstructured Lagrangian Explicit Shock Hydrodynamics.

[Eikonal](http://imsc.uni-graz.at/haasegu/) is an Eikonal solver developed at the University of Graz by Prof. Gundolf Haase.

[NPB3.O-IS](https://www.nas.nasa.gov/publications/npb.html) is an Integer Sort from NASA parallel benchmarks.

[blackscholes](http://parsec.cs.princeton.edu/overview.htm) solves the Blackscholes PDE it belongs to the PARSEC benchmark suite.

## How to use codelets

In the following we will show how to use the codelets. A codelet captures as a standalone application a short representative slice of a large program. It can be used as a proxy for performance analysis, simulation, optimization. Instead of running the whole program, one concentrate in a small region.

We illustrate how to use the codelets with the Blackscholes codelet.

First unpack the archive and go to the working directory,

```bash
$ tar xvf blackscholes.tar.xz
$ cd blackscholes/
```

To measure performance one should use the CERE instrumentation library.
The standard CERE instrumentation library `libcere_instrument.so.0` returns 
in ARMv8 the number of cycles according to the `cntvct_el0` cycle counter.
See below how to replace the library to use your own instrumentation hooks.

First one, should copy the instrumentation library used to the working directory.
To use the standard `cntvct_el0` instrumentation use,

```bash
$ wget https://benchmark-subsetting.github.io/cere/montblanc3-arm64-codelets/libcere_instrument.so.0
```

### Run the original 

Each codelet comes with the original application bundled within, to be used as
a reference. The file `how-run-original.txt` explains how to launch the original
program.

For blackscholes,

```bash
$ ./blackscholes-original <nbthreads> in_64K.txt out.txt
```

After running with 4 threads, for example. We can get the performance of the
region of interest (in blackscholes.m4.cpp starts at line 368):

```bash
$ cat __cere__blackscholes_m4__Z9bs_threadPv_368.csv 
Codelet Name,Call Count,CPU_CLK_UNHALTED_CORE
__cere__blackscholes_m4__Z9bs_threadPv_368,1,1395254
```

This is measured with `mrs %0, cntvct_el0`, where for example 50000000 cycles ~ 1 second in a Juno r1 platform.

### Run the codelet:
One can set the number of threads,

```
$ export OMP_NUM_THREADS=<nbthreads> 4
```

or even select a detailed affinity, 

```
$ export KMP_AFFINITY="verbose,granularity=fine,proclist=[4,5,3,{0,1,2}],explicit" 
```

Then the codelet can be run with,

```
$ ./blackscholes-codelet
$ cat __cere__blackscholes_m4__Z9bs_threadPv_368.csv
Codelet Name,Call Count,CPU_CLK_UNHALTED_CORE
__cere__blackscholes_m4__Z9bs_threadPv_368,1,1377455
```

### Changing the number of repetitions or the warmup mode

The warmup mode and the number of repetitions can be changed using CERE
environment variables. See CERE documentation for more details.

### What if I want to use my own profiling library ?

When running codelets, CERE follows the following sequence:

     warmup-code
     codelet
     warmup-code
     codelet
     ...

It is important that you do not measure the warmup code. For this we provide
some hooks.  In `cere/src/librdtsc/` you will find a small example of how to
use hooks.  You can modify this file by including calls to your own profiling
library.

* `cere_markerStartRegion` is called before entering the codelet regions of interest
* `cere_markerStopRegion` is called after exiting the codelet regions of interest

So for example for `gem5` you should call `m5_resetstats` in `cere_markerStartRegion`
and call `m5_dumpstats` in `cere_markerStopRegion`.

Once you have modified the wrappers, just do make and copy the
`libcere_instrument.so.0` file to your working directory.







