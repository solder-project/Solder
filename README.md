# Solder

Framework for replacing and modifying functions in external files

## Overview

**Solder** is used for swapping out discrete components within a codebase. Currently it works at the function-level by targeting specific functions within a basic block and removing all references to that function and replacing it with a new target function. 

As this is implemented as a set of LLVM passes, the one requirement is that our target languages can compile to LLVM bitcode. Otherwise, the function swapping is language agnostic and can even be done cross-language. E.g., we can swap out an add function in some C program with another add function that was implemented in Rust. This has the potential for swapping out known-unsafe implementations in C with something memory safe like Rust, or something in an “easily” formally verifiable language. 

## Building and Running Instructions
First, we need to build the set of shared libraries for the LLVM passes:
```
mkdir build && cd build
cmake ..
make -j
```
This will build all the libraries we use in the `build` directory.

**TODO:** include notes about the libstd Rust shared objects.

To run the current version:
```
./scripts/dev-runner.sh
```
This will run the end-to-end test for Solder. Currently, this has the paths and function names all hard-coded. To run against different targets, we comment/uncomment the corresponding lines in that script. There are comments for each line in that script to help understand what's happening. Eventually, however, we will have the `solder.py` executable handle the whole process without needing to switch hard-coded paths.

### Notes for building OpenSSH patches
In the current version, we link our patch to the same `*.so` files that the regular OpenSSH build does. Make sure that all the sources are available in this project and built:

```
git submodule update --init
cd targets/OpenSSH/src
autoreconf
./configure
make sshd -j
```

## Components

### LLVM
The bulk of the work that this system does is implemented as a set of LLVM passes. This makes targeting arbitrary codebases pretty straightforward and modular. Because this is all operating on LLVM IR, we can also use [libFuzzer](http://blog.llvm.org/2015/04/fuzz-all-clangs.html) to run coverage-guided tests on the bitcode that we modify in lieu of using something like AFL.

### KLEE
We use KLEE in this system for targeting potentially-buggy functions/basic blocks in a given codebase. This works towards automating the process of function patching. We also save all the path constraints for functions that contain bugs so we can use them as a specification for synthesis.

### Coq
This is still a work in progress but the idea is to specify some functionality in Coq and generate a function based on it that is inherently verified.
