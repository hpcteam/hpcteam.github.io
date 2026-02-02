# Metis 5.1.0 Theory and HPC Implications


---

## 1. Introduction

**Metis** is a high-performance library for **graph partitioning**, **mesh partitioning**, and **sparse matrix ordering**. It is widely used in scientific and high-performance computing applications where large graphs and meshes must be efficiently decomposed.

Metis helps in:

- Reducing communication overhead in parallel applications
- Improving cache efficiency
- Optimizing load balancing across processors

Typical application areas include:

- Finite Element Analysis (FEA)
- Computational Fluid Dynamics (CFD)
- Large-scale graph analytics
- Scientific simulations

---

## 2. Key Features

- Multilevel graph partitioning algorithms
- k-way and recursive bisection partitioning
- Mesh partitioning support
- Sparse matrix reordering (nested dissection)
- Efficient handling of large-scale graphs
- Shared and static library support

---

## 3. Architecture Overview

Metis uses a **multilevel partitioning strategy**, which consists of three phases:

### 3.1 Coarsening Phase

The original graph is reduced in size by collapsing vertices and edges while preserving the overall structure.

### 3.2 Initial Partitioning Phase

The coarsened graph is partitioned using fast and lightweight algorithms.

### 3.3 Uncoarsening and Refinement Phase

The partition is projected back to the original graph, with refinement at each level to improve partition quality.

This approach provides high-quality partitions with low computational cost.

---

## 4. Prerequisites

Ensure the following dependencies are available:

- CMake (required)
- GCC or a compatible C/C++ compiler
- GKlib (included in the Metis source tree)
- Linux-based operating system

> ⚠️ **Important**  
> For Metis 5.1.0, **`make` must not be used directly**. The build process must be performed **exclusively via CMake**.

---

## 5. Cleanup Old Builds (If Any)

Before starting a fresh build, remove any existing build artifacts:

```bash
rm -rf build CMakeCache.txt CMakeFiles

```
This removes:

* Previous build directory
* Cached CMake configuration
* Temporary CMake files

---

## 6. Create Build Directory

It is recommended to build Metis in a separate directory.

```bash
mkdir build
cd build
```

If errors occur, you can safely delete this directory and retry.

---

## 7. Configure Using CMake

Run the following command from inside the `build` directory:

```bash
cmake \
  -DGKLIB_PATH=../GKlib \
  -DCMAKE_INSTALL_PREFIX=/opt/Metis-5.1.0 \
  -DSHARED=ON \
  ..
```

### CMake Options Explained

* **`-DGKLIB_PATH=../GKlib`**
  Specifies the path to the GKlib directory bundled with Metis.

* **`-DCMAKE_INSTALL_PREFIX=/opt/Metis-5.1.0`**
  Defines the target installation directory.

* **`-DSHARED=ON`**
  Builds shared libraries (`.so`).

### Example (OpenFOAM Integration)

```bash
cmake \
  -DGKLIB_PATH=/opt/OpenFoam2112/ThirdParty-v2112/sources/metis/metis-5.1.0/GKlib \
  -DCMAKE_INSTALL_PREFIX=/opt/OpenFoam2112/ThirdParty-v2112/platforms/linux64Gcc/metis-5.1.0 \
  -DSHARED=ON \
  ..
```

---

## 8. Build the Project

After successful configuration, compile Metis:

```bash
cmake --build . -j$(nproc)
```

This uses all available CPU cores for faster compilation.

> ⚠️ If errors occur, review the output carefully, fix the issue, and re-run the build.

---

## 9. Install Metis

Install Metis to the configured prefix:

```bash
cmake --install .
```

---

## 10. Verify Installation

Confirm that Metis was installed correctly:

```bash
ls /opt/Metis-5.1.0
```

Expected directories:

* `bin/` – Executable utilities
* `lib/` – Shared and static libraries
* `include/` – Header files

---

## 11. Post-Installation Notes

### Update Library Path

If the library directory is not in the system default path:

```bash
export LD_LIBRARY_PATH=/opt/Metis-5.1.0/lib:$LD_LIBRARY_PATH
```

### Linking Against Metis

Applications using Metis must:

* Include headers from `include/`
* Link against libraries in `lib/`

---

## 12. Conclusion

Metis 5.1.0 provides efficient and scalable graph and mesh partitioning capabilities.
When properly configured and installed, it can significantly enhance the performance of parallel and scientific computing applications.

```

---
