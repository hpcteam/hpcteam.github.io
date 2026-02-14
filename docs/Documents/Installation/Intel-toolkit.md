# Intel® oneAPI HPC Toolkit 2025.1 – Offline Installation Guide (Linux)

## Overview

This document describes the **offline installation procedure** for **Intel® oneAPI HPC Toolkit v2025.1** on a Linux system.
The installation was performed in a **Local Machine(Laptop)** using the **offline GUI installer**.

**Toolkit Version:** 2025.1

**Installation Mode:** Offline Installer (GUI)

**OS:** Rocky Linux / RHEL-based Linux

**Install Location:** `/home/admin/intel/oneapi`

---

## Prerequisites

Ensure the following before installation:

* Linux system with GUI (X11)
* Minimum **20 GB free disk space**
* Bash shell
* Offline installer available:

```bash
  intel-oneapi-hpc-toolkit-2025.1.0.666_offline.sh
```

---

## Step 1: Launch the Offline Installer

Run the installer from the terminal:

```bash
sh intel-oneapi-hpc-toolkit-2025.1.0.666_offline.sh
```

This extracts the package and launches the installer in **GUI mode**.

![Installer launch](/assets/intel/sh-intel.png)

---

## Step 2: Welcome Screen

The installer checks system requirements automatically.

Click **Continue** to proceed.

![Welcome screen](/assets/intel/sh-intel.png)

---

## Step 3: Select Components

For a complete HPC development environment, the following components were selected:

### Installed Components

* Intel® oneAPI DPC++ / C++ Compiler
* Intel® Fortran Compiler
* Intel® oneAPI Math Kernel Library (MKL)
* Intel® MPI Library
* Intel® oneAPI Threading Building Blocks (TBB)
* Intel® Advisor
* Intel® VTune™ Profiler
* Intel® Distribution for GDB
* Intel® DPC++ Compatibility Tool
* Intel® oneAPI Deep Neural Network Library

![Selecting features](/assets/intel/selecting-feactures.png)

---

## Step 4: IDE Integration (Optional)

IDE integration is optional and **not required for HPC servers or clusters**.

✔ Selected option:
**Skip Eclipse IDE configuration**

This is recommended for:

* Headless servers
* HPC compute nodes
* Production clusters

![Skip Eclipse IDE](/assets/intel/skip-Eclipse.png)

---

## Step 5: License Agreement & Installation Mode

* Accepted Intel® license agreement
* Selected **Recommended Installation**
* Installation path:

  ```
  /home/admin/intel/oneapi
  ```

![License and features](/assets/intel/license-and-feactures.png)

---

## Step 6: Installation in Progress

The installer performs integrity checks and installs all selected components.

* Integrity check: ✅ Completed
* Installation monitored via GUI

![Installing oneAPI](/assets/intel/installing.png)

---

## Step 7: Installation Completed Successfully

All components were installed successfully.

Installed toolkit:

* **Intel® oneAPI HPC Toolkit v2025.1**

![Installation finished](/assets/intel/finished.png)

---

## Post-Installation Setup

Initialize the oneAPI environment:

```bash
source /home/admin/intel/oneapi/setvars.sh
```

To make it persistent:

```bash
echo "source /home/admin/intel/oneapi/setvars.sh" >> ~/.bashrc
```

Verify installation:

```bash
icx --version
ifx --version
mpiicc -v
```

---

## Use Cases in HPC Environment

This setup supports:

* MPI-based parallel applications
* Fortran & C/C++ HPC workloads
* Numerical simulations (CFD, FEA)
* Performance profiling and tuning
* CPU-based AI/ML workloads

---

## Skills Demonstrated

* Offline software installation in restricted environments
* HPC compiler and MPI stack deployment
* Intel oneAPI toolchain configuration
* Linux system administration
* Performance analysis and optimization
