# HPL Benchmark Installation and Setup Using Intel MKL (Step-by-Step Guide)

This guide explains how to **install, compile, and run the High Performance Linpack (HPL) benchmark** on a Linux system using **Intel oneAPI MKL and MPI**.

HPL is commonly used to measure **supercomputer performance in FLOPS** and is the benchmark used in the **TOP500 supercomputer rankings**.

---

# 1. System Requirements

Before starting, ensure your system has:

* Linux (CentOS / RHEL / Ubuntu)
* GCC compiler
* Intel oneAPI toolkit
* MPI (Intel MPI or OpenMPI)
* Make utility
* At least **8GB RAM (recommended 32GB+)**

Check system resources:

```bash
nproc
free -h
lscpu
```

Example output:

```
CPU cores : 48
RAM       : 124GB
```

---

# 2. Install Intel oneAPI

Download and install **Intel oneAPI Base Toolkit which includes **Intel Math Kernel Library (MKL).

After installation, initialize the environment:

```bash
source /opt/intel/oneapi/setvars.sh
```

Verify MKL installation:

```bash
ls /opt/intel/oneapi/mkl/latest/lib/intel64
```

---

# 3. Install Required Development Tools

Install build tools:

For **RHEL / CentOS**

```bash
yum groupinstall "Development Tools"
yum install wget make gcc gcc-c++
```

For **Ubuntu**

```bash
sudo apt install build-essential wget
```

---

# 4. Download HPL Benchmark

Download **High Performance Linpack:

```bash
wget https://www.netlib.org/benchmark/hpl/hpl-2.3.tar.gz
```

Extract the package:

```bash
tar -xvf hpl-2.3.tar.gz
cd hpl-2.3
```

Directory structure:

```
hpl-2.3/
 ├── bin
 ├── include
 ├── lib
 ├── setup
 ├── src
 └── testing
```

---

# 5. Configure the Makefile

Navigate to the setup directory:

```bash
cd setup
```

Copy the example configuration:

```bash
cp Make.Linux_Intel64 ../Make.Linux_Intel64
```

Go back to the HPL root directory:

```bash
cd ..
```

Edit the configuration file:

```bash
vi Make.Linux_Intel64
```

---

# 6. Configure MKL Libraries

Set MKL directory:

```make
LAdir = /opt/intel/oneapi/mkl/latest
```

Add include path:

```make
LAinc = -I$(LAdir)/include
```

Add MKL libraries:

```make
LAlib = -L$(LAdir)/lib/intel64 \
        -Wl,--start-group \
        $(LAdir)/lib/intel64/libmkl_intel_lp64.a \
        $(LAdir)/lib/intel64/libmkl_intel_thread.a \
        $(LAdir)/lib/intel64/libmkl_core.a \
        -Wl,--end-group \
        -lpthread -lm -ldl -liomp5
```

Enable OpenMP:

```make
OMP_DEFS = -fopenmp
```

---

# 7. Compile HPL

Load Intel environment:

```bash
source /opt/intel/oneapi/setvars.sh
```

Build HPL:

```bash
make arch=Linux_Intel64
```

After compilation completes successfully:

```
bin/Linux_Intel64/xhpl
```

will be created.

Verify:

```bash
ls bin/Linux_Intel64
```

Output:

```
HPL.dat
xhpl
```

---

# 8. Configure HPL Benchmark Input

Edit the configuration file:

```bash
cd bin/Linux_Intel64
vi HPL.dat
```

Example configuration:

```
HPLinpack benchmark input file
HPL.out
6
1
90000
1
256
0
1
6
6
16.0
1
1
1
4
1
2
1
1
1
1
0
1
0
0
1
8
```

Key parameters:

| Parameter | Meaning      |
| --------- | ------------ |
| N         | Matrix size  |
| NB        | Block size   |
| P x Q     | Process grid |

---

# 9. Run the Benchmark

HPL must be executed using **MPI**, not directly.

Example:

```bash
mpirun -np 4 ./xhpl
```

Run using more cores:

```bash
mpirun -np 16 ./xhpl
```

or

```bash
mpirun -np 48 ./xhpl
```

Where:

```
-np = number of MPI processes
```

---

# 10. Sample Output

Example benchmark output:

```
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR00R2R4        90000   256     6     6            120.45              2100.34
```

Explanation:

| Column | Description    |
| ------ | -------------- |
| N      | Matrix size    |
| NB     | Block size     |
| P,Q    | MPI grid       |
| Time   | Execution time |
| Gflops | Performance    |

---

# 11. Performance Expectations

For a system with:

* 48 CPU cores
* Intel MKL
* 120+ GB RAM

Expected performance:

```
1.5 – 2.5 TFLOPS
```

---
# HPL Benchmark: Common Errors and Their Solutions

When compiling and running **High Performance Linpack (HPL)** with **Intel MKL + MPI**, beginners often encounter errors. This guide explains the errors you faced and provides solutions.

---

## 1. Compilation Errors with GCC

### Error Example:

```
gcc: error: unrecognized command-line option ‘-ansi-alias’
gcc: error: unrecognized command-line option ‘-i-static’; did you mean ‘--static’?
gcc: error: unrecognized command-line option ‘-nocompchk’
```

### Cause:

The HPL Makefile was **configured for the Intel compiler**, but you used **GCC** instead. GCC does not recognize Intel-specific flags like `-ansi-alias`, `-i-static`, and `-nocompchk`.

### Solution:

1. **Use Intel Compiler** (`icc` / `icpc`) instead of GCC:

```bash
source /opt/intel/oneapi/setvars.sh
make arch=Linux_Intel64 CC=icc
```
2. Or **modify the Makefile** to remove Intel-only flags if using GCC:

```make
# Remove or comment these lines in Make.Linux_Intel64
# -ansi-alias -i-static -nocompchk
```

---

## 2. Linking Errors with MKL OpenMP

### Error Example:

```
undefined reference to `__kmpc_global_thread_num'
undefined reference to `omp_get_thread_num'
undefined reference to `sqrt'
```

### Cause:

These errors occur because the linker **cannot find the OpenMP runtime or math library**. When using **libmkl_intel_thread**, you must also link:

* OpenMP runtime (`-liomp5`)
* Standard math library (`-lm`)

### Solution:

Update the MKL library link line in your Makefile:

```make
LAlib = -L$(MKLROOT)/lib/intel64 \
        -Wl,--start-group \
        $(MKLROOT)/lib/intel64/libmkl_intel_lp64.a \
        $(MKLROOT)/lib/intel64/libmkl_intel_thread.a \
        $(MKLROOT)/lib/intel64/libmkl_core.a \
        -Wl,--end-group \
        -lpthread -lm -ldl -liomp5
```

Ensure `-fopenmp` is enabled in compilation flags:

```make
CFLAGS = -O3 -fopenmp
```

---

## 3. HPL Execution Error: Not Enough Processes

### Error Example:

```
HPL ERROR from process # 0, on line 419 of function HPL_pdinfo:
>>> Need at least 4 processes for these tests <<<
```

### Cause:

HPL requires at least as many **MPI processes** as specified in your **process grid (P x Q)** in `HPL.dat`.

Your `HPL.dat` configuration had:

```
3            # of process grids (P x Q)
2 1 4        Ps
2 4 1        Qs
```

This combination requires **at least 4 processes**, but you may have run with `-np 1`.

### Solution:

Run with sufficient MPI processes:

```bash
mpirun -np 4 ./xhpl
```

Or adjust the **process grid (P x Q)** to match the number of processes you want to use.

---

## 4. HPL Execution Error: Illegal Input in HPL.dat

### Error Example:

```
HPL ERROR from process # 0, on line 621 of function HPL_pdinfo:
>>> Illegal input in file HPL.dat. Exiting ...
```

### Cause:

This occurs when the **parameters in HPL.dat are inconsistent**. Common issues:

* `NB` (block size) larger than `N` (matrix size)
* `P x Q` process grid does not divide the number of MPI processes
* Negative or zero values in configuration fields

### Solution:

* Ensure `N` (matrix size) > 0
* `NB` (block size) ≤ `N`
* `P x Q` ≤ number of MPI processes (`-np`)
* Check that all numeric fields are valid integers

Example working snippet:

```
4            # number of problems
256 512 1024 2048  # problem sizes (N)
64 128 256 512     # block sizes (NB)
2 2                # process grid (P x Q)
```

---

## 5. Build Error: Missing libhpl.a

### Error Example:

```
No rule to make target '/root/apple/hpl-2.3/lib/Linux_Intel64/libhpl.a', needed by 'dexe.grd'. Stop.
```

### Cause:

* `libhpl.a` (HPL library) **was not built** before building the tests.
* Often caused by **cleaning directories without rebuilding src**.

### Solution:

1. Rebuild HPL library:

```bash
make arch=Linux_Intel64 build_src
```

2. Then build the tests:

```bash
make arch=Linux_Intel64 build_tst
```

3. Confirm `libhpl.a` exists:

```bash
ls lib/Linux_Intel64/libhpl.a
```

---

## 6. General Make Clean Error

### Error Example:

```
Make.top:49: Make.UNKNOWN: No such file or directory
make: *** [Makefile:76: clean] Error 2
```

### Cause:

`make clean` without specifying the architecture fails if the Makefile **cannot detect your architecture**.

### Solution:

Always specify the architecture:

```bash
make arch=Linux_Intel64 clean
```

Or delete the build directories manually:

```bash
rm -rf bin/Linux_Intel64 lib/Linux_Intel64
```

---

## 7. Summary of Best Practices

1. **Always load Intel environment** before building:

```bash
source /opt/intel/oneapi/setvars.sh
```

2. **Use Intel compilers (icc, ifort) with MKL**
   Avoid GCC unless you modify flags.

3. **Follow architecture-specific Makefile**:

```bash
make arch=Linux_Intel64
```

4. **Check HPL.dat for consistency** before running HPL.

5. **Use enough MPI processes**:

```
np >= P * Q
```

6. **Rebuild HPL library** if tests fail due to missing `libhpl.a`.

---

This document can serve as a **ready troubleshooting guide** for your website readers who are installing and running HPL.

### HPL Build & Run Flowchart with Errors and Solutions


Start
  │
  ▼
[Step 1: Setup Environment]
  │
  │  Commands:
  │    source /opt/intel/oneapi/setvars.sh
  │
  ▼
[Step 2: Prepare Makefile]
  │
  │  Action:
  │    Copy architecture-specific Make.Linux_Intel64 to src/ and testing/
  │
  ▼
[Step 3: Build HPL Library (libhpl.a)]
  │
  │  Command:
  │    make arch=Linux_Intel64 build_src
  │
  │  Possible Error:
  │    - gcc: unrecognized command-line option ‘-ansi-alias’
  │    Solution:
  │      → Use Intel compiler (icc/ifort) or remove Intel-specific flags for GCC
  │
  ▼
[Step 4: Build HPL Tests]
  │
  │  Command:
  │    make arch=Linux_Intel64 build_tst
  │
  │  Possible Errors:
  │    1) Missing libhpl.a
  │       Solution: Rebuild library (Step 3)
  │    2) Linking errors (undefined references to OpenMP or math functions)
  │       Solution: Ensure flags -fopenmp, -liomp5, -lm are included
  │
  ▼
[Step 5: Verify bin/ and lib/]
  │
  │  Expected files:
  │    bin/Linux_Intel64/xhpl
  │    lib/Linux_Intel64/libhpl.a
  │
  ▼
[Step 6: Prepare HPL.dat]
  │
  │  Common Errors:
  │    - Illegal input in HPL.dat
  │      Solution: Ensure N, NB, P x Q, and other fields are valid
  │    - Process grid mismatch
  │      Solution: Ensure MPI processes ≥ P * Q
  │
  ▼
[Step 7: Run HPL Benchmark]
  │
  │  Command example:
  │    mpirun -np 4 ./xhpl
  │
  │  Possible Errors:
  │    - Need at least X processes
  │      Solution: Increase number of MPI processes or adjust process grid
  │
  ▼
[Step 8: Check Output]
  │
  │  Output: HPL.out or console stdout
  │
  ▼
End






















