## LAMMPS Installation For compute-nodes

This guide explains how to install **LAMMPS** on **Red Hat Enterprise Linux (RHEL)** for an HPC cluster environment with **MPI and OpenMP** support. Each step and flag is explained so administrators understand *why it is required*.

---

## 1. Create Cluster Software Directory

Most HPC systems install applications in a **shared filesystem** so all compute nodes can access the same binaries.

Typical locations:

- `/apps`
- `/opt/apps`
- `/shared/software`

Command:

```bash
mkdir -p /apps/lammps
cd /apps/lammps
```

### Why this step is required

| Command | Purpose |
|------|------|
| `mkdir -p` | Creates the directory structure for software installation |
| `/apps/lammps` | Central location so login nodes and compute nodes can access LAMMPS |


This structure also allows **multiple LAMMPS versions** to coexist.

Example:

```
/apps/lammps/2024
/apps/lammps/2025
```

---

## 2. Extract LAMMPS Source

```bash
tar -xvf lammps-22Jul2025.tar.gz
cd lammps-22Jul2025
```

### Why this step is required

| Command | Purpose |
|------|------|
| `tar -xvf` | Extracts the compressed source code archive |

After extraction, the directory contains:

```
bench/      Benchmark inputs
cmake/      Build configuration files
examples/   Example simulations
lib/        External libraries
src/        Core LAMMPS source code
```

---

## 3. Create Build Directory

```bash
mkdir build
cd build
```

### Why this step is required

LAMMPS uses **CMake out-of-source builds**.

Advantages:

- Keeps source code clean
- Allows multiple build configurations
- Easier upgrades and rebuilds

Example:

```
lammps-22Jul2025/
 ├── src
 ├── cmake
 └── build
```

The `build` directory will store:

- object files
- compiled binaries
- temporary build configuration

---

## 4. Configure Build with CMake

```bash
cmake ../cmake \
-D CMAKE_INSTALL_PREFIX=/apps/lammps/22Jul2025 \
-D BUILD_MPI=on \
-D BUILD_OMP=on \
-D PKG_MOLECULE=on \
-D PKG_KSPACE=on \
-D PKG_MANYBODY=on \
-D PKG_EXTRA-COMPUTE=on \
-D PKG_EXTRA-FIX=on \
-D PKG_EXTRA-PAIR=on
```

### What CMake Does

CMake prepares the **build configuration** by:

- Detecting compilers
- Checking system libraries
- Enabling requested packages
- Generating Makefiles

---

# Explanation of Each Flag

## `-D CMAKE_INSTALL_PREFIX=/apps/lammps/22Jul2025`

Defines where the compiled software will be installed.

Without this flag, CMake installs to:

```
/usr/local
```

Cluster administrators prefer custom paths so they can manage versions.

Resulting structure:

```
/apps/lammps/22Jul2025
 ├── bin
 ├── lib
 └── share
```

---

## `-D BUILD_MPI=on`

Enables **MPI parallel execution**.

MPI allows simulations to run across multiple nodes.

Example:

```
mpirun -np 64 lmp -in input.in
```

Without MPI:

- LAMMPS runs on **one CPU only**
- Cannot scale to cluster workloads

MPI is essential for:

- large atom simulations
- multi-node scaling
- production HPC workloads

---

## `-D BUILD_OMP=on`

Enables **OpenMP threading**.

OpenMP allows each MPI process to use **multiple CPU cores**.

Example configuration:

```
MPI ranks: 8
OpenMP threads per rank: 8
Total cores used: 64
```

Benefits:

- better CPU utilization
- improved memory sharing
- faster performance on multi-core nodes

---

## `-D PKG_MOLECULE=on`

Enables **molecular topology features**.

Required for simulations containing:

- bonds
- angles
- dihedrals

Without this package:

LAMMPS cannot simulate:

- polymers
- biomolecules
- complex molecular systems

---

## `-D PKG_KSPACE=on`

Enables **long-range electrostatic solvers**.

Important for simulations involving:

- charged particles
- ionic systems
- biomolecules

Algorithms included:

- PPPM
- Ewald

These methods compute electrostatic forces efficiently.

---

## `-D PKG_MANYBODY=on`

Adds **many-body potential models**.

Examples:

- Tersoff
- Stillinger–Weber
- Embedded Atom Method

Used in simulations of:

- semiconductors
- metals
- materials science

---

## `-D PKG_EXTRA-COMPUTE=on`

Adds additional **analysis calculations**.

Examples:

- stress tensors
- structural analysis
- advanced diagnostics

Useful for research workloads.

---

## `-D PKG_EXTRA-FIX=on`

Adds extra **simulation control features**.

"Fix" commands control system behavior such as:

- thermostats
- constraints
- time integration

Extra fixes expand simulation capabilities.

---

## `-D PKG_EXTRA-PAIR=on`

Adds additional **pair interaction potentials**.

These define how atoms interact.

Examples include:

- Lennard-Jones variants
- Buckingham potentials

More pair styles allow broader simulation types.

---

## 5. Compile LAMMPS

```bash
make -j
```

### Explanation

| Component | Purpose |
|------|------|
| `make` | Compiles the source code |
| `-j` | Enables parallel compilation |

Parallel compilation significantly reduces build time.

Example:

```
64-core node → compilation uses 64 threads
```

---

## 6. Install LAMMPS

```bash
make install
```

### Why installation is needed

Compilation creates binaries in the **build directory**, but installation moves them to the final location.

Installed files include:

```
/apps/lammps/22Jul2025/bin/lmp
/apps/lammps/22Jul2025/share/lammps
```

This makes the program available to cluster users.

---

## 7. Add LAMMPS to PATH

```bash
export PATH=/apps/lammps/22Jul2025/bin:$PATH
```

### Why this is needed

PATH tells the shell where to find executables.

Without modifying PATH, users must run:

```
/apps/lammps/22Jul2025/bin/lmp
```

After updating PATH, they can simply run:

```
lmp
```

---

## 8. Configure Environment Script

LAMMPS provides an environment script that helps users automatically load required variables such as `PATH` and `LD_LIBRARY_PATH`.

To activate it for the current shell:

```bash
source /apps/lammps/22Jul2025/etc/profile.d/lammps.sh
```

### Why this step is useful

This script ensures users can run the `lmp` command without specifying the full path.

Without the script:

```
/apps/lammps/22Jul2025/bin/lmp
```

With the script:

```
lmp
```

### What the script does internally

The script typically sets environment variables such as:

| Variable | Purpose |
|------|------|
| `PATH` | Adds the LAMMPS binary directory so commands are globally accessible |
| `LD_LIBRARY_PATH` | Ensures required shared libraries can be found |

Example effect:

```bash
export PATH=/apps/lammps/22Jul2025/bin:$PATH
```

### Making it permanent for all users

System administrators can enable it globally:

```bash
cp /apps/lammps/22Jul2025/etc/profile.d/lammps.sh /etc/profile.d/
```

Reload environment:

```bash
source /etc/profile
```

Now every user on the system can run:

```bash
lmp -h
```

---
# LAMMPS Installation Test Documentation

## Purpose
This document records a **basic functionality test of LAMMPS** to confirm that the software runs correctly in **parallel using MPI**.

The test verifies:
- LAMMPS execution
- MPI parallel processing
- Successful completion of a simulation run

---

## Test Command

```bash
mpirun -np 16 lmp -in in.lj_16cores | tee 16_cores.log
```

Explanation:

- `mpirun` → launches MPI jobs
- `-np 16` → runs the job on **16 CPU cores**
- `lmp` → LAMMPS executable
- `-in in.lj_16cores` → input script
- `tee 16_cores.log` → saves output to a log file

---

## Input File Used for Testing

File: `in.lj_16cores`

```
# LAMMPS parallel test for 16 cores
# Lennard-Jones example (used only for testing)

units           lj
dimension       3
boundary        p p p
atom_style      atomic

lattice         fcc 0.8442
region          box block 0 16 0 16 0 16
create_box      1 box
create_atoms    1 box

mass            1 1.0

pair_style      lj/cut 2.5
pair_coeff      1 1 1.0 1.0 2.5

neighbor        0.3 bin
neigh_modify    delay 5

velocity        all create 1.44 87287

fix             1 all nve

thermo          500
thermo_style    custom step temp pe ke etotal press

timestep        0.005
run             20000
```

---

## Test Output 

LAMMPS started successfully:

```
LAMMPS (22 Jul 2025 - Update 3)
```

MPI processor layout:

```
2 by 2 by 4 MPI processor grid
```

Atoms created:

```
Created 16384 atoms
```

Simulation completed:

```
Loop time of 14.5137 on 16 procs for 20000 steps with 16384 atoms
```

CPU usage:

```
99.8% CPU use with 16 MPI tasks x 1 OpenMP threads
```

Total runtime:

```
Total wall time: 0:00:14
```

---

## Result

The simulation completed successfully without errors. This confirms that:

- LAMMPS is installed correctly
- MPI parallel execution is functioning
- The system can run multi-core LAMMPS jobs

---
# Note

The input file used in this test is based on **public LAMMPS example scripts available online** and is used only for **installation and functionality testing purposes**.

This work was performed **only for skill development and learning purposes** to build practical experience with LAMMPS installation, MPI execution, and multi-core simulation testing.

