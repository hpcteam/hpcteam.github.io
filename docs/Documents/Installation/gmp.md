# GMP (GNU Multiple Precision Arithmetic Library) Installation Guide

**Note:**  
This installation guide is intended **for self-learning and skill improvement** only. The installation is performed on a **personal computer** and is **not suitable for cluster-level installations**. It is meant for **reference purposes only** and should not be used in production or large-scale environments.

## What is GMP?

The **GNU Multiple Precision Arithmetic Library (GMP)** is a free and open-source library that provides arbitrary-precision arithmetic. It supports integer, rational, and floating-point arithmetic with arbitrary precision. GMP is an essential dependency for **GCC** (GNU Compiler Collection), enabling precise arithmetic operations that are necessary during the compilation process.

## Why is GMP Needed?

GMP is required for:

* **High-Precision Arithmetic**: Enables efficient and accurate calculations during the compilation process in GCC.
* **Optimized Performance**: Handles complex calculations and mathematical operations, which are essential for building software that depends on precise number crunching.
* **Core Dependency**: GMP provides essential functionality that other tools and libraries in GCC require, making it an indispensable part of the build process.

---

## Installation Steps

### Step 1: Download the Source Code

First, download the GMP source code from the official FTP server:

```bash
wget https://ftp.gnu.org/gnu/gmp/gmp-<version>.tar.lz
```

Replace `<version>` with the desired version (e.g., `6.2.1` or any version you are using). This will download the source code to your local machine.

### Step 2: Extract the Source Code

Once the source code has been downloaded, extract the contents of the `.tar.lz` archive using the following command:

```bash
tar -xvf gmp-<version>.tar.lz
```

After extracting the files, verify that the directory has been created by running:

```bash
ls
```

This command will show the list of files and directories, and you should see the `gmp-<version>` directory.

### Step 3: Check for the `configure` Script

Navigate into the extracted GMP directory:

```bash
cd gmp-<version>
```

Now, run the `ls` command to check if the `configure` script is present:

```bash
ls
```

* If the `configure` script is found, you can proceed to the next step.
* If not, check whether an `autogen.sh` script is available. If it is, run the following command to generate the `configure` script:

```bash
./autogen.sh
```

This will create the `configure` script needed for building GMP.

### Step 4: Run the `./configure` Script

To prepare for the installation, run the `configure` script, which checks your system for necessary tools and libraries. To see available configuration options, use:

```bash
./configure --help
```

This command will display a list of options that can be used during the configuration process. The most important option is the **`--prefix`** option, which determines where GMP will be installed on your system.

### Step 5: Run `./configure` with the Prefix Option

Run the `configure` script with the `--prefix` option to specify the installation directory:

```bash
./configure --prefix=/path/to/install
```

Replace `/path/to/install` with the directory where you want GMP to be installed (e.g., `/usr/local/gmp`). This ensures GMP is installed in the specified location on your system.

* **`configure` File Check:**
  ![Configure Check](/assets/gmp/configure.png)

### Step 6: Compile the Code

Now that the configuration is complete, compile the GMP source code using the `make` command. The `-j` flag enables parallel compilation, utilizing multiple CPU cores to speed up the build process:

```bash
make -j
```

The `-j` option allows you to specify the number of cores to use, for example, `-j4` for four cores. This can significantly speed up the compilation, especially on systems with multiple cores.

* **Running the `make -j` Command:**
  ![Make -j](/assets/gmp/make-j.png)

After compilation, you can run a check to ensure that the build process was successful:

* **Make Check:**
  ![Make Check](/assets/gmp/make-check.png)

### Step 7: Install GMP

Once the compilation is complete, install GMP by running:

```bash
sudo make install
```

This command installs GMP into the directory specified by the `--prefix` option.

* **Running `make install`:**
  ![Make Install](/assets/gmp/makeinstall.png)

### Step 8: Verify Installation

Finally, verify the installation by checking the directories to ensure that the necessary GMP files were created.

```bash
ls /path/to/install/

```

* **Final Check:**
  ![Final Check](/assets/gmp/finalcheck.png)

---


