# HWLOC Installation Guide

## Introduction to HWLOC

**HWLOC (Hardware Locality)** is an open-source software package that provides a comprehensive view of the hardware architecture of a system. It allows you to see how your system's hardware is structured, such as CPUs, memory, NUMA nodes, caches, and other hardware resources. This tool is especially useful for developers and system administrators working in **high-performance computing (HPC)** environments, where understanding and optimizing the layout of hardware resources can significantly improve performance.

### Why Use HWLOC?

1. **Hardware Topology Awareness**:
   HWLOC provides detailed information about the physical layout of hardware in a machine. It helps you understand how the CPU cores, caches, memory, and other resources are organized, which is crucial for performance tuning, particularly in multi-threaded applications.

2. **Optimizing Performance in HPC Systems**:
   In HPC systems, performance can be affected by the placement of processes and memory. HWLOC helps optimize process binding, which refers to placing processes on specific cores or NUMA nodes to reduce latency and improve bandwidth utilization.

3. **Cross-platform Support**:
   HWLOC supports a wide range of platforms, including Linux, macOS, and Windows, making it a versatile tool for system administrators across different environments.

4. **Compatibility with MPI**:
   HWLOC is often used with MPI (Message Passing Interface) applications. It helps manage process affinity in parallel computing, ensuring that processes are placed optimally on the hardware.

---

## Prerequisites

Ensure that you have the necessary tools to compile the source code:

* `wget`
* `tar`
* `make`
* `gcc`

---

## Steps to Install HWLOC

### 1. **Download HWLOC Source Code**

First, download the HWLOC source tarball using `wget`:

```bash
wget https://download.open-mpi.org/release/hwloc/v2.5/hwloc-2.5.0.tar.bz2
```

You should see output similar to the following:

![Image 1](/assets/hwclock/hwclockintsallation1.png)

---

### 2. **Extract the Tarball**

After the download is complete, extract the contents of the tarball:

```bash
tar xf hwloc-2.5.0.tar.bz2
```

---

### 3. **Configure the Build**

Navigate into the extracted directory:

```bash
cd hwloc-2.5.0/
```

Now, run the `configure` script to prepare the build environment:

```bash
./configure
```

This will check for the build system and necessary dependencies. The output should look similar to this:

![Image 2](/assets/hwclock/hwclockconfigure2.png)

---

### 4. **Compile HWLOC**

Once the configuration is complete, proceed with building HWLOC using the `make` command:

```bash
make
```

During this step, the compilation process will begin:

![Image 3](/assets/hwclock/hwclock-make3.png)

---

### 5. **Install HWLOC**

After the compilation is finished, install the binaries:

```bash
make install
```

This will install the HWLOC libraries and executables on your system:

![Image 4](/assets/hwclock/hwclockmakeinstall4.png)

---

### 6. **Verification**

After the installation, you can verify that HWLOC has been installed successfully by running:

```bash
lstopo
```

![Image 5](/assets/hwclock/hwlotopo.png)

This should display the hardware topology of your system.

---

This document serves as a basic guide to help with HWLOC installation from the source on a Linux machine.

---

