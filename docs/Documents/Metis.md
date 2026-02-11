# Metis 5.1.0 
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

## 3. Prerequisites

Ensure the following dependencies are available:

- CMake (required)
- GCC or a compatible C/C++ compiler
- GKlib (included in the Metis source tree)
- Linux-based operating system

> ⚠️ **Important**  
> For Metis 5.1.0, **`make` must not be used directly**. The build process must be performed **exclusively via CMake**.

---

## 4. Cleanup Old Builds (If Any)

Before starting a fresh build, remove any existing build artifacts:

```bash
rm -rf build CMakeCache.txt CMakeFiles

```
This removes:

* Previous build directory
* Cached CMake configuration
* Temporary CMake files

---

## 5. Create Build Directory

It is recommended to build Metis in a separate directory.

```bash
mkdir build
cd build
```

If errors occur, you can safely delete this directory and retry.

---

## 6. Configure Using CMake

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

## 7. Build the Project

After successful configuration, compile Metis:

```bash
cmake --build . -j$(nproc)
```

This uses all available CPU cores for faster compilation.

> ⚠️ If errors occur, review the output carefully, fix the issue, and re-run the build.

---

## 8. Install Metis

Install Metis to the configured prefix:

```bash
cmake --install .
```

---

## 9. Verify Installation

Confirm that Metis was installed correctly:

```bash
ls /opt/Metis-5.1.0
```

Expected directories:

* `bin/` – Executable utilities
* `lib/` – Shared and static libraries
* `include/` – Header files

---

## 10. Post-Installation Notes

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

## 11. Conclusion

Metis 5.1.0 provides efficient and scalable graph and mesh partitioning capabilities.
When properly configured and installed, it can significantly enhance the performance of parallel and scientific computing applications.
