
````md
# METIS 5.1.0 Installation Guide (Using CMake)

This document explains how to install **METIS 5.1.0** using **CMake** with paths aligned to the **OpenFOAM ThirdParty directory structure**.

---

## 1. Overview

- **Software**: METIS
- **Version**: 5.1.0
- **Build System**: CMake
- **Target Platform**: Linux 64-bit
- **Integration**: OpenFOAM ThirdParty

> ⚠️ METIS 5.1.0 does **not** support `make config`.  
> Configuration must be done using **CMake only**.

---

## 2. Prerequisites

Ensure the following tools are available:

gcc --version
cmake --version
make --version

---

## 3. METIS Source Location

Example source directory:


/home/apple/OpenFoam2112/ThirdParty-v2112/sources/metis/metis-5.1.0/
├── GKlib/
├── include/
├── libmetis/
├── programs/
├── CMakeLists.txt

---

## 4. Clean Previous Builds (If Any)

From the METIS source directory:

rm -rf build CMakeCache.txt CMakeFiles

---

## 5. Create Build Directory


mkdir build
cd build


---

## 6. Configure METIS Using CMake

### Method 1: Relative `GKlib` Path

Run the following command **from inside the `build` directory**:

cmake \
-DGKLIB_PATH=../GKlib \
-DCMAKE_INSTALL_PREFIX=/home/apple/OpenFoam2112/ThirdParty-v2112/build/linux64Gcc/metis-5.1.0 \
-DSHARED=ON \
..


### Explanation

| Option                 | Description                                       |
| ---------------------- | ------------------------------------------------- |
| `GKLIB_PATH=../GKlib`  | Uses GKlib relative to the METIS source directory |
| `CMAKE_INSTALL_PREFIX` | Installation location for METIS                   |
| `SHARED=ON`            | Builds shared library (`libmetis.so`)             |

---

### Method 2: Absolute `GKlib` Path (Recommended for OpenFOAM)

cmake \
-DGKLIB_PATH=/home/apple/OpenFoam2112/ThirdParty-v2112/sources/metis/metis-5.1.0/GKlib \
-DCMAKE_INSTALL_PREFIX=/home/apple/OpenFoam2112/ThirdParty-v2112/platforms/linux64Gcc/metis-5.1.0 \
-DSHARED=ON
```

### When to Use This Method

* Preferred for **OpenFOAM builds**
* Avoids path ambiguity
* Works reliably with scripted builds

---

## 7. Build METIS

cmake --build . -j$(nproc) or make -j


### Sample Output

[100%] Built target metis

---

## 8. Install METIS


cmake --install  or make install


### Installed Files

metis-5.1.0/
├── include/
│   └── metis.h
├── lib/
│   ├── libmetis.so
│   └── libmetis.a


---

## 9. Set Environment Variables (Required)

Add the following to `~/.bashrc`:

export METIS_ARCH_PATH=/home/apple/OpenFoam2112/ThirdParty-v2112/platforms/linux64Gcc/metis-5.1.0
export LD_LIBRARY_PATH=$METIS_ARCH_PATH/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=$METIS_ARCH_PATH/lib:$LIBRARY_PATH
export CPATH=$METIS_ARCH_PATH/include:$CPATH


Reload environment:


source ~/.bashrc

---

## 10. Verify METIS Installation

### Verify Libraries

ls $METIS_ARCH_PATH/lib


Expected output:

libmetis.so  libmetis.a

---

### Verify Header File


ls $METIS_ARCH_PATH/include/metis.h

---

## 11. Common Errors and Fixes

### ❌ Error: `No rule to make target 'config'`

**Cause**: METIS 5.1.0 does not use `make config`
**Fix**: Use CMake only (as shown above)

### ❌ Error: `libmetis.so not found`

export LD_LIBRARY_PATH=$METIS_ARCH_PATH/lib:$LD_LIBRARY_PATH

## 12. Uninstall METIS


rm -rf /home/apple/OpenFoam2112/ThirdParty-v2112/platforms/linux64Gcc/metis-5.1.0

---

## 13. Summary

* METIS 5.1.0 must be built using **CMake**
* Use **absolute GKlib paths** for OpenFOAM
* Enable shared libraries with `-DSHARED=ON`
* Set environment variables before building OpenFOAM
* Verify detection using `foamSystemCheck`

---


