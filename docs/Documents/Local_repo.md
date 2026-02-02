# **Creating and Configuring a Local Repository in RHEL (Without Subscription)**


This guide explains how to create and configure a **local YUM/DNF repository** on **RHEL** systems **without an active Red Hat subscription**.  
The repository can be created using **downloaded RPMs** or **RHEL ISO images**.

---

## 1. Install Required Dependencies

Install the required tool to generate repository metadata:


```bash
dnf install -y createrepo
```

---

## 2. Obtain Repository Packages

You can obtain RPM packages in **two ways**:

### Option 1: Download Packages Using `reposync`

If the system has temporary internet access:

```bash
reposync --reponame=<repository-name> -p /local-repo
```

Example:

```bash
reposync --reponame=baseos -p /local-repo
```

---

### Option 2: Use RHEL DVD ISO Images (Offline Method)

If you have RHEL ISO images (recommended for offline environments), you can configure the local repository using those images.

---

## 3. Create Directory Structure for Local Repositories

Create directories to store RPM files:

```bash
mkdir -p /local-repo/baseos
mkdir -p /local-repo/appstream
mkdir -p /local-repo/highavailability
```

---

## 4. Copy RPM Files to Local Repository

### 4.1 Copy from External Media (USB / Mounted Disk)

If the RPMs are already available on external media:

```bash
rsync -avz /run/media/<user>/RHEL/BaseOS/* /local-repo/baseos
rsync -avz /run/media/<user>/RHEL/AppStream/* /local-repo/appstream
rsync -avz /run/media/<user>/RHEL/HighAvailability/* /local-repo/highavailability
```

---

### 4.2 Copy from RHEL ISO Image

Mount the RHEL ISO image:

```bash
mkdir -p /mnt/iso_image
mount rhel9.4.iso /mnt/iso_image
```

Verify contents:

```bash
ls -l /mnt/iso_image
```

You should see directories such as `BaseOS` and `AppStream`.

Copy the RPMs:

```bash
rsync -avz /mnt/iso_image/BaseOS/* /local-repo/baseos
rsync -avz /mnt/iso_image/AppStream/* /local-repo/appstream
```

> ⚠️ **Note**
> The **HighAvailability** repository is **not included** in standard RHEL ISO images.
> It must be downloaded separately if required.

---

## 5. Create Repository Metadata

Generate metadata so `dnf` can recognize the repositories:

```bash
createrepo /local-repo/baseos
createrepo /local-repo/appstream
createrepo /local-repo/highavailability
```

> ⚠️ If errors occur, verify that RPM files were copied correctly.

---

## 6. Configure Local Repository Files

Create a repository configuration file:

```bash
vi /etc/yum.repos.d/local.repo
```

Add the following content:

```ini
[local-baseos]
name=Local BaseOS Repository
baseurl=file:///local-repo/baseos
enabled=1
gpgcheck=0

[local-appstream]
name=Local AppStream Repository
baseurl=file:///local-repo/appstream
enabled=1
gpgcheck=0

[local-highavailability]
name=Local HighAvailability Repository
baseurl=file:///local-repo/highavailability
enabled=1
gpgcheck=0
```

> 🔹 `gpgcheck=0` is recommended for offline/local repositories.
> If GPG keys are available, you may enable it.

---

## 7. Verify Repository Configuration

Check whether repositories are enabled:

```bash
dnf repolist
```

or

```bash
yum repolist
```

---

## 8. Clean DNF Cache

Clear existing cache and reload metadata:

```bash
dnf clean all
dnf makecache
```

---

## 9. Install Packages from Local Repository

Example: Install `pcs` package:

```bash
dnf install pcs
```

---

## 10. Verify PCS Installation

Check whether the service is installed and running:

```bash
systemctl status pcsd
```

Check available commands:

```bash
pcs --help
```

---

## Conclusion

Configuring a local repository in RHEL without a subscription allows systems to operate fully in **offline or restricted environments**.
Using ISO images or synced RPMs ensures consistent package availability and simplifies system management.

---
