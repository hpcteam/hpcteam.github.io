---
# SU2 Installation and Application Overview

## 1. Introduction and Application Theory

**SU2** is an open-source software suite designed primarily for **computational fluid dynamics (CFD)** and **multiphysics simulations**. It is widely used in academia and industry for **aerodynamic analysis, design optimization, and uncertainty quantification**.

### 1.1 Governing Equations

SU2 solves a variety of partial differential equations, including:

- Euler equations (inviscid flow)
- Navier–Stokes equations (viscous flow)
- Reynolds-Averaged Navier–Stokes (RANS)
- Compressible and incompressible flow equations

These equations are solved using the **finite volume method (FVM)** on structured and unstructured meshes.

### 1.2 Numerical Methods

Key numerical features include:

- Upwind and central discretization schemes
- Explicit and implicit time integration
- Multigrid acceleration
- Turbulence models (Spalart–Allmaras, k–ω, SST)
- Adjoint-based methods for **gradient computation**

### 1.3 Design Optimization and Adjoint Method

One of SU2’s core strengths is its **discrete adjoint solver**, which enables:

- Efficient gradient computation
- Shape and topology optimization
- Sensitivity analysis with respect to design variables

This makes SU2 particularly suitable for **aerodynamic shape optimization** problems where traditional finite-difference methods would be computationally expensive.

## 2. SU2 Installation Guide (Version 8.0.0)

### Step 1: Download the source code

Download the SU2 source code from the official Git repository.

````
Example:
wget https://github.com/su2code/SU2/archive/refs/tags/v8.0.0.tar.gz -O SU2-8.0.0.tar.gz
````
---

### Step 2: Extract the source files

```bash
tar zxf SU2-8.0.0.tar.gz
ls
cd SU2-8.0.0/
ls
```

---

### Step 3: Minimal requirements

Ensure the following tools are installed:

* C/C++ compiler (GCC, G++, or Clang)
* Python 3 (version 3.6 or later)

**Note:**
All additional build tools and dependencies are either included in the source tree or automatically downloaded during the configuration process.

Optional but recommended:

* MPI library (OpenMPI or MPICH) for parallel simulations
* CMake

---

### Step 4: Configure the build using Meson

SU2 uses the Meson build system. The configuration script `meson.py` is located in the root source directory.

Example configuration:

```bash
./meson.py build -Denable-autodiff=true --prefix=/home/username/SU2
```

Optional configuration flags:

* Enable unit tests:

  ```bash
  -Denable-tests=true
  ```
* Enable MPI:

  ```bash
  -Denable-mpi=true
  ```

This step creates the `build/` directory and prepares SU2 for compilation.

---

### Step 5: Compile and install SU2

```bash
./ninja -C build install
```

After installation, the SU2 executables and Python modules will be located in:

```
/home/username/SU2/
```

---

### Step 6: Set environment variables

To run SU2 from any directory, add the following lines to your `~/.bashrc` or `~/.zshrc` file:

```bash
export SU2_HOME=/home/username/SU2
export PATH=$SU2_HOME/bin:$PATH
export PYTHONPATH=$SU2_HOME:$PYTHONPATH
```

Apply the changes:

```bash
source ~/.bashrc
```

Verify the installation:

```bash
which SU2_CFD
SU2_CFD --help
```

---

### Step 7: Run sample tutorial cases

The tutorial cases are located in:

```bash
SU2-8.0.0/Tutorials/
```

Example test case:

```bash
cd Tutorials/incompressible_flow/channel
SU2_CFD channel.cfg
```

---

## 3. Notes and Troubleshooting

* Ensure Python 3 is used:

  ```bash
  python3 --version
  ```
* For parallel execution:

  ```bash
  mpirun -np 4 SU2_CFD config.cfg
  ```
* If executables are not found, recheck the `PATH` variable.

---

## 4. References

* SU2 Official Website: [https://su2code.github.io](https://su2code.github.io)
* SU2 GitHub Repository: [https://github.com/su2code/SU2](https://github.com/su2code/SU2)

```

---

