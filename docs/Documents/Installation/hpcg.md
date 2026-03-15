# **Step-by-Step Guide: Building and Running HPCG 3.1**

### **Step 0: Prerequisites**

Make sure your system has:

* **Linux OS**
* **Intel oneAPI toolkit installed** (for `icpc` compiler)
* **MPI installed** (comes with Intel oneAPI or system MPI)
* Basic commands: `tar`, `make`, `cp`, `ls`, `cd`

---

### **Step 1: Move the HPCG archive to your working directory**

```bash
mv /home/Downloads/hpcg-3.1.tar.gz .
```

* This moves `hpcg-3.1.tar.gz` to the current directory (here, `/root/apple`).

---

### **Step 2: Extract the archive**

```bash
tar xvf hpcg-3.1.tar.gz
```

* This creates a folder called `hpcg-3.1` with `src`, `setup`, `bin`, and documentation.
* Check contents:

```bash
cd hpcg-3.1
ls
```

You should see folders like: `src`, `setup`, `bin`, files like `README.md`, `Makefile`.

---

### **Step 3: Choose a Makefile configuration**

HPCG has multiple Makefile templates for different compilers and parallel options:

```bash
ls setup/
```

Some examples:

* `Make.GCC_OMP` → GCC compiler with OpenMP
* `Make.ICPC_OMP` → Intel compiler with OpenMP
* `Make.Linux_MPI` → Linux system with MPI
* `Make.Mac_Serial` → Mac, serial build

* Copies the template to the working directory for modification if needed.
* Optional: edit `Make.Linux_MPI` with `vi` if you want to change compiler flags.

---

* **Copy the template to the working directory for modification if needed**

```bash
  cp setup/Make.Linux_MPI Make.Linux_MPI
```
* **Optional:** Edit `Make.inc` with `vi` if you want to change compiler flags.

```bash
 vi Make.Linux_MPI
```
* **Add required paths and libraries:**
  Make sure to set **MPI, MKL, and other Intel oneAPI paths** in `Make.inc` so the compiler and linker can find headers and libraries. Example:

```make
  CXX = mpicxx
  CXXFLAGS = -O3 -xHost -qopenmp -mavx
  INCLUDES = -I$(MKLROOT)/include -I./src
  LIBS = -L$(MKLROOT)/lib/intel64 -lmkl_intel_lp64 -lmkl_core -lmkl_sequential -lpthread -lm -ldl
```


---

### **Step 4: Build HPCG**

Use `make` with the `arch` parameter:

```bash
make arch=Linux_MPI
```

* This tells `Makefile` which configuration to use.
* The compiler `mpicxx` will compile source files in `src/`.
* Output: `bin/xhpcg` executable.

> ⚠️ If you get an error like `icpc: No such file`, make sure you run:

```bash
source /opt/intel/oneapi/setvars.sh --force
```

This sets the Intel compiler environment.

---

### **Step 5: Verify the executable**

```bash
ls bin/
```

You should see:

```
xhpcg  hpcg.dat
```

* `xhpcg` → the compiled HPCG program
* `hpcg.dat` → input file with problem parameters

---

### **Step 6: Check input file**

```bash
cat bin/hpcg.dat
```

Example content:

```
HPCG benchmark input file
Sandia National Laboratories; University of Tennessee, Knoxville
104 104 104   # Grid size: X, Y, Z
60            # Number of iterations
```

* You can adjust grid size if your system has limited memory.

---

### **Step 7: Run HPCG with MPI**

From the `bin/` directory:

```bash
cd bin
mpirun -np 4 ./xhpcg
```

* `-np 4` → run with 4 MPI processes
* If your machine is small, use `-np 2` or `-np 1`.
* HPCG will run and produce output in the terminal.

> ⚠️ Common issues:
>
> * `BAD TERMINATION` → too many MPI ranks for your system
> * Hangs → memory too small or MPI mismatch

---

### **Step 8: Debugging Tips**

1. Reduce **grid size** in `hpcg.dat` if it fails:

```
32 32 32
10
```

2. Test **serial build** to verify compilation:

```bash
make arch=Linux_Serial   # or Mac_Serial
./xhpcg
```

3. Make sure MPI runtime matches the compiler:

```bash
which mpicxx
which mpirun
```

---

### **Step 9: Clean Build (optional)**

To remove compiled objects:

```bash
make clean
```

* Use this before changing Makefile or compiler settings.

---

### Running HPCG and Understanding the Output

### Step 1: Run the benchmark

```bash
mpirun -np 4 ./xhpcg
```

* `-np 4` → Runs HPCG on **4 MPI processes**.
* `xhpcg` → HPCG executable reads the input file `hpcg.dat`.

---

### Step 2: Output files generated

After the run, HPCG produces:

| File                   | Description                                                                  |
| ---------------------- | ---------------------------------------------------------------------------- |
| `hpcg*.txt`            | Quick output showing iterations and scaled residuals.                        |
| `HPCG-Benchmark_*.txt` | Full benchmark report including memory, performance, and validation results. |

---

### Step 3: Key sections in the full report

#### Machine & MPI Info

* Confirms number of MPI processes and threads:

```
Distributed Processes=4
Threads per processes=1
Processor Dimensions: npx=2, npy=2, npz=1
```

* Shows how the 3D problem is split among MPI ranks.

#### Problem & Memory Info

* Problem size and local domain:

```
Global nx=208, ny=208, nz=104
Local nx=104, ny=104, nz=104
Memory used ~ 3.2 GB
```

* Total equations: 4,499,456
* Non-zero entries: 119,934,040

#### Validation (V&V) Tests

```
Spectral Convergence Tests::Result=PASSED
Departure from Symmetry::Result=PASSED
Iteration Count::Result=PASSED
Reproducibility::Result=PASSED
```

* All tests **PASSED**, confirming the computation is correct.

#### Performance Summary

* Execution time and floating point performance:

```
Benchmark Time::Total=69.7925 sec
GFLOP/s::Total=5.88
```

* Confirms the benchmark **ran fully**.

#### Final Result

```
HPCG result is VALID with a GFLOP/s rating of 5.88173
```

* This is the official output, confirming the run was successful.

---

### Step 4: How to know HPCG actually ran

1. Output files are generated (`HPCG-Benchmark_*.txt`).
2. GFLOPS and runtime values are populated.
3. Validation tests (PASSED) indicate computations were performed correctly.

⚠️ Note: Warnings about “unpreconditioned iterations” are normal and do **not** indicate failure.




