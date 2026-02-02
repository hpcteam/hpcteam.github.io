# OpenMPI Theory and HPC Implications

**OpenMPI** is an **open-source implementation of MPI (Message Passing Interface)** — a standard used to write programs that run **in parallel across multiple computers or CPU cores**.



👉 **OpenMPI lets many processes talk to each other efficiently**, even if they’re running on different machines in a cluster.

### What it’s used for

* High-performance computing (HPC)
* Scientific simulations (physics, chemistry, climate models)
* Machine learning & data processing at scale
* Any program that needs to run **faster by splitting work across processors**

### How it works (conceptually)

* You write one program
* OpenMPI runs **multiple copies** of it
* Each copy (called a *process*) communicates by sending/receiving messages

Example idea:

* Process 0 handles part A
* Process 1 handles part B
* They exchange results via MPI messages

### Key features

* Runs on laptops, servers, or large clusters
* Supports C, C++, Fortran, Python (via bindings like `mpi4py`)
* Very fast and scalable
* Works over Ethernet, InfiniBand, shared memory, etc.


##  Installing Compilers from Source code 

### Step 1: Download Source Code

Download from official website (gcc, llvm, openmpi, etc.)

---

### Step 2: Extract the Source

```
tar -xvf package.tar.gz
cd package
```

---

### Step 3: Check Extracted Files

Look for:

```
configure
configure.ac
autogen.sh
```

---

### Step 4: Generate Configure Script (if missing)

If `configure` is not present:

```
./autogen.sh
```

---

### Step 5: Configure the Build

```
./configure --prefix=/opt/openmpi/5.0.1
```

---

### Step 6: Build the Software

```
make -j <number_of_cores>
```

Example:

```
make -j 16
```

---

### Step 7: Run Checks (Optional)

```
make check
```

---

### Step 8: Install

```
make install
```

This is the final step.

---

## 6. Post-Installation

After installation completes:

1. Create a module file for the compiler
2. Add correct PATH and LD_LIBRARY_PATH
3. Test compiler and MPI

---

## 7. Final Verification

```
module load compiler/version
module load openmpi/5.0.1
mpicc --version
mpirun -np 2 hostname
```

---

## Notes (Best Practices)

* Always install under `/home/software` or `/opt/software`
* Always use module files for HPC environments
* Keep one version per directory
* Document prefix paths
* Test with small MPI jobs

---

**Document prepared for HPC / Linux System Administration use**
