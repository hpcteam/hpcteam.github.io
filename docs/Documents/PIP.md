# PIP
### **What is `pip`?**

`pip` stands for **Pip Installs Packages** and is the default package manager for Python. It allows you to install, upgrade, and manage Python libraries and packages that are available in the Python Package Index (PyPI).

Here’s a quick overview of what `pip` does:

* **Install Packages**: You can use `pip` to install Python packages from PyPI or from a local directory.
* **Upgrade Packages**: You can upgrade installed packages to their latest versions.
* **Uninstall Packages**: You can remove unwanted packages from your Python environment.
* **List Packages**: You can list all the installed packages in your Python environment.

### **Basic Commands:**

1. **Install a Package**:

   To install a package from PyPI:

```
pip install package_name
```

   Example:

```bash
pip install numpy
```

2. **Upgrade a Package**:

   To upgrade an existing package to the latest version:

```bash
pip install --upgrade package_name
```

   Example:

```bash
pip install --upgrade numpy
```

3. **Uninstall a Package**:

   To remove an installed package:

```bash
pip uninstall package_name
```

   Example:

```bash
pip uninstall numpy
```

4. **List Installed Packages**:

   To list all the installed packages:

```bash
pip list
```

5. **Check Package Version**:

   To check the version of a specific package:

```bash
pip show package_name
```

   Example:

```bash
pip show numpy
```
---

### **Step 1: Download Python Packages on the Internet-Connected Machine**

#### **a. Create a `requirements.txt` File**

If you know which packages you want to install, create a `requirements.txt` file. For example:

```text
numpy
pandas
matplotlib
```

#### **b. Download Packages and Dependencies Using `pip`**

On an internet-connected machine, use `pip` to download the packages and their dependencies:

```bash
mkdir ~/offline_pkgs

pip download -r requirements.txt -d ~/offline_pkgs
```
![Creating Direcory](/assets/PIP/downloading-all-pacakages.png)

This command will download all the necessary `.whl` (wheel) files and dependencies into the `~/offline_pkgs` directory.

if you want to download a specific package (e.g., `numpy`), use:

```bash
pip download numpy -d ~/offline_pkgs
```
above command will download numpy with dependencies

#### **c. Transfer the Downloaded Files to the Offline System**

Once you have downloaded all the required files, transfer them to the offline system. You can use `scp`, `rsync`, or a USB drive to copy the `offline_pkgs` directory.

For example:

```bash
scp -r ~/offline_pkgs user@offline-server:/tmp/
```
```
scp -r ~/offline_pkgs root@(ip addres or hostname):/tmp/

```
---

### **Step 2: Install Python Packages Offline**

#### **a. Install Packages Using `pip`**

Once the `.whl` files are transferred to the offline system, you can install them using `pip`.

Navigate to the directory where the `.whl` files are stored (e.g., `/tmp/offline_pkgs`).

```bash
cd /tmp/offline_pkgs
```

Install the packages using `pip` with the `--no-index` option to avoid looking for the packages online:

```bash
pip install --no-index --find-links=/tmp/offline_pkgs -r /tmp/requirements.txt
```

This will install the packages from the `.whl` files in the `/tmp/offline_pkgs` directory.

if you downloaded a **specific package**, you can install it directly:

```bash
pip install --no-index --find-links=/tmp/offline_pkgs numpy-*.whl
```

#### **b. Verify Installation**

Once the packages are installed, you can verify the installation:

```bash
pip list | grep numpy
```

---

## **2. Additional Methods for Offline Installation**

### **Method 1: Offline Installation from Source (Tarballs)**

If you cannot use wheels for some packages, you can download the source code (tarballs) and build from source.

#### **Step 1: Download Source Code (Tarballs)**

On an internet-connected machine:

```bash
pip download --no-binary :all: numpy
```

This will download the source code for `numpy` as a `.tar.gz` file.

#### **Step 2: Transfer and Build on the Offline Machine**

Transfer the `.tar.gz` file to the offline system and install using:

```bash
pip install numpy-*.tar.gz
```



---

