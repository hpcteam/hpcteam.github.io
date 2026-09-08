# Building a 2-Node OpenHPC Cluster in VMs: Rocky 9 + Warewulf4 + Slurm

A step-by-step account of building a small HPC cluster from scratch in VMware — one head node
(`sms`) and one diskless compute node (`c1`) — using OpenHPC 3.2, Warewulf4 for provisioning, and
Slurm as the job scheduler. This walks through every command run, plus every real problem hit
along the way and how it was diagnosed and fixed.

---

## What You'll Build

- A head node (**sms**) that provisions and manages the cluster
- A diskless compute node (**c1**) that PXE-boots its OS fresh every time, with no local disk install
- Slurm configured as the job scheduler across both nodes
- A working, tested setup that can run real distributed jobs

## Prerequisites

- A hypervisor supporting two virtual networks per VM (this guide uses VMware Workstation)
- Rocky Linux 9 ISO
- Basic Linux command-line familiarity
- At least 6 GB RAM combined across both VMs (more detail on why, below — this bit us)

---

## 1. Lab Layout

| Role | Hostname | vCPU / RAM | Disk | NICs |
|---|---|---|---|---|
| Head node (SMS) | `sms` | 2 vCPU / 4 GB+ | 20 GB | 2 (external NAT + internal) |
| Compute node | `c1` | 2 vCPU / **4–6 GB** | none (PXE boot, diskless) | 1 (internal only) |

**Networking**, in this build:

- `sms` external NIC (`ens192`): NAT, for internet access to pull packages
- `sms` internal NIC (`ens224`): static IP `169.254.67.50/16` — this is the provisioning network `c1` boots over
- `c1`'s single NIC: must be on the **same internal virtual network** as `sms`'s `ens224` — this
  turned out to be the single biggest source of early failures (see Troubleshooting below)

> **Note on the IP range used here:** `169.254.0.0/16` is technically the link-local/APIPA range.
> It works fine for an isolated internal lab network like this one, but if you're building something
> more permanent, a private range like `192.168.x.0/24` is the more conventional choice.

---

## 2. Base OS Setup on the Head Node

Install Rocky Linux 9 (minimal, no GUI needed) on `sms`, set a proper hostname, and disable
firewall/SELinux for the lab (re-enable and configure properly for anything beyond a lab):

```bash
hostnamectl set-hostname sms
systemctl disable --now firewalld
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config

# Set the internal-facing NIC's static IP
nmcli connection modify ens224 ipv4.addresses 169.254.67.50/16 ipv4.method manual autoconnect yes
nmcli connection up ens224
```

---

## 3. Enable the OpenHPC Repository, Install Base + Warewulf

```bash
dnf -y install http://repos.openhpc.community/OpenHPC/3/EL_9/x86_64/ohpc-release-3-1.el9.x86_64.rpm

dnf -y install dnf-plugins-core
dnf config-manager --set-enabled crb

dnf -y install ohpc-base warewulf-ohpc hwloc-ohpc
```

---

## 4. NTP Setup

`sms` acts as its own local time source (stratum 10) for the cluster, since this is an isolated
network with no external NTP reachable from the internal interface:

```bash
systemctl enable chronyd.service
echo "local stratum 10" >> /etc/chrony.conf
echo "server 169.254.67.50" >> /etc/chrony.conf
echo "allow all" >> /etc/chrony.conf
systemctl restart chronyd
```

---

## 5. Install and Configure the Slurm Server

```bash
dnf -y install ohpc-slurm-server

cp /etc/slurm/slurm.conf.ohpc /etc/slurm/slurm.conf
cp /etc/slurm/cgroup.conf.example /etc/slurm/cgroup.conf

perl -pi -e "s/SlurmctldHost=\S+/SlurmctldHost=sms/" /etc/slurm/slurm.conf
```

> **Heads up:** the default `slurm.conf.ohpc` template assumes a 4-node cluster (`c1`–`c4`), each
> with 16 threads. If your lab only has one compute node with different specs (very likely), you'll
> need to fix this later — see the Troubleshooting section, "Node hardware mismatch."

---

## 6. Complete Warewulf Setup on the Head Node

Bring up the internal interface for provisioning and wire it into Warewulf's config:

```bash
ip link set dev ens224 up
ip address add 169.254.67.50/16 broadcast + dev ens224

perl -pi -e "s/ipaddr:.*/ipaddr: 169.254.67.50/" /etc/warewulf/warewulf.conf
perl -pi -e "s/netmask:.*/netmask: 255.255.0.0/" /etc/warewulf/warewulf.conf
perl -pi -e "s/network:.*/network: 169.254.0.0/" /etc/warewulf/warewulf.conf
perl -pi -e 's/template:.*/template: static/' /etc/warewulf/warewulf.conf
sed '/range start:/d;/range end:/d;' -i /etc/warewulf/warewulf.conf

perl -pi -e "s/defaults,noauto,nofail,ro/defaults,nofail,ro/" /etc/warewulf/nodes.conf
```

Create a node profile and the overlay that will carry per-node config files:

```bash
wwctl profile add nodes --profile default --comment "Nodes profile"
wwctl overlay create nodeconfig
wwctl profile set --yes nodes --system-overlays nodeconfig --runtime-overlays syncuser

wwctl profile set -y nodes --netname=default --netdev=eth0
wwctl profile set -y nodes --netname=default --netmask=255.255.0.0
```

> No `--gateway` or `--nettagadd=DNS=...` was set here — this is an isolated network with no
> router, and `c1` resolves `sms` fine through the `/etc/hosts` overlay Warewulf manages
> automatically.

Update the hosts template and bring up all provisioning services:

```bash
wwctl overlay cat host /etc/hosts.ww | \
  sed -e 's_\({{$node.Id}}{{end}}\)_{{$node.Id}}.localdomain \1_g' | \
  EDITOR=tee wwctl overlay edit host /etc/hosts.ww

systemctl enable --now warewulfd
wwctl configure --all
bash /etc/profile.d/ssh_setup.sh
```

> **Gotcha:** `wwctl profile set` will fail with `ERROR: failed to reload warewulfd` if run
> *before* `warewulfd` is actually enabled and running. Run `wwctl configure --all` first if you
> hit this.

---

## 7. Build the Compute Node Image

Warewulf4 uses container images (not raw chroots) as the base filesystem for provisioning nodes.

**Import the base image** (needs internet access on the external NIC):

```bash
wwctl image import docker://ghcr.io/warewulf/warewulf-rockylinux:9 rocky-9 --syncuser
```

**Enter the image interactively** and install everything a compute node needs — OpenHPC's
compute meta-package, Slurm client, NTP, and the Lmod module system:

```bash
wwctl image shell rocky-9
```
```bash
# --- now inside the image ---
dnf -y install http://repos.openhpc.community/OpenHPC/3/EL_9/x86_64/ohpc-release-3-1.el9.x86_64.rpm
dnf -y update
dnf -y install ohpc-base-compute
dnf -y install ohpc-slurm-client
systemctl enable munge
systemctl enable slurmd
dnf -y install chrony
dnf -y install lmod-ohpc
exit
```

Exiting `wwctl image shell` automatically triggers a rebuild of the image.

---

## 8. Push Configuration Files Into the Image (Overlays)

These steps use Warewulf's "overlay" system to distribute config files — including the critical
shared authentication key — from `sms` down to every compute node.

**Support unprivileged containers (Apptainer/Singularity):**

```bash
wwctl overlay import --parents nodeconfig /etc/subuid
wwctl overlay import --parents nodeconfig /etc/subgid
```

**Tell compute nodes which NTP server to use**, via a templated tag rather than a static value
(so it applies automatically to any future node added to the `nodes` profile):

```bash
echo 'server {{.Tags.ntpserver}} iburst' | \
  wwctl overlay import --parents nodeconfig <(cat) /etc/chrony.conf.ww
wwctl profile set --yes nodes --tagadd ntpserver=169.254.67.50
```

**Make systemd wait for the network before starting dependent services** — important on a
PXE-booted node where services can otherwise race the network coming up:

```bash
cat <<- EOF | wwctl overlay import --parents nodeconfig <(cat) \
  /etc/systemd/system/NetworkManager-wait-online.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/nm-online -q
EOF
```

**Configure Slurm "configless" mode**, so `slurmd` fetches its config live from `sms` instead of
needing a static copy on every node:

```bash
echo SLURMD_OPTIONS='--conf-server {{.Tags.slurmctld}}' | \
  wwctl overlay import --parents nodeconfig <(cat) /etc/sysconfig/slurmd.ww
wwctl profile set --yes nodes --tagadd slurmctld=169.254.67.50
```

**Distribute the munge authentication key** — every node must have an *identical* copy of this
file, or Slurm authentication between nodes fails outright:

```bash
wwctl overlay import --parents nodeconfig /etc/munge/munge.key
wwctl overlay chown nodeconfig /etc/munge/munge.key $(id -u munge):$(id -g munge)
wwctl overlay chown nodeconfig /etc/munge $(id -u munge):$(id -g munge)
wwctl overlay chmod nodeconfig /etc/munge 0700
```

> `wwctl overlay chown` takes a single `uid:gid` argument, not two separate arguments — an easy
> mistake to make when adapting commands from elsewhere.

---

## 9. Build, Register, and Start Services

**Build the final image and overlays:**

```bash
wwctl image build rocky-9
wwctl overlay build
```

**Register `c1`** with its network details and the MAC address of its VM's NIC:

```bash
wwctl node add --image=rocky-9 --profile=nodes --netname=default \
  --ipaddr=169.254.67.11 --hwaddr=<C1_MAC_ADDRESS> c1

wwctl overlay build
wwctl configure --all
```

**Start munge and slurmctld on `sms`** — only after the node is registered and Warewulf is
reconfigured:

```bash
systemctl enable --now munge
systemctl enable --now slurmctld
```

---

## 10. Boot the Compute Node

With `c1` having no OS installed, boot it into its UEFI Boot Manager and choose **EFI Network**
(or the specific NIC entry, e.g. "EFI Network 1") to PXE boot.

If everything is wired correctly, the console will show, in order:

1. DHCP lease acquired
2. HTTP downloads of the compressed image, system overlay, and runtime overlay (Warewulf4 ships
   these over HTTP on port 9873, not classic TFTP, despite TFTP also being configured for the
   initial bootloader stage)
3. Kernel and initrd loading
4. A normal Linux boot sequence, ending at a `c1 login:` prompt

You don't need to log in at the console — the cluster is managed from `sms` via SSH and Slurm.

---

## 11. Verify Everything Works

From `sms`:

```bash
ssh root@169.254.67.11 hostname   # confirm SSH access
sinfo                              # confirm Slurm sees c1
srun -N1 -w c1 hostname            # run an actual scheduled job on c1
```

A healthy result looks like:

```
PARTITION  AVAIL  TIMELIMIT  NODES  STATE  NODELIST
normal*    up     infinite   1      idle   c1
```

---

## Troubleshooting: Real Problems Hit During This Build

These are, in the order encountered, the actual failures that came up — and how each was
diagnosed and resolved. If you hit the same messages, these are worth checking first.

### "No Media" on PXE boot / DHCPDISCOVER shows "no free leases"

**Cause:** the compute node's virtual NIC was on the wrong virtual network (initially set to
NAT instead of the internal network `sms`'s second NIC lives on).

**Fix:** in the hypervisor's VM settings, make sure the compute node's network adapter is on the
*exact same* internal/host-only virtual network (or LAN Segment) as the head node's internal NIC.
Also double check for a **second, unregistered NIC** on the compute VM generating constant
"no free leases" noise in the DHCP log — remove it, or ignore it if harmless.

### `EFI stub: ERROR: Failed to allocate memory for files`

**Cause:** insufficient VM RAM (2 GB was not enough) for UEFI firmware to stage the kernel,
initrd, and both compressed overlay images simultaneously before the OS takes over memory
management.

**Fix:** increase the compute node VM's memory. 2 GB failed; 4 GB got further; if you still see
`Initramfs unpacking failed: write error` followed by a kernel panic, increase again (6 GB is a
safe, comfortable amount for this size of lab image).

### `Initramfs unpacking failed: write error` → kernel panic

**Cause:** same root cause as above — the entire root filesystem is unpacked into RAM (tmpfs) at
boot on a diskless node, so this is really just the memory shortage manifesting slightly further
into the boot process.

**Fix:** more RAM, as above. Also worth doing a clean `wwctl image build` + `wwctl overlay build`
afterward in case a prior failed attempt left a stale or partial build artifact.

### `ssh: Host key verification failed`

**Cause:** `sms` had a stale cached SSH host key for the compute node's IP from an earlier, failed
boot attempt (a different, broken image presenting a different host key).

**Fix:**
```bash
ssh-keygen -R 169.254.67.11
ssh root@169.254.67.11 hostname
```

### `slurmd`: `_fetch_child: failed to fetch remote configs: Protocol authentication error`

**Cause:** this looked like a munge/authentication problem, but the real cause was **clock skew**
between `sms` and `c1` — over 5 hours apart in this case. Munge rejects credentials outside its
default 300-second time-to-live window, so any noticeable clock drift between nodes presents as
an authentication failure, not an obviously time-related one.

**Diagnosis:** test munge directly, independent of Slurm:
```bash
munge -n | ssh sms unmunge
```
A `STATUS: Rewound credential` result (rather than `Success`) with a large gap between
`ENCODE_TIME` and `DECODE_TIME` confirms clock skew, not a key mismatch.

**Fix:**
```bash
systemctl restart chronyd
chronyc makestep
timedatectl   # confirm "System clock synchronized: yes" on both nodes
```

### `sinfo` shows node state `inval`

**Cause:** the default `slurm.conf.ohpc` hardware definition (`Sockets=2 CoresPerSocket=8
ThreadsPerCore=2`, i.e. a 16-thread node) didn't match the compute VM's actual, much smaller
allocation.

**Diagnosis:** ask `slurmd` what it thinks its own hardware actually is:
```bash
slurmd -C
```
Compare that output directly against the `NodeName=...` line in `slurm.conf`.

**Fix:** edit `slurm.conf` to match reality, then restart both sides:
```bash
perl -pi -e 's/NodeName=c\[1-4\].*/NodeName=c1 Sockets=2 CoresPerSocket=1 ThreadsPerCore=1 RealMemory=5950 State=UNKNOWN/' /etc/slurm/slurm.conf
perl -pi -e 's/Nodes=c\[1-4\]/Nodes=c1/' /etc/slurm/slurm.conf

systemctl restart slurmctld
scontrol reconfigure
```
Then on the compute node:
```bash
systemctl restart slurmd
```

---

## What's Next

With a working scheduler and a compute node that takes jobs, the natural next steps are:

- **Install compilers and MPI** (`gnu14-compilers-ohpc`, `openmpi5-*-ohpc`, etc.) so real
  parallel/distributed jobs can run, not just single commands like `hostname`
- **Add shared storage** — NFS-export `/home` and `/opt/ohpc/pub` from the head node so user
  files and the software stack are visible identically on every compute node
- **Register additional compute nodes** — repeat the image registration step with new node names,
  MAC addresses, and IPs against the same `rocky-9` image and `nodes` profile
