#📘 LVM (Logical Volume Manager)

---

## What is LVM?

**LVM (Logical Volume Manager)** is a storage management system in Linux that adds an **abstraction layer** between physical storage devices and filesystems.

Its main purpose is to make disk storage:

* Flexible
* Expandable
* Easier to manage

Instead of working directly with fixed disk partitions, LVM allows administrators to work with **logical storage units** that can be resized and reorganized easily.

---

## Why LVM Is Needed

Traditional disk partitioning has limitations:

* Partitions have fixed sizes
* Expanding storage often requires downtime
* Adding new disks is difficult
* Disk space cannot be easily shared

LVM overcomes these limitations by separating:

* **Physical storage** from
* **Logical storage**

This separation allows dynamic storage management.

---

## LVM Storage Layers

LVM is built using multiple logical layers. Each layer has a specific role.

```
Physical Disk → Physical Volume → Volume Group → Logical Volume → Filesystem → Mount Point
```

---

## 1. Physical Disk

A **physical disk** is a real storage device such as:

* Hard disk
* SSD
* Virtual disk

It provides raw storage capacity but has no understanding of files or directories.

---

## 2. Partition (Optional Layer)

A **partition** is a defined section of a physical disk.

Although LVM can work directly on entire disks, partitions are often used for:

* Organization
* Compatibility
* Disk layout clarity

Partitions act as containers that can later be managed by LVM.

---

## 3. Physical Volume (PV)

A **Physical Volume (PV)** is a disk or partition that has been initialized for use by LVM.

When a PV is created:

* LVM writes metadata on it
* The space is divided into equal-sized blocks called **Physical Extents**

A PV is the **basic building block** of LVM.

---

## 4. Volume Group (VG)

A **Volume Group (VG)** is a storage pool created by combining one or more Physical Volumes.

Key characteristics:

* Aggregates total storage capacity
* Can grow by adding more PVs
* Acts as a single large storage unit

The VG hides the complexity of multiple disks.

---

## 5. Logical Volume (LV)

A **Logical Volume (LV)** is a virtual storage unit created from a Volume Group.

Characteristics:

* Acts like a traditional disk partition
* Can be resized
* Can span multiple physical disks
* Used to store filesystems

Logical Volumes are the main objects users interact with in LVM.

---

## 6. Filesystem

A **filesystem** defines how data is stored and organized on a Logical Volume.

Responsibilities:

* Manages files and directories
* Handles permissions and metadata
* Controls space allocation

Common filesystems include ext4 and XFS.

---

## 7. Mount Point

A **mount point** is a directory in the Linux filesystem hierarchy where a filesystem is attached.

Once mounted:

* Data on the Logical Volume becomes accessible
* The filesystem integrates into the system’s directory tree

---

## Internal LVM Concepts 

### Extents

* **Physical Extents (PE):** Fixed-size blocks on a Physical Volume
* **Logical Extents (LE):** Mapped blocks used by Logical Volumes

LVM maps LEs to PEs dynamically, enabling flexibility and resizing.

---

## Key Advantages of LVM

* Dynamic resizing of storage
* Ability to add disks without downtime
* Improved storage utilization
* Support for snapshots
* Better abstraction of physical hardware

---

## Summary 

* LVM separates physical storage from logical usage
* Storage is managed in layers
* Logical Volumes provide flexibility over traditional partitions
* Filesystems operate on LVM, not directly on disks

---

## **Create and Mount an LVM Volume on a New Disk**

## Attach a Virtual Disk to Your Hyper-V VM

1. Shut down the VM.
2. Open Hyper-V Manager.
3. Right-click on the VM and select **Settings**.
4. Under **Hardware**, select **SCSI Controller** (or add one if it's not there already).
5. Click **Add** > **Hard Drive** > **New**.
6. Choose VHDX or VHD, set the size for the new virtual disk, and click **OK**.
7. Start the VM.

## LVM Setup Steps

### 1. Verify the New Disk

Check if the new disk is visible to the system:
```
lsblk
fdisk -l
```

**Example output:**
```
sdb       8:16   0  100G  0 disk
```

### 2. Create a New Partition

Use `fdisk` to create a Linux partition on `/dev/sdb`:

```bash
fdisk /dev/sdb
```

Inside `fdisk`, perform the following:
- Press `n` → create a new partition
- Press `p` → primary partition
- Accept defaults for sector unless specific sector sizes are needed
- Define size, e.g.: `+95G`
- Press `w` → write changes and exit

Verify the new partition:
```bash
fdisk -l /dev/sdb
```

You should now have `/dev/sdb1`.

### 3. Create a Physical Volume (PV)

```bash
pvcreate /dev/sdb1
```

Confirm:
```bash
pvs
pvsdisplay
```

### 4. Create a Volume Group (VG)

```bash
vgcreate earth_volume_group /dev/sdb1
```

Check VG details:
```bash
vgdisplay
```

### 5. Create a Logical Volume (LV)

Allocate 90 GB for the new logical volume:

```bash
lvcreate -L 90G -n my_logical_volume earth_volume_group
```

List LVs:
```bash
lvs
```

Confirm LV path:
```
/dev/earth_volume_group/my_logical_volume
```

### 6. Create a Filesystem on the LV

Format the logical volume with the `ext4` filesystem:

```bash
mkfs.ext4 /dev/earth_volume_group/my_logical_volume
```

### 7. Create a Mount Directory

```bash
mkdir -p /HOME
```

### 8. Mount the New Filesystem

```bash
mount /dev/earth_volume_group/my_logical_volume /HOME
```

### 9. Verify the Mount

```bash
df -Th
```

**Expected output:**
```
/dev/mapper/earth_volume_group-my_logical_volume  ext4  89G   24K  84G  1% /HOME
```

### 10. Make the Mount Persistent

**Before editing `/etc/fstab`, always back it up:**
```bash
cp /etc/fstab /etc/fstab.bak
```

Get the UUID of the newly formatted LV:
```bash
blkid /dev/earth_volume_group/my_logical_volume
```

**Example output:**
```
/dev/earth_volume_group/my_logical_volume: UUID="6a62873b-95f8-4252-b73d-82daa7378b11" TYPE="ext4"
```

Copy the UUID shown for the device. Edit `/etc/fstab`:
```bash
vi /etc/fstab
```

Add this line at the end (replace `<UUID>` with the actual UUID):
```
UUID=<UUID>  /HOME  ext4  defaults  0  0
```

### 11. Test the fstab Entry

Unmount and remount to ensure the entry works:
```bash
umount /HOME
mount -a
df -Th
```

You should see `/HOME` mounted automatically.

## LVM Structure Diagram

```
[Physical Disk: /dev/sdb]
          │
          ▼
[Partition: /dev/sdb1] ← we have to create
          │
          ▼
[Physical Volume (PV): /dev/sdb1] ← we have to create
          │
          ▼
[Volume Group (VG): earth_volume_group] ← we have to create
          │
          ▼
[Logical Volume (LV): my_logical_volume] ← we have to create
          │
          ▼
[Filesystem: ext4]
          │
          ▼
[Mount Point: /HOME]
```

## LVM Components Summary

- **Physical Disk**: The entire disk device.
- **Partition**: Disk section created with `fdisk`, e.g., `/dev/sdb1`.
- **Physical Volume (PV)**: LVM-initialized partition, e.g., after `pvcreate /dev/sdb1`.
- **Volume Group (VG)**: Pool of storage created by combining PV(s), e.g., `earth_volume_group`.
- **Logical Volume (LV)**: Virtual disk partition allocated from VG, e.g., `my_logical_volume`.
- **Filesystem**: Format the LV with a filesystem like ext4.
- **Mount Point**: Directory to access the data, e.g., `/HOME`.
```

***
