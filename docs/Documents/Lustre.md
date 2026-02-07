
# Lustre Distributed File System – Overview & Installation Guide

## What is Lustre?

**Lustre** is a high-performance, distributed (parallel) file system mainly used in:

- HPC clusters (scientific simulations, CFD, weather, AI/ML)
- Supercomputers
- Large research data centers
- GPU clusters and big data workloads

### Key Ideas

- Compute nodes do the calculations
- Lustre stores and serves data to all nodes at very high speed
- Many nodes can read/write the **same file at the same time**
- Lustre separates **metadata** from **file data** for extreme performance

---

## Lustre Architecture Overview

### Main Components

| Term | Meaning |
|-----|--------|
| **MDS** | Metadata Server |
| **MDT** | Metadata Target (disk for metadata) |
| **OSS** | Object Storage Server |
| **OST** | Object Storage Target (data disks) |
| **Client** | Compute node using Lustre |
| **Striping** | Splitting a file across many OSTs |
| **Object** | A chunk of a file |
| **LNet** | Network layer used by Lustre |
| **Parallel I/O** | Many nodes read/write simultaneously |

---

## Metadata Server (MDS)

The **MDS** is like the librarian of the Lustre filesystem.

- Stores **file information only**
- Does **NOT** store file data

### Metadata Includes

- File name
- Directory structure
- Owner and group
- Permissions (ACLs, mode bits)
- File size
- Layout information (OST striping)

Metadata is stored on the **MDT (Metadata Target)**.

---

## Metadata Operation Flow

```

Client → MDS → MDT → MDS → Client

```

### Step-by-Step

1. **Client → MDS**
   - Requests: create, open, stat, permission check

2. **MDS → MDT**
   - Queries or updates metadata

3. **MDT**
   - Stores:
     - File names
     - Directory hierarchy
     - Permissions
     - Owners
     - Timestamps (atime, mtime, ctime)
     - File layout (OST striping)

4. **MDT → MDS**
   - Returns metadata or confirms updates

5. **MDS Processing**
   - Permission checks
   - Metadata locks
   - Consistency management

6. **MDS → Client**
   - Sends final result

---

## Object Storage Server (OSS)

**OSS = Object Storage Server**

- A **server (machine)**
- Handles **read/write data requests**
- Manages one or more OSTs

---

## Object Storage Target (OST)

**OST = Object Storage Target**

- The **actual storage** (disk or disk group)
- Stores the real file data (objects)

### Relationship

```

OSS
├── OST1
├── OST2
└── OST3

```

- OSS = controls access
- OST = stores data

---

## How OSS and OST Work Together

1. Client wants to read/write a file
2. Client asks MDS:
   - “Where is the data stored?”
3. MDS replies:
   - “OST1 and OST2”
4. Client talks **directly** to OSS
5. OSS accesses OST disks
6. Data flows back to client

```

Client ⇄ OSS ⇄ OST

````

---

## Why Separate Metadata and Data?

### Performance Benefits

- Multiple MDS/OSS servers
- Many OST disks
- Thousands of clients

This enables:

- Parallel data access
- Massive throughput
- Low latency

---

## Real-Life Analogy

- **MDS** → Reception desk
- **OSS/OST** → Warehouse
- **Client** → Customer

The receptionist never touches the package — only tells you where it is.

---

# Lustre Server Installation Guide (EL9 / RHEL9)

## MGS + MDT using `ldiskfs`

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

```bash
dnf clean all
dnf update -y
reboot
````

---

## Step 2: Install Kernel Headers and Build Tools

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

```bash
dnf install -y \
https://downloads.lustre.org/public/lustre/latest-release/el9/x86_64/lustre-release-2.16.0-1.el9.x86_64.rpm
```

---

## Step 4: Install Lustre Server Packages

```bash
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

---

## Step 6: Create MGS + MDT

⚠️ **This destroys all data on the device**

```bash
mkfs.lustre \
  --fsname=hpcfs \
  --mgs \
  --mdt \
  --index=0 \
  --reformat \
  /dev/nvme0n2
```

---

## Step 7: Mount MDT

```bash
mkdir -p /lustre01
mount -t lustre /dev/nvme0n2 /lustre01
```

---

## Step 8: Validate

```bash
lctl dl
lctl status
lfs df
```

---

# Lustre OSS (OST Server) Installation Guide – EL9

## Environment

* Hostname: `OSS`
* Filesystem: `hpcfs`
* MGS IP: `192.168.1.13`
* Network: TCP
* OST Device: `/dev/nvme0n2`
* Mount Point: `/OST`

---

## Create OST

```bash
mkfs.lustre \
  --fsname=hpcfs \
  --ost \
  --index=1 \
  --mgsnode=192.168.1.13@tcp \
  --reformat \
  /dev/nvme0n2
```

---

## Mount OST

```bash
mkdir -p /OST
mount -t lustre /dev/nvme0n2 /OST
```

---

## Verify OST

```bash
lctl list_targets
lfs df
```

---

## Notes

* Do not mix Lustre versions
* Kernel updates require matching `kmod-lustre`
* Each OST must have a unique index
* Use separate devices for MDT and OSTs in production

---

## Next Steps

* Configure Lustre clients
* Add more OSTs
* Enable HA (Pacemaker + Corosync)
* LNet & network tuning

