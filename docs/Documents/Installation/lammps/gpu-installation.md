# GPU-Based Installation of LAMMPS

## 1. Requirements

To run LAMMPS on GPU nodes, the following components are required:

| Component      | Purpose                                                    |
| -------------- | ---------------------------------------------------------- |
| NVIDIA GPU     | Hardware accelerator used for high-performance simulations |
| CUDA Toolkit   | Provides libraries and tools for GPU computing             |
| MPI            | Enables parallel execution across multiple CPU cores       |
| CMake          | Build system used to configure and compile LAMMPS          |
| GCC / Compiler | Compiles the LAMMPS source code                            |

LAMMPS uses **GPU acceleration** to offload computationally intensive operations such as **pairwise force calculations between atoms**, significantly improving simulation performance.

---

# 2. Check GPU Availability

Before installing LAMMPS with GPU support, verify that the GPU is available on the node.

```bash
nvidia-smi
```

Example output:

```
NVIDIA-SMI 535.xx
GPU Name: Tesla V100 / A100 / RTX
```

Purpose:

* Confirms that the **GPU driver is installed**
* Verifies that the **GPU node is accessible**

---

# 3. Install CUDA Toolkit

LAMMPS GPU support requires the **CUDA Toolkit**.

Check CUDA installation:

```bash
nvcc --version
```

If CUDA is not installed, install it:

```bash
sudo dnf install cuda
```

or install it from the official NVIDIA CUDA repository.

CUDA provides:

* GPU libraries
* CUDA compiler (`nvcc`)
* GPU runtime environment

---

# 4. Install Required Build Tools

Install the necessary development tools:

```bash
sudo dnf install gcc gcc-c++ cmake git openmpi openmpi-devel
```

Explanation:

| Tool    | Purpose                          |
| ------- | -------------------------------- |
| gcc     | Compiles C++ source code         |
| cmake   | Configures the build environment |
| openmpi | Enables MPI parallel execution   |
| git     | Downloads the LAMMPS source code |

---

# 5. Download LAMMPS Source Code

LAMMPS source code can be downloaded from the official website:

[https://www.lammps.org/download.html](https://www.lammps.org/download.html)

The website provides:

* **Stable Release (recommended)**
* **Feature Release (latest development version)**

Example release:

```
LAMMPS Stable Release – 22 Jul 2025
```

You can also download using Git:

```bash
git clone https://github.com/lammps/lammps.git
cd lammps
```

---

# 6. Create Build Directory

LAMMPS uses an **out-of-source build** to keep compiled files separate from the source code.

```bash
mkdir build
cd build
```

---

# 7. Configure LAMMPS with GPU Support

Run the following CMake command to enable GPU acceleration:

```bash
cmake ../cmake \
-D BUILD_MPI=yes \
-D PKG_GPU=yes \
-D GPU_API=cuda \
-D GPU_ARCH=sm_80
```

### Explanation of Flags

| Flag             | Meaning                                               |
| ---------------- | ----------------------------------------------------- |
| `BUILD_MPI=yes`  | Enables MPI parallel computing                        |
| `PKG_GPU=yes`    | Enables the LAMMPS GPU package                        |
| `GPU_API=cuda`   | Uses NVIDIA CUDA for GPU acceleration                 |
| `GPU_ARCH=sm_80` | Optimizes the build for the specific GPU architecture |

### GPU Architecture Examples

| GPU        | Architecture Flag |
| ---------- | ----------------- |
| Tesla V100 | `sm_70`           |
| Tesla A100 | `sm_80`           |
| RTX 3090   | `sm_86`           |

---

# 8. Compile LAMMPS

Compile the software using multiple CPU cores:

```bash
make -j 48
```

Explanation:

* `-j 48` allows compilation using **48 CPU cores**, which speeds up the build process.

---

# 9. Verify GPU Support

After compilation, verify that the GPU package is enabled:

```bash
./lmp -h | grep GPU
```

Expected output:

```
GPU package installed
```

---

# 10. Run a GPU Test Simulation

Run a simple GPU simulation test:

```bash
mpirun -np 16 ./lmp -sf gpu -pk gpu 1 -in in.lj
```

Explanation:

| Option      | Meaning                             |
| ----------- | ----------------------------------- |
| `-sf gpu`   | Enables GPU-accelerated styles      |
| `-pk gpu 1` | Uses one GPU device                 |
| `-np 16`    | Runs the simulation on 16 CPU cores |

---

# 11. Monitor GPU Usage

While the simulation is running, verify GPU utilization:

```bash
nvidia-smi
```

You should see **LAMMPS processes using GPU memory and compute resources**.

---

# Advantages of GPU-Accelerated LAMMPS

GPU acceleration offers several benefits:

* **10×–100× faster simulations**
* Faster **pair interaction calculations**
* Improved performance for **large-scale molecular dynamics simulations**

This is particularly useful for:

* Large atomistic systems
* Long simulation times
* Materials science and nanotechnology research

---

# Example HPC GPU Execution

Example production run on a GPU node:

```bash
mpirun -np 48 lmp -sf gpu -pk gpu 2 -in input.in
```

Resource usage:

| Resource      | Value |
| ------------- | ----- |
| CPU Cores     | 48    |
| GPUs          | 2     |
| MPI Processes | 48    |

---

# Simple GPU Test Input Script

Example LAMMPS input file used for GPU testing:

```bash
units lj
atom_style atomic
lattice fcc 0.8442
region box block 0 20 0 20 0 20
create_box 1 box
create_atoms 1 box

pair_style lj/cut 2.5
pair_coeff 1 1 1.0 1.0 2.5

velocity all create 1.0 12345
fix 1 all nve

run 5000
```

---

# Important Notes

Before compiling LAMMPS with GPU support, verify the following:

1. GPU driver is installed correctly
2. CUDA version is compatible with the GPU driver
3. MPI libraries are properly installed

Incorrect configuration of these components may cause **GPU build or runtime failures**.

---

