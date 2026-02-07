---
# Lustre Server Installation Guide (EL9 / RHEL9)
**MGS + MDT using ldiskfs**

---

## Environment
- OS: RHEL 9 / Rocky Linux 9 / AlmaLinux 9
- Architecture: x86_64
- Lustre Version: 2.16.x
- Filesystem Name: `hpcfs`
- Target Disk: `/dev/nvme0n2`
- Mount Point: `/lustre01`

---

## Step 1: Update System

```
dnf clean all
dnf update -y
reboot
```

---

## Step 2: Install Kernel Headers and Build Tools

Lustre kernel modules must match the running kernel.

```bash
dnf install -y \
  kernel-devel-$(uname -r) \
  kernel-headers-$(uname -r) \
  gcc make elfutils-libelf-devel
```

Verify:

```bash
uname -r
ls /usr/src/kernels/$(uname -r)
```

---

## Step 3: Install Lustre Repository

Use **only one Lustre version** (do not mix releases).

```bash
dnf install -y \
https://downloads.lustre.org/public/lustre/latest-release/el9/x86_64/lustre-release-2.16.0-1.el9.x86_64.rpm
```

Verify:

```bash
dnf repolist | grep lustre
```

---

## Step 4: Refresh DNF Metadata

```bash
dnf clean all
dnf makecache
```

---

## Step 5: Install Lustre Server Packages

```bash
dnf install -y \
  lustre \
  kmod-lustre-osd-ldiskfs \
  e2fsprogs \
  ldiskfsprogs
```

Verify modules:

```bash
rpm -qa | grep lustre
```

---

## Step 6: Load Lustre Kernel Module

```bash
modprobe lustre
```

Verify:

```bash
lsmod | grep lustre
dmesg | grep -i lustre
```

---

## Step 7: Remove ZFS (If Installed)

ZFS conflicts with ldiskfs.

```bash
rpm -qa | grep zfs
dnf remove -y zfs
reboot
```

After reboot:

```bash
modprobe lustre
```

---

## Step 8: Verify Block Device

⚠️ **This step destroys all data on the device**

```bash
lsblk
fdisk -l /dev/nvme0n2
```

---

## Step 9: Create Lustre Filesystem (MGS + MDT)

```bash
mkfs.lustre \
  --fsname=hpcfs \
  --mgs \
  --mdt \
  --index=0 \
  --reformat \
  /dev/nvme0n2
```

Expected output includes:

```
Target: hpcfs:MDT0000
Mount type: ldiskfs
(MDT MGS first_time)
```

---

## Step 10: Mount Lustre Filesystem

```bash
mkdir -p /lustre01
mount -t lustre /dev/nvme0n2 /lustre01
```

Verify:

```bash
df -Th | grep lustre
```

---

## Step 11: Validate Lustre Services

```bash
lctl dl
lctl status
lfs df
```

---

## Step 12: Configure Mount at Boot

Edit `/etc/fstab`:

```fstab
/dev/nvme0n2  /lustre01  lustre  defaults,_netdev  0 0
```

Test:

```bash
umount /lustre01
mount -a
```

---

## Step 13: Final Validation

```bash
ls /lustre01
touch /lustre01/testfile
ls -l /lustre01
```

---

## Optional: Enable Cockpit

```bash
systemctl enable --now cockpit.socket
```

---

## Verification Commands Reference

```bash
lsmod | grep lustre
lctl dl
lctl status
lfs df
dmesg | grep lustre
```

---

## Notes

* Do **not** mix Lustre versions in repositories.
* Kernel updates require reinstalling matching `kmod-lustre`.
* Use separate devices for MDT and OSTs in production.

---

## Next Steps

* Add OSTs
* Configure Lustre clients
* Performance tuning (MDT journal, inode sizing)
* HA with Pacemaker + Corosync

---

# Lustre OSS (OST Server) Installation Guide – EL9
This document describes how to configure a **Lustre Object Storage Server (OSS)** and create an **OST** connected to an existing MGS.

---

## Environment
- OS: RHEL 9 / Rocky Linux 9 / AlmaLinux 9
- Hostname: `OSS`
- Lustre Version: 2.16.x
- Filesystem Name: `hpcfs`
- MGS IP: `192.168.1.13`
- Network: TCP
- OST Device: `/dev/nvme0n2`
- OST Mount Point: `/OST`

---

## Step 1: Set Hostname
```
hostnamectl set-hostname OSS
reboot
```

Verify:

```
hostname
```

---

## Step 2: Install Kernel Headers and Tools

```bash
dnf install -y \
  kernel-devel-$(uname -r) \
  kernel-headers-$(uname -r) \
  gcc make elfutils-libelf-devel
```

---

## Step 3: Install Lustre Repository

⚠️ Ensure the same Lustre version is used on **MGS, MDT, and OSS**.

```bash
dnf install -y \
https://downloads.lustre.org/public/lustre/latest-release/el9/x86_64/lustre-release-2.16.0-1.el9.x86_64.rpm
```

Verify:

```bash
dnf repolist | grep lustre
```

---

## Step 4: Install Lustre OSS Packages

```bash
dnf clean all
dnf makecache
dnf install -y \
  lustre \
  kmod-lustre-osd-ldiskfs \
  e2fsprogs \
  ldiskfsprogs
```

---

## Step 5: Load Lustre Kernel Module

```bash
modprobe lustre
```

Verify:

```bash
lsmod | grep lustre
dmesg | grep -i lustre
```

---

## Step 6: System Tuning (Recommended)

Increase minimum free memory for OST stability.

```bash
sysctl -w vm.min_free_kbytes=262144
echo "vm.min_free_kbytes=262144" >> /etc/sysctl.conf
```

Optional Lustre tuning:

```bash
cat << EOF > /etc/modprobe.d/lustre.conf
options lustre osc_page_pool=16
options lustre osc_page_pool_max=32
options lustre obd_timeout=600
EOF
```

Apply:

```bash
sysctl -p
reboot
```

---

## Step 7: Verify Disk

⚠️ **This will erase all data on the device**

```bash
lsblk
fdisk -l /dev/nvme0n2
wipefs -a /dev/nvme0n2
```

---

## Step 8: Create OST

```bash
mkfs.lustre \
  --fsname=hpcfs \
  --ost \
  --index=1 \
  --mgsnode=192.168.1.13@tcp \
  --reformat \
  /dev/nvme0n2
```

Expected output includes:

```
Target: hpcfs:OST0001
Mount type: ldiskfs
```

---

## Step 9: Mount OST

```bash
mkdir -p /OST
mount -t lustre /dev/nvme0n2 /OST
```

Verify:

```bash
df -Th | grep lustre
```

---

## Step 10: Validate OST Registration

```bash
lctl list_targets
lctl dl
```

The OST should appear as:

```
hpcfs-OST0001
```

---

## Step 11: Enable OST Mount at Boot

Edit `/etc/fstab`:

```fstab
/dev/nvme0n2  /OST  lustre  defaults,_netdev  0 0
```

Test:

```bash
umount /OST
mount -a
```

---

## Step 12: Verify from MGS/MDT Node

On the MGS or MDT server:

```bash
lfs df
```

Expected:

* OST0001 visible
* Space reported correctly

---

## Step 13: Troubleshooting Commands

```bash
dmesg | grep lustre
lctl list_targets
mount | grep lustre
lsmod | grep lustre
lsof /dev/nvme0n2
```

---

## Notes

* Do not mix ZFS and ldiskfs OSTs on the same node.
* Kernel updates require reinstalling `kmod-lustre`.
* Each OST must have a unique index.
* Multiple OSTs can be added per OSS.

---

# Lustre Client Installation Guide (EL9)

This document describes how to configure a **Lustre client** and mount an existing Lustre filesystem.

---

## Environment
- OS: RHEL 9 / Rocky Linux 9 / AlmaLinux 9
- Hostname: `client`
- Lustre Version: 2.16.x
- Filesystem Name: `hpcfs`
- MGS Address: `192.168.1.13@tcp`
- Mount Point: `/mnt/lustre`

---

## Step 1: Set Hostname

```
hostnamectl set-hostname client
```
```
reboot
```

Verify:

```
hostname
```

## Step 2: Update System

```
dnf clean all
dnf update -y
reboot
```

---
`
## Step 3: Install Lustre Repository

⚠️ The Lustre version **must match** MGS/MDT/OSS.

```bash
dnf install -y \
https://downloads.lustre.org/public/lustre/latest-release/el9/x86_64/lustre-release-2.16.0-1.el9.x86_64.rpm
```

Verify:

```bash
dnf repolist | grep lustre
```

---

## Step 4: Install Lustre Client Packages

```bash
dnf clean all
dnf makecache
dnf install -y \
  lustre-client \
  kmod-lustre-client
```

Verify:

```bash
rpm -qa | grep lustre
```
---

## Step 5: Load Lustre Kernel Module

```bash
modprobe lustre
```

Verify:

```bash
lsmod | grep lustre
ls /lib/modules/$(uname -r)/kernel/fs/lustre
dmesg | grep -i lustre
```

---

## Step 6: Bring Up LNet Network

```bash
lctl network up
lctl network list
```

Expected:

```
tcp
```

---

## Step 7: Disable SELinux and Firewall (Recommended)

```bash
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

systemctl disable --now firewalld
reboot
```

---

## Step 8: Create Mount Point

```bash
mkdir -p /mnt/lustre
```

---

## Step 9: Mount Lustre Filesystem

```bash
mount -t lustre 192.168.1.13@tcp:/hpcfs /mnt/lustre
```

Verify:

```bash
df -Th | grep lustre
ls /mnt/lustre
```

---

## Step 10: Mount at Boot

Edit `/etc/fstab`:

```fstab
192.168.1.13@tcp:/hpcfs  /mnt/lustre  lustre  defaults,_netdev  0 0
```

Test:

```bash
umount /mnt/lustre
mount -a
```

---

## Step 11: Enable Cockpit Web Console

Cockpit provides a web-based management interface.

```bash
dnf install -y cockpit
systemctl enable --now cockpit.socket
```

Access via browser:

```
https://<client-ip>:9090
```

---

## Step 12: Validation Commands

```bash
mount | grep lustre
lctl network list
lsmod | grep lustre
df -h
```

---

## Troubleshooting

```bash
dmesg | grep lustre
journalctl -xe
lctl dl
```

---

## Notes

* Do NOT install `lustre-server` packages on clients
* Kernel updates require matching `kmod-lustre-client`
* TCP works for testing; LNet + IB recommended for production

---