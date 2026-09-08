# BeeGFS Lab Setup Guide (5–6 VMs)

BeeGFS is a parallel cluster file system made of independent services that talk to each other over the network. For learning, you can run every service on its own small VM. This guide walks through a 6-VM lab.

## 1. Architecture Overview

| Node | Hostname | Role | BeeGFS service(s) |
|---|---|---|---|
| VM1 | beegfs-mgmt | Management node | `beegfs-mgmtd` |
| VM2 | beegfs-meta1 | Metadata node | `beegfs-meta` |
| VM3 | beegfs-storage1 | Storage node 1 | `beegfs-storage` |
| VM4 | beegfs-storage2 | Storage node 2 | `beegfs-storage` |
| VM5 | beegfs-client1 | Client | `beegfs-client`, `beegfs-helperd` |
| VM6 | beegfs-client2 (optional) | 2nd client / Admon | `beegfs-client` or `beegfs-admon` |

Why this layout: BeeGFS separates metadata (file names, directory structure, permissions) from actual file data. Having two storage nodes lets you see how data gets striped across targets, and a second client lets you confirm the file system behaves like a real shared FS.

If you only want 5 VMs, drop VM4 (single storage node) — everything still works, you just won't see striping across multiple targets.

## 2. VM Specs and OS Choice

For a lab, each VM can be very light:
- **CPU:** 1–2 vCPU
- **RAM:** 1–2 GB (mgmt/meta), 1 GB (storage), 1 GB (client)
- **Disk:** 1 small OS disk (10–15 GB) + 1 extra virtual disk for storage/meta nodes (5–10 GB each, can be thin-provisioned)

**OS:** Use Rocky Linux 9 or Ubuntu 22.04/24.04 — both have official BeeGFS repos. This guide shows both `dnf` (Rocky/RHEL) and `apt` (Ubuntu) commands.

In your hypervisor (VirtualBox, VMware, or KVM/libvirt):
- Create one **internal/host-only network** so all VMs can talk to each other and to your laptop.
- Optionally add a NAT/bridged adapter on each VM for internet access (needed to install packages).

## 3. Network Setup

Assign static IPs on the internal network, e.g.:

```
192.168.56.10  beegfs-mgmt
192.168.56.11  beegfs-meta1
192.168.56.12  beegfs-storage1
192.168.56.13  beegfs-storage2
192.168.56.14  beegfs-client1
192.168.56.15  beegfs-client2
```

On **every VM**, edit `/etc/hosts` and add all six entries above. Then set each VM's own hostname:

```bash
sudo hostnamectl set-hostname beegfs-mgmt   # change per node
```

Test connectivity between all nodes with `ping` before going further.

Disable firewalls for the lab (or open the BeeGFS ports: 8008 mgmtd, 8003 storage, 8004 client, 8005 meta, 8006 helperd, 8007 admon — TCP+UDP):

```bash
# Rocky/RHEL
sudo systemctl disable --now firewalld

# Ubuntu
sudo ufw disable
```

## 4. Add the BeeGFS Repository (all relevant nodes)

BeeGFS publishes per-distro repo files. The exact URL depends on the BeeGFS version (this guide assumes the current 8.x series). Visit the BeeGFS download page on doc.beegfs.io to grab the repo file matching your OS/version, then on each node:

**Rocky/RHEL 9:**
```bash
curl -o /etc/yum.repos.d/beegfs-rhel9.repo \
  https://www.beegfs.io/release/beegfs_8/dists/beegfs-rhel9.repo
sudo dnf clean all
```

**Ubuntu 22.04/24.04:**
```bash
curl -fsSL https://www.beegfs.io/release/beegfs_8/gpg/GPG-KEY-beegfs \
  | sudo gpg --dearmor -o /usr/share/keyrings/beegfs.gpg

echo "deb [signed-by=/usr/share/keyrings/beegfs.gpg] https://www.beegfs.io/release/beegfs_8/dists/beegfs-$(lsb_release -cs).list \
  | sudo tee /etc/apt/sources.list.d/beegfs.list
sudo apt update
```

(If those exact paths 404, just go to the official download page and copy the link for your distro/version — the repo setup is a one-time `curl`/`apt-add-repository` step they provide directly.)

## 5. Install Packages per Node

**VM1 — Management (beegfs-mgmt):**
```bash
sudo dnf install -y beegfs-mgmtd       # Rocky
sudo apt install -y beegfs-mgmtd       # Ubuntu
```

**VM2 — Metadata (beegfs-meta1):**
```bash
sudo dnf install -y beegfs-meta
```

**VM3 & VM4 — Storage (beegfs-storage1/2):**
```bash
sudo dnf install -y beegfs-storage
```

**VM5 & VM6 — Client (beegfs-client1/2):**
```bash
sudo dnf install -y beegfs-client beegfs-helperd beegfs-utils beegfs-tools
```

The client package needs to build a kernel module, so make sure kernel headers and build tools are present first:

```bash
# Rocky
sudo dnf install -y kernel-devel-$(uname -r) gcc make elfutils-libelf-devel

# Ubuntu
sudo apt install -y linux-headers-$(uname -r) build-essential
```

## 6. Prepare Storage Directories

On each node that needs a data directory, create (and ideally mount a separate disk at) a target directory:

```bash
# On meta node
sudo mkdir -p /data/beegfs_meta

# On each storage node
sudo mkdir -p /data/beegfs_storage

# On mgmt node
sudo mkdir -p /data/beegfs_mgmtd
```

If you attached an extra virtual disk, format and mount it (e.g. `/dev/sdb`) to these paths instead of using the root filesystem — closer to a real deployment and avoids filling your root disk.

```bash
sudo mkfs.xfs /dev/sdb
sudo mount /dev/sdb /data/beegfs_storage
echo "/dev/sdb /data/beegfs_storage xfs defaults 0 0" | sudo tee -a /etc/fstab
```

## 7. Run beegfs-setup-* Scripts

BeeGFS provides helper scripts (`beegfs-setup-<service>`) that initialize the data directories and register each node with the management daemon. **Always set up the management node first.**

**On VM1 (mgmt):**
```bash
sudo beegfs-setup-mgmtd -p /data/beegfs_mgmtd
sudo systemctl enable --now beegfs-mgmtd
```

**On VM2 (meta):**
```bash
sudo beegfs-setup-meta -p /data/beegfs_meta -s 2 -m beegfs-mgmt
sudo systemctl enable --now beegfs-meta
```
(`-s` is a unique numeric "node ID" for this service type — pick small distinct numbers per node, e.g. meta=2, storage1=3, storage2=4. `-m` points to the management host.)

**On VM3 (storage1):**
```bash
sudo beegfs-setup-storage -p /data/beegfs_storage -s 3 -i 301 -m beegfs-mgmt
sudo systemctl enable --now beegfs-storage
```

**On VM4 (storage2):**
```bash
sudo beegfs-setup-storage -p /data/beegfs_storage -s 4 -i 401 -m beegfs-mgmt
sudo systemctl enable --now beegfs-storage
```
(`-i` is the storage **target ID**, must be unique cluster-wide; `-s` the node ID.)

**On VM5/VM6 (clients):**
```bash
sudo beegfs-setup-client -m beegfs-mgmt
```

This writes `/etc/beegfs/beegfs-mounts.conf` and configures the client kernel module. Then start the client:
```bash
sudo systemctl enable --now beegfs-helperd
sudo systemctl enable --now beegfs-client
```

By default this mounts BeeGFS at `/mnt/beegfs` on the client.

## 8. Connection Authentication (recommended even for a lab)

BeeGFS supports a shared-secret auth file so only trusted nodes can join the cluster. Generate one on the mgmt node and copy it to every node:

```bash
# on mgmt
sudo dd if=/dev/random of=/etc/beegfs/connauthfile bs=128 count=1

# copy to all other nodes
sudo scp /etc/beegfs/connauthfile root@beegfs-meta1:/etc/beegfs/
sudo scp /etc/beegfs/connauthfile root@beegfs-storage1:/etc/beegfs/
sudo scp /etc/beegfs/connauthfile root@beegfs-storage2:/etc/beegfs/
sudo scp /etc/beegfs/connauthfile root@beegfs-client1:/etc/beegfs/
```

Then add `connAuthFile = /etc/beegfs/connauthfile` to each service's config file in `/etc/beegfs/beegfs-<service>.conf`, and restart all services.

## 9. Verify the Cluster

From any node with `beegfs-tools` installed (client nodes have it):

```bash
beegfs-check-servers              # checks reachability of mgmt, meta, storage nodes
beegfs-df                         # shows capacity per storage target and inode usage per meta node
beegfs-net                        # shows connections between nodes
```

On the client, confirm the mount:
```bash
df -h /mnt/beegfs
mount | grep beegfs
```

## 10. Test It

```bash
cd /mnt/beegfs
echo "hello beegfs" > test.txt
cat test.txt

# Check striping/layout info for a file
beegfs-ctl --getentryinfo /mnt/beegfs/test.txt

# Write a bigger file and watch which storage targets get used
dd if=/dev/zero of=/mnt/beegfs/bigfile bs=1M count=200
beegfs-ctl --getentryinfo /mnt/beegfs/bigfile
```

From VM6 (second client), mount the same way and confirm you can see files created from VM5 — this demonstrates the shared namespace.

## 11. Things to Experiment With Next

- **Striping**: use `beegfs-ctl --setpattern` on a directory to change chunk size / number of targets a file is striped across, then write files and inspect with `--getentryinfo`.
- **Buddy mirroring**: with 2 storage nodes you can configure mirror groups (`beegfs-ctl --addmirrorgroup`) for storage redundancy, and 2 meta nodes for metadata mirroring (would need a 3rd metadata VM, or repurpose VM6).
- **Failure simulation**: stop a storage service and observe behavior with `beegfs-check-servers` / `beegfs-df`.
- **Quotas and ACLs**: BeeGFS supports user/group quotas — good for understanding multi-tenant HPC storage.
- **Admon (monitoring)**: install `beegfs-admon` on VM6 instead of a second client, and connect with the BeeGFS GUI for graphical monitoring.

## 12. Common Pitfalls

- **Hostname resolution**: every node must resolve every other node's hostname (via `/etc/hosts` or DNS) — BeeGFS is picky about this.
- **Time sync**: install `chrony`/`ntp` on all VMs; clock drift causes odd auth/connection errors.
- **Client kernel module build fails**: usually missing kernel headers matching `uname -r` exactly, especially after a kernel update — reinstall headers and re-run `dkms` or rebuild.
- **Node IDs/Target IDs must be unique** across the whole cluster, not just per node type.
- **Firewall**: if you re-enable firewalls later, open TCP+UDP 8003–8008.

---

This setup gives you a fully functional, if tiny, BeeGFS cluster — enough to learn the architecture, configuration files (`/etc/beegfs/*.conf`), `beegfs-ctl` administration commands, and striping/mirroring concepts that scale directly to production-sized deployments.
