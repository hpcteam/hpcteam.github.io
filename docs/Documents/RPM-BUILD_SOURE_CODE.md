
##  Important Notes

This document describes how I installed and configured `rpmbuild` from source, set up the necessary environment for building RPM packages, and successfully built an RPM for Slurm. The steps outlined here were part of my self-learning journey to enhance my skills in RPM package creation and cluster management.

- **Self-learning Focus**: This guide is designed for self-learning and improving my skills in RPM packaging and system administration. It is not intended for deployment in production environments without further testing and adjustments.
- **Tested on Laptop**: The setup and build process described in this document were tested on my personal laptop. For larger-scale or multi-node environments, additional configuration and testing are required to ensure compatibility.
- **Build Dependencies**: During the process, I encountered dependency issues that were resolved by installing the required `-devel` packages. Ensure all dependencies are checked before starting the build process.

## 📌 What is `rpmbuild`?

**`rpmbuild`** is a Linux tool used to **create RPM packages from source code**.

### In simple words:

> **`rpmbuild` converts source code into `.rpm` files**

These RPM files can then be installed, upgraded, or removed using standard package managers like:

* `dnf`
* `rpm`

---

## 📌 Why RPM Packages Are Used

On **Rocky Linux**, **RHEL**, and **CentOS**, software is managed using **RPM packages**.

**RPM** stands for **Red Hat Package Manager**.

RPM packages help to:

* Install software easily
* Track installed versions
* Remove software cleanly
* Upgrade software safely
* Manage dependencies automatically

---

## 📌 Problems Without `rpmbuild`

If software is installed directly from source using:

```bash
./configure
make
make install
```

It creates several issues:

❌ No record of installed files

❌ Difficult to uninstall

❌ Hard to upgrade

❌ Not suitable for multiple systems

❌ No dependency tracking

This approach is **not recommended** for production or cluster environments.

---

## 1. Introduction

In RHEL-based systems like **Rocky Linux**, software distribution is done using **RPM packages**.
The `rpmbuild` tool is used to convert **source code** into **RPM files** that follow system standards.

This document explains step by step how to:

* Set up an RPM build environment
* Build RPMs using `rpmbuild`
* Understand build dependency failures
* Successfully generate **Slurm RPM packages**

---

## 2. Why Use `rpmbuild`

### Why it is needed

* Manual source installations are hard to manage
* No clean uninstall or upgrade path
* Difficult to deploy across multiple nodes

### Benefits of `rpmbuild`

* Standard package management
* Clean install and uninstall
* Version tracking
* Easy cluster-wide deployment (HPC-friendly)

---

## 3. Install Required Tools

### Purpose

To prepare the system for building RPMs from source code.

```bash
dnf install rpm-build rpmdevtools gcc make autoconf automake libtool wget tar
```

![Installing rpm-build tools](/assets/source-rpm/yuminstall.png)

---

## 4. Verify `rpmbuild` Installation


Ensure `rpmbuild` is installed and available.



```bash
rpm -qa | grep rpm-build
```

![Checking rpm-build installation](/assets/source-rpm/checking-rpm-build-installed-or-not.png)

---

## 5. Create RPM Build Directory Structure



`rpmbuild` requires a **standard directory layout** to work correctly.



```bash
rpmdev-setuptree
```

### Created structure

```text
~/rpmbuild/
├── BUILD
├── RPMS
├── SOURCES
├── SPECS
└── SRPMS
```

![rpmdev setuptree](/assets/source-rpm/rpmdev-setuptree.png)

---

## 6. Download Slurm Source Code



RPM packages are always built from **source archives**.



```bash
wget https://download.schedmd.com/slurm/slurm-24.11.7.tar.bz2
```

Move the source archive to the SOURCES directory:

```bash
mv slurm-24.11.7.tar.bz2 ~/rpmbuild/SOURCES/
```

![Downloading Slurm source code](/assets/source-rpm/downloading sourecode.png)

---

## 7. Extract Source Code


Verify the contents of the source archive before building.


```bash
cd ~/rpmbuild/SOURCES
tar xf slurm-24.11.7.tar.bz2
ls
```

![Extracting source archive](/assets/source-rpm/tar extract.png)

---

## 8. Copy Slurm SPEC File


The **SPEC file** controls the entire RPM build process.



```bash
cp slurm-24.11.7/slurm.spec ~/rpmbuild/SPECS/
```

![Copying spec file](/assets/source-rpm/cpoing.specfile.png)

---

## 9. Run `rpmbuild` 


```bash
cd ~/rpmbuild/SPECS
rpmbuild -ba slurm.spec
```



The build fails due to **missing build dependencies**.

![rpmbuild failed](/assets/source-rpm/rpmbuild-faild.png)

---

##  Why the Build Failed

`rpmbuild` checks the **BuildRequires** section in the SPEC file.

If required `*-devel` packages are missing:

* The build stops immediately
* This prevents creating broken RPMs

This behavior is **expected and correct**.

---

## 10. Install Slurm Build Dependencies

### Required packages

* `mariadb-devel`
* `munge-devel`
* `pam-devel`
* `readline-devel`



```bash
dnf install mariadb-devel munge-devel pam-devel readline-devel
```

![Installing build dependencies](/assets/source-rpm/installed-dependeces-for-source-pacaage.png)

---

## 11. Run `rpmbuild` Again 



```bash
rpmbuild -ba slurm.spec
```

![rpmbuild output](/assets/source-rpm/rpmbuild.png)

![rpmbuild success](/assets/source-rpm/rpmbuild-final-sucess.png)

---

### What happens internally during `rpmbuild`

1. `%prep` – Unpacks source code
2. `%build` – Compiles Slurm
3. `%install` – Installs files into buildroot
4. `%files` – Selects files for packaging
5. RPM packages are generated

---

## 12. Generated RPM Packages

### Location

```bash
~/rpmbuild/RPMS/x86_64/
```

### Example RPMs

* `slurm-24.11.7-1.el9.x86_64.rpm`
* `slurm-slurmd-24.11.7-1.el9.x86_64.rpm`
* `slurm-slurmctld-24.11.7-1.el9.x86_64.rpm`
* `slurm-slurmdbd-24.11.7-1.el9.x86_64.rpm`
* `slurm-devel-24.11.7-1.el9.x86_64.rpm`

![RPMs created successfully](/assets/source-rpm/rpm-created.png)

---

##  Final Summary

* `rpmbuild` is the **standard RPM build tool** for RHEL-based systems
* It converts **source code → RPM packages**
* SPEC file controls the entire process
* Dependency checks ensure safe and reliable builds
* This method is **production-ready and HPC-friendly**


