# HPCC 1.5.0 Installation and Build Guide (Intel Linux)

This document guides you through downloading, building, and running HPCC 1.5.0 using Intel oneAPI compilers on a Linux system.

---

## 1. Download the HPCC Source Code

1. Download HPCC 1.5.0 from Netlib:

```bash
wget http://www.netlib.org/benchmark/hpcc/hpcc-1.5.0.tar.gz
```

2. Extract the archive:

```bash
tar -xzvf hpcc-1.5.0.tar.gz
cd hpcc-1.5.0
```

* The extracted source contains several directories including: `hpl/`, `DGEMM/`, `STREAM/`, `FFT/`, `PTRANS/`, `RandomAccess/`.

---

## 2. Go to the HPL Directory

1. Enter the HPL source folder:

```bash
cd hpl
```

* This is where the **High-Performance Linpack (HPL)** library is built.

---

## 3. Prepare the Makefile

1. Copy the template Makefile for your system:

```bash
cp setup/Make.LinuxIntel Make.LinuxIntel
```

2. Open `Make.LinuxIntel` in an editor and make the following necessary changes:

* **Compiler settings**:

```makefile
CC = mpicc
LINKER = mpicc
```

* **Compiler flags** (example for Intel CPUs):

```makefile
CFLAGS = -O3 -xHost -I$(MKL_INCLUDE_PATH)
LDFLAGS = -L$(MKL_LIB_PATH) -lmkl_intel_lp64 -lmkl_sequential -lmkl_core -lpthread -lm
```

* **Libraries**:

```makefile
LIBS = -lmkl_intel_lp64 -lmkl_sequential -lmkl_core
```
we need to mention PATH before the flages 

* **MPI include/library paths**:

```makefile
MPI_INC = /opt/intel/oneapi/mpi/latest/include
MPI_LIB = /opt/intel/oneapi/mpi/latest/lib
```

* Make sure the architecture name matches your chosen name (here, `LinuxIntel`):

```makefile
ARCH = LinuxIntel
```

> ⚠️ Note: Paths like `MKL_INCLUDE_PATH` and `MKL_LIB_PATH` should point to your Intel MKL installation, typically `/opt/intel/oneapi/mkl/latest/include` and `/opt/intel/oneapi/mkl/latest/lib/intel64`.

---

## 4. Source Intel oneAPI Environment

Before building, source the oneAPI environment to make `mpicc` and MKL libraries available:

```bash
source /opt/intel/oneapi/setvars.sh --force
```

---

## 5. Build HPL Library

From the `hpl` directory:

```bash
make arch=LinuxIntel
```

* **Notes**:

  * Replace `LinuxIntel` with the architecture name specified in your Makefile.
  * Warnings like `mkdir: cannot create directory … File exists` can be ignored.
  * Ensure no errors like `mpicc: command not found`. If this occurs, check step 4.

* Successful build produces **static library**:

```bash
ls lib/LinuxIntel/
# libhpl.a
```

---

## 6. Build the HPCC Executable

1. Return to the top-level HPCC directory:

```bash
cd ..
```

2. Build HPCC using the same architecture:

```bash
make arch=LinuxIntel
```

* **Notes**:

  * This compiles all modules including HPL, RandomAccess, STREAM, PTRANS, FFT, and DGEMM.
  * Output executable:

```bash
ls
# hpcc
```

---

## 7. Create HPCC Input File

1. Create `_hpccinf.txt` or `hpcc.dat` to configure test parameters. Example:

```text
1          # number of times to run each test
5000 5000  # problem size N and block size NB for HPL
1 1        # P Q process grid
```

---

## 8. Run HPCC

Use MPI to execute:

```bash
mpirun -np 4 ./hpcc
```

* Adjust `-np 4` based on available cores.
* Results are written to `_hpccinf.txt` in the same directory.

---

## 9. Verify Output

```bash
cat _hpccinf.txt
```

* Sections include performance results for:

  * HPL (Linpack)
  * RandomAccess
  * STREAM
  * PTRANS (Parallel Matrix Transpose)
  * FFT
  * DGEMM

---

## 10. Common Issues and Solutions

| Issue                                    | Solution                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------ |
| `mpicc: command not found`               | Source Intel oneAPI environment: `source /opt/intel/oneapi/setvars.sh --force` |
| `mkdir: cannot create directory`         | Harmless; Makefile ignores it                                                  |
| Linking errors with MKL                  | Verify MKL library paths in `Make.LinuxIntel`                                  |
| Indentation warnings in HPL              | Safe to ignore; does not affect compilation                                    |
| Symbolic link errors (`Make.inc` exists) | Remove existing link and retry: `rm src/auxil/LinuxIntel/Make.inc`             |

---

## 11. Optional: Clean Build

```bash
make arch=LinuxIntel clean
make arch=LinuxIntel
```

* Rebuilds all components from scratch.

