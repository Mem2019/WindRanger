# Modified Version of WindRanger Fuzzer

The repository contains a modified version of [WindRanger](https://hub.docker.com/r/ardu/windranger). The following changes are made:

1. The instrumentation module is changed from executable to Link Time Optimization (LTO) shared library. This also includes:
	a. Change `clang` version from `10.0.0` to `12.0.0`.
	b. Change SVF version to `2.2` to support LLVM 12.
	c. Remove `${llvm_libs}` in `CMakeLists.txt` according to [this issue](https://github.com/SVF-tools/SVF/issues/53).
	d. Change outputs in `cbi.cpp` from `stdout` to `stderr`.
2. The analysis results stored in files (e.g., `distance.txt`) are saved in a newly created temporary directory instead of simply current directory. This includes:
	a. In `afl-clang-fast.c`, create such directory and pass its path into the WindRanger pass.
	b. In `cbi.cpp`, store all generated files into this directory and save the path of such directory into generated executable.
	c. In `afl-fuzz.c`, get the directory path from executable and read the contents of files.

## Usage

```bash
# LLVM 12 must be used for LTO
sudo apt-get install clang-12

# Environment variables
export CC=clang-12
export CXX=clang++-12
export LLVM_CONFIG=llvm-config-12
export AFL_REAL_LD=ld.lld-12

# Compile WindRanger
cd WindRanger && make clean all && cd llvm_mode && make clean all

# Set AFL compilers
export CC="/path/to/WindRanger/afl-clang-fast"
export CXX="/path/to/WindRanger/afl-clang-fast++"

# Make sure all compilation tools are using the LLVM 12 tool chain
export AS="$(which llvm-as-12)"
export RANLIB="$(which llvm-ranlib-12)"
export AR="$(which llvm-ar-12)"
export LD="$(which ld.lld-12)"
export NM="$(which llvm-nm-12)"
export AFL_CC=clang-12
export AFL_CXX=clang++-12

# Set the targets, the file BBtargets.txt has the same format as that in AFLGo
export WR_BB_TARGETS="/path/to/BBtargets.txt"
# Set the programs used for directed fuzzing, separated by ':'
export WR_TARGETS="prog1:prog2" # or "::" for all programs

# Use WindRanger to compile the target program
$CC prog1.c -o prog1

# Run the directed fuzzer
WindRanger/afl-fuzz -d -m none -i ./in -o ./out -- ./prog1 @@
```
