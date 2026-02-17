## MPC (Multiple Precision Complex Library) Installation Guide

**Note:**  
This installation guide is intended **for self-learning and skill improvement** only. The installation is performed on a **personal computer** and is **not suitable for cluster-level installations**. It is meant for **reference purposes only** and should not be used in production or large-scale environments.

## What is MPC?

The **Multiple Precision Complex Library (MPC)** is a free library designed for complex floating-point arithmetic with multiple precision. MPC is often used alongside **MPFR** and **GMP** to provide precision in complex number computations that MPFR handles for floating-point numbers.

MPC is a core dependency for **GCC** as it aids in complex number calculations during the compilation process.

## Why is MPC Needed?

MPC is required for:

* **High-Precision Complex Calculations**: Provides support for multiple precision complex arithmetic operations.
* **Scientific Computations**: Used in fields requiring high accuracy in complex number calculations, like engineering, physics, and computer science.
* **Core Dependency**: MPC supports MPFR by providing arithmetic for complex numbers, which is essential for scientific software and GCC.

---

## Installation Steps

### Step 1: Download the Source Code

Download the **MPC 1.3.1** source code manually from the official website:

[MPC Official Website](https://www.multiprecision.org/)

Make sure to download the `.tar.gz` file for version 1.3.1.

### Step 2: Transfer the Source Code to the Offline Machine

Once the source code is downloaded, transfer the `.tar.gz` file to your offline machine. You can use any method of file transfer that works in your environment (e.g., USB drive, external hard drive).

### Step 3: Extract the Source Code

On your offline machine, extract the MPC source code using the following command:

```bash
tar -xvzf mpc-1.3.1.tar.gz
```

After extracting the files, use the `ls` command to check if the extraction was successful and if the `mpc-1.3.1` directory is created:

```bash
ls
```

You should see the extracted `mpc-1.3.1` directory.

### Step 4: Check for the `configure` Script

Navigate into the MPC directory:

```bash
cd mpc-1.3.1
```

List the files to check if the `configure` script is present:

```bash
ls
```
* **Directory Listing after Extraction**
  ![Directory Listing](/assets/mpc/tarxfmpc.png)
  
* If the `configure` script is found, proceed to the next step.
* If the `configure` script is missing, check for the `autogen.sh` script. If present, run it to generate the `configure` script:

```bash
./autogen.sh
```

This will create the `configure` script, which is necessary for the next steps.

### Step 5: Link GMP and MPFR as Dependencies

MPC depends on **GMP** and **MPFR** for its functionality. There are two methods to link these dependencies during the MPC installation process:

#### Method 1: Export Environment Variables for GMP and MPFR

You can export the installation directories for GMP and MPFR before running the `configure` script. This method allows MPC to detect these libraries automatically.

```bash
export GMP_DIR=/path/to/gmp
export MPFR_DIR=/path/to/mpfr
export PATH=$GMP_DIR/bin:$MPFR_DIR/bin:$PATH
export LD_LIBRARY_PATH=$GMP_DIR/lib:$MPFR_DIR/lib:$LD_LIBRARY_PATH
export C_INCLUDE_PATH=$GMP_DIR/include:$MPFR_DIR/include:$C_INCLUDE_PATH
```

Replace `/path/to/gmp` and `/path/to/mpfr` with the correct paths to GMP and MPFR installations.

#### Method 2: Using the `--with-gmp` and `--with-mpfr` Flags

Alternatively, you can specify the GMP and MPFR directories directly in the `configure` script using the `--with-gmp` and `--with-mpfr` options:

```bash
./configure --prefix=/path/to/install --with-gmp=/path/to/gmp --with-mpfr=/path/to/mpfr
```

Replace `/path/to/gmp`, `/path/to/mpfr`, and `/path/to/install` with the correct directories.

 **`configure` with GMP and MPFR Path Options**
  ![MPC Configure](/assets/mpc/configure--mpfr-gmp.png)

### Step 6: Run the `./configure` Script

Run the `configure` script to prepare the build environment:

```bash
./configure --prefix=/path/to/install --with-gmp=/path/to/gmp --with-mpfr=/path/to/mpfr
```

You can check available configuration options by running:

```bash
./configure --help
```

This will display the configuration options. Ensure that the GMP and MPFR paths are correctly set.
* **`configure` --help**
  ![MPC Configure](/assets/mpc/configure--help.png)

### Step 7: Compile the Code

Once the configuration is complete, compile MPC by running:

```bash
make -j
```

The `-j` flag utilizes multiple CPU cores to speed up the compilation process. Adjust the number of cores based on your system's capabilities (e.g., `-j4` for four cores).

* **Running the `make -j` Command:**
  ![Make -j](/assets/mpc/make-j.png)

You can also verify that the compilation process ran smoothly by performing a check:

```bash
make check
```

### Step 8: Install MPC

Once the compilation is finished, install MPC using:

```bash
sudo make install
```

This command installs MPC into the directory specified by the `--prefix` option during the configuration process.

* **Running `make install`**
  ![Make Install](/assets/mpc/makeinstall.png)

### Step 9: Verify Installation

After the installation is complete, verify that MPC has been installed correctly by checking the contents of the installation directory:

```bash
ls /path/to/install/bin
ls /path/to/install/lib
```

You should see the necessary MPC files in the `bin` and `lib` directories.

* **Final Installation Check**
  ![Final Check](/assets/mpc/final-check-directory-createdor-notpng)

---

## Screenshots

Here are the screenshots demonstrating the installation steps:

* **Directory Listing after Extraction**
  ![Directory Listing](/assets/mpc/tarxfmpc.png)

* **`configure` with GMP and MPFR Path Options**
  ![MPC Configure](/assets/mpc/configure--mpfr-gmp.png)

* **Running the `make -j` Command**
  ![Make -j](/assets/mpc/make-j.png)

* **Running `make install`**
  ![Make Install](/assets/mpc/makeinstall.png)

* **Final Installation Check**
  ![Final Check](/assets/mpc/final-check-directory-createdor-notpng)


