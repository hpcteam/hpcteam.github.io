## Quantum ESPRESSO Installation Guide with Specific Compilation Commands

### Step-by-Step Process for Installing Quantum ESPRESSO (with `make` options)

### Prerequisites

Before starting, make sure the following dependencies are installed:

1. **MPI (Message Passing Interface)** - for parallel execution.
2. **Intel MKL (Math Kernel Library)** - for optimized math routines.
3. **Compilers**:

   * **Intel Fortran (ifort)** or **GFortran (gfortran)**.
   * **Intel C Compiler (icc)** or **GCC (gcc)**.
4. **BLAS** and **LAPACK** libraries for linear algebra operations.

---

### Step 1: Set Up Environment

1. **Source the Intel oneAPI variables** (if using Intel compilers and libraries):

```bash
   source /opt/intel/oneapi/setvars.sh --force
```

2. **Set environment for MPI (OpenMPI or Intel MPI)** as required.

```bash
   export PATH=/path/to/mpi/bin:$PATH
   export LD_LIBRARY_PATH=/path/to/mpi/lib:$LD_LIBRARY_PATH
```

---

### Step 2: Download Quantum ESPRESSO

1. **Extract the downloaded tar file**:

```bash
   tar xvf q-e-qe-7.5.tar.gz
```

2. **Rename the extracted folder** (optional but keeps things organized):

```bash
   mv q-e-qe-7.5 QE
   cd QE
```

---

### Step 3: Configure the Build

Run the `./configure` command to set up the build environment.

* **With MPI, Intel compilers, and Intel MKL libraries**:

  ```bash
  ./configure --prefix=/root/apple/qe \
      CC=mpicc FC=mpif90 F90=mpif90 \
      --enable-parallel \
      LIBDIRS="/opt/intel/oneapi/mkl/2025.3/lib /opt/intel/oneapi/compiler/2025.3/lib /opt/intel/oneapi/mpi/2025.3/lib" \
      LDFLAGS="-L/opt/intel/oneapi/mkl/2025.3/lib -L/opt/intel/oneapi/compiler/2025.3/lib -L/opt/intel/oneapi/mpi/2025.3/lib"
  ```

  **Explanation of the command**:

  * `--prefix=/root/apple/qe`: Install Quantum ESPRESSO in `/root/apple/qe`.
  * `CC=mpicc`: Set the MPI C compiler.
  * `FC=mpif90`: Set the MPI Fortran compiler.
  * `F90=mpif90`: Set the Fortran 90 compiler.
  * `--enable-parallel`: Enable parallel execution.
  * `LIBDIRS` and `LDFLAGS`: Link the necessary Intel libraries, such as Intel MKL, Intel MPI, and Intel compilers.

---

### Step 4: Verifying Compilers and Libraries

Before proceeding with the build, verify that the correct compilers and libraries are being used:

```bash
which mpicc    # Check MPI C compiler
which mpif90   # Check MPI Fortran compiler
which ifort    # Check Intel Fortran compiler (if applicable)
which icc      # Check Intel C compiler (if applicable)
```

Also, confirm the availability of necessary libraries like Intel MKL and MPI:

```bash
ldconfig -p | grep blas
ldconfig -p | grep lapack
```

---

### Step 5: Building Quantum ESPRESSO

After the configuration is complete, use the `make` command to compile Quantum ESPRESSO. You can compile everything or specific components of the software.

#### Option 1: Compile All Components

To build all components of Quantum ESPRESSO using parallel execution:

```bash
make -j $(nproc) all
```

* `-j $(nproc)` utilizes all available CPU cores to speed up the compilation.
* `all` compiles the entire Quantum ESPRESSO suite, including all modules and components.

#### Option 2: Compile Specific Components (e.g., `pw`)

If you want to compile a specific component (e.g., `pw` for Plane-Wave):

```bash
make -j $(nproc) pw
```

This will only compile the `pw.x` binary, which is the core program of Quantum ESPRESSO for plane-wave-based simulations.

#### Option 3: Compile All Components with Additional Options

Sometimes, you might want to compile everything with optimizations or special configurations:

```bash
make -j $(nproc) pwall  # Builds all PW-related components
make -j $(nproc) all    # Compiles all components, including optional ones like TDDFPT, GWW, etc.
```

---

### Step 6: Installation

Once the build process is complete, you can install the compiled binaries by running:

```bash
make install
```

This will install all the compiled binaries into the directory specified by the `--prefix` option (e.g., `/root/apple/qe`).

---

### Step 7: Verifying the Installation

Check that the binaries have been installed correctly by listing the `bin` directory:

```bash
ls /root/apple/qe-gcc/bin/
```

You should see executable binaries like `pw.x`, `ph.x`, `cp.x`, etc.

---

### Step 8: Running Quantum ESPRESSO

You can now run Quantum ESPRESSO simulations using the installed binaries. For example:

```bash
./root/apple/qe-gcc/bin/pw.x -in input_file.in
```

Where `input_file.in` is your input file for the simulation.

---

### Troubleshooting Tips

* **Missing libraries**: Ensure the correct paths for libraries like Intel MKL, MPI, and other dependencies are properly configured.
* **Compiler issues**: If you encounter issues with the compiler, verify the compiler versions and make sure that the correct environment is loaded (`source /opt/intel/oneapi/setvars.sh`).
* **Parallel execution problems**: If parallel execution is not working as expected, check that your MPI installation is correctly configured and test with simpler MPI programs.

---

### Summary of `make` Commands

1. **Build All Components** (using all CPU cores):

```bash
   make -j $(nproc) all
```

2. **Build Specific Components** (e.g., `pw` for plane-wave calculations):

```bash
   make -j $(nproc) pw
```

3. **Build Other Components** (e.g., `pwall` for Plane-Wave calculations or `all` for everything):

```bash
   make -j $(nproc) pwall
   make -j $(nproc) all
```

4. **Install**:

```bash
   make install
```

5. **Run Quantum ESPRESSO**:

```bash
   ./root/apple/qe-gcc/bin/pw.x -in input_file.in
```

---

This guide provides the essential steps and commands to configure, compile, and install Quantum ESPRESSO with parallel execution. It also demonstrates how to build specific components using `make` commands tailored to your needs.

