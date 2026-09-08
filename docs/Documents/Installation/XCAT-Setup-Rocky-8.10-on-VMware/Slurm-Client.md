# Slurm Compute Node Setup — Standard Operating Procedure

**Adding a New Compute Node to an Existing xCAT + Slurm Cluster**

Rocky Linux 8.10 | xCAT | OpenHPC 2.x | Munge | Slurm 25.05

> *Self-contained: every command needed is in this document. No prior context required.*

---

## 1. What This Document Covers

This is the complete, standalone procedure for bringing a new compute node into an already-running xCAT-managed cluster with a working Slurm controller. Follow every step in order — do not skip ahead, since later steps depend on files created in earlier ones.

By the end, the new node will:

- Be defined in xCAT and provisioned with Rocky Linux 8.10
- Trust the cluster's shared Munge authentication key
- Have the OpenHPC/EPEL yum repositories and their GPG keys available
- Have Slurm's client packages (ohpc-slurm-client) installed
- Run slurmd, registered correctly against the existing controller with accurate hardware specs
- Be visible as an idle, usable node in `sinfo` and able to run real jobs via `srun`/`sbatch`

## 2. Assumptions — Read Before Starting

This procedure assumes the following already exist on the master/controller node. If any of these are missing, complete them first (they are one-time, cluster-level setup steps, not per-node steps):

- The master node's hostname, static IP, and xCAT installation are already working.
- Munge is installed, a key has been generated with `create-munge-key`, and munged is running on the master.
- The OpenHPC 2.x repo (ohpc-release) and EPEL are installed and enabled on the master.
- ohpc-slurm-server is installed on the master, slurm.conf exists at `/etc/slurm/slurm.conf`, and slurmctld is running.
- xCAT postscripts are served over HTTP from the master automatically — no extra web server setup is needed.

| Field | Example value used in this document |
|---|---|
| Master / controller hostname | labtesting (192.168.245.128) |
| New compute node name | cnode01 |
| New compute node IP | 192.168.245.10 |
| New compute node MAC | 00:0C:29:88:E7:00 |
| xCAT compute group | compute |
| OS image | rocky8.10-x86_64-install-compute |

Replace these example values with your actual node name, IP, and MAC throughout this document.

---

## Phase 1 — Define the Node in xCAT

Skip this phase if the node is already defined and provisioned (e.g. you are re-running Slurm setup on an existing node). Otherwise, run every step below.

### 1.1 Define the node

```bash
mkdef -t node cnode01 groups=all,compute \
  mac=00:0C:29:88:E7:00 \
  ip=192.168.245.10 \
  netboot=xnba \
  os=rocky8.10 arch=x86_64 profile=compute \
  installnic=mac primarynic=mac
```

**Why this step matters** Registers the node's identity (MAC, IP, OS image) in xCAT's database. `installnic=mac` and `primarynic=mac` tell xCAT to identify the node by MAC address rather than a specific NIC name, which is more portable across hardware.

**How to verify** `lsdef cnode01` — confirm the attributes above are listed back correctly.

### 1.2 Publish the node into hosts, DNS, and DHCP

```bash
makehosts cnode01
makedns -n
makedhcp cnode01
```

**Why this step matters** A node definition alone does not make the node reachable — these commands push the definition into the actual DNS/DHCP services so the node can be found on the network and receive the correct IP at boot.

**How to verify** `grep cnode01 /etc/hosts` should show the correct hostname/IP mapping.

### 1.3 Stage the OS install and boot the node

```bash
nodeset cnode01 osimage=rocky8.10-x86_64-install-compute
# Power on the node and force network/PXE boot
# (VM: via console; real hardware: rpower cnode01 boot)
```

**Why this step matters** This tells xCAT which OS image to install the next time this node's MAC PXE-boots, completing the base OS provisioning before any cluster software is layered on.

**How to verify** Once the node finishes installing and reboots, confirm you can log in: `ssh root@cnode01`

---

## Phase 2 — Prepare Files on the Master for Distribution

These files must exist on the master, in a location served over HTTP by xCAT, before any compute-node postscript can fetch them. Run this phase once per master node setup — if these files already exist from a previous node, skip to Phase 3.

### 2.1 Create the shared files directory

```bash
mkdir -p /install/postscripts/files
```

**Why this step matters** xCAT automatically serves everything under `/install/postscripts` over HTTP on port 80 — this subdirectory is where files a postscript will curl down should live.

### 2.2 Copy the Munge key

```bash
cp /etc/munge/munge.key /install/postscripts/files/munge.key
```

**Why this step matters** Every node must share the exact same Munge key. This makes the master's key available for compute nodes to fetch.

### 2.3 Copy the yum repo definition files

```bash
cp /etc/yum.repos.d/OpenHPC.repo /install/postscripts/files/
cp /etc/yum.repos.d/epel.repo /install/postscripts/files/
cp /etc/yum.repos.d/epel-modular.repo /install/postscripts/files/
```

**Why this step matters** Compute nodes need these .repo files to know where to download OpenHPC/Slurm/EPEL packages from — they are not present on a freshly-provisioned node by default.

### 2.4 Copy the GPG keys the repos reference

```bash
cp /etc/pki/rpm-gpg/RPM-GPG-KEY-OpenHPC-2 /install/postscripts/files/
cp /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8 /install/postscripts/files/
```

**Why this step matters** Each .repo file references a local GPG key file path for package signature verification. Copying only the .repo file is not enough — dnf will fail with a "Couldn't open file" error on the compute node unless the actual key file is present at that same path.

> **Note:** Confirm the exact key filenames on your master first with: `ls /etc/pki/rpm-gpg/` — names can vary slightly by OS/repo version. Copy every key referenced by `gpgkey=` in each .repo file.

### 2.5 Copy the cgroup configuration

```bash
cp /etc/slurm/cgroup.conf.example /etc/slurm/cgroup.conf
cp /etc/slurm/cgroup.conf /install/postscripts/files/cgroup.conf
```

**Why this step matters** Without a cgroup.conf, slurmd starts with a warning ("No cgroup.conf file, using defaults") and falls back to built-in defaults for process tracking. Providing an explicit file — copied from OpenHPC's own example — avoids relying on undocumented defaults and keeps controller and compute node behavior consistent.

**How to verify** `cat /etc/slurm/cgroup.conf` should show OpenHPC's example content, not be empty.

### 2.6 Copy the current slurm.conf

```bash
cp /etc/slurm/slurm.conf /install/postscripts/files/slurm.conf
```

**Why this step matters** The controller and every compute node must run from a byte-identical slurm.conf. This is the file every compute node will fetch.

> **Important:** repeat this single command (2.6) every single time slurm.conf is edited on the master, even outside of adding a new node. A stale copy here is the single most common cause of nodes showing "inval" or "down" — the controller and the node silently disagree on hardware or config. Consider aliasing this command (see Section 7).

---

## Phase 3 — Create the Postscripts (one-time, reusable for every future node)

Skip this phase entirely if these postscripts already exist from a previous node setup — go straight to Phase 4.

### 3.1 Repo + GPG key sync postscript

```bash
cat > /install/postscripts/setup_repos << 'EOF'
#!/bin/bash
MASTER_IP=$(getent hosts $MASTER | awk '{print $1}')
curl -s -o /etc/yum.repos.d/OpenHPC.repo http://${MASTER_IP}/install/postscripts/files/OpenHPC.repo
curl -s -o /etc/yum.repos.d/epel.repo http://${MASTER_IP}/install/postscripts/files/epel.repo
curl -s -o /etc/yum.repos.d/epel-modular.repo http://${MASTER_IP}/install/postscripts/files/epel-modular.repo
curl -s -o /etc/pki/rpm-gpg/RPM-GPG-KEY-OpenHPC-2 http://${MASTER_IP}/install/postscripts/files/RPM-GPG-KEY-OpenHPC-2
curl -s -o /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8 http://${MASTER_IP}/install/postscripts/files/RPM-GPG-KEY-EPEL-8
dnf clean all
echo "repos and GPG keys synced from ${MASTER_IP}"
EOF
chmod +x /install/postscripts/setup_repos
```

**Why this step matters** Separating repo/key sync into its own postscript, run before any package install, guarantees dnf can verify and install OpenHPC/Slurm/EPEL packages without signature or "repo not found" errors.

### 3.2 Munge postscript

```bash
cat > /install/postscripts/setup_munge << 'EOF'
#!/bin/bash
dnf install -y munge munge-libs
MASTER_IP=$(getent hosts $MASTER | awk '{print $1}')
curl -s -o /etc/munge/munge.key http://${MASTER_IP}/install/postscripts/files/munge.key
chown munge:munge /etc/munge/munge.key
chmod 400 /etc/munge/munge.key
systemctl restart munge
systemctl enable munge
echo "munge configured from master key"
EOF
chmod +x /install/postscripts/setup_munge
```

**Why this step matters** Fetches and installs the shared Munge key, then explicitly RESTARTS (not just enables) the munge daemon — munged only reads its key file once at startup, so a restart is required for a freshly-copied key to actually take effect.

### 3.3 Slurm client postscript

```bash
cat > /install/postscripts/setup_slurm_client << 'EOF'
#!/bin/bash
MASTER_IP=$(getent hosts $MASTER | awk '{print $1}')
if ! dnf repolist 2>/dev/null | grep -qi openhpc; then
  echo "ERROR: OpenHPC repo not found — run setup_repos before this postscript"
  exit 1
fi
dnf install -y ohpc-slurm-client
curl -s -o /etc/slurm/slurm.conf http://${MASTER_IP}/install/postscripts/files/slurm.conf
curl -s -o /etc/slurm/cgroup.conf http://${MASTER_IP}/install/postscripts/files/cgroup.conf
mkdir -p /var/spool/slurmd
chown slurm:slurm /var/spool/slurmd
systemctl enable --now munge
systemctl enable --now slurmd
echo "slurmd configured against controller ${MASTER_IP}"
EOF
chmod +x /install/postscripts/setup_slurm_client
```

**Why this step matters** Installs the Slurm client packages, fetches BOTH slurm.conf and cgroup.conf from the master, and creates `/var/spool/slurmd` — the exact directory name slurm.conf's `SlurmdSpoolDir` setting expects. This exact path (no extra characters, matching what OpenHPC's example config specifies) is required; a mismatched spool directory causes slurmd to fail its domain-socket setup even though the service reports as running.

**How to verify** `cat /install/postscripts/setup_repos /install/postscripts/setup_munge /install/postscripts/setup_slurm_client` — review all three files exist and read correctly before proceeding.

---

## Phase 4 — Run the Postscripts Against the New Node

### 4.1 Assign all three postscripts, in order

```bash
chdef cnode01 -p postscripts=setup_repos,setup_munge,setup_slurm_client
```

**Why this step matters** The order matters: repos must exist before Slurm packages can install, and Munge must be running before slurmd starts (slurmd will fail its authentication handshake with the controller otherwise). Using `-p` APPENDS to the existing postscripts list rather than replacing it, preserving xCAT's own default postscripts (syslog, remoteshell, syncfiles).

**How to verify** `lsdef cnode01 -i postscripts` — should list all default AND the three new postscripts, in this relative order.

### 4.2 Execute them

```bash
updatenode cnode01 -P setup_repos,setup_munge,setup_slurm_client
```

**Why this step matters** `updatenode -P` runs specific postscripts against an already-provisioned, already-booted node — no reinstall needed.

**How to verify** Watch the output for each postscript's final echo line ("repos and GPG keys synced...", "munge configured...", "slurmd configured...") with no errors above it.

---

## Phase 5 — Set the Node's Real Hardware Spec in slurm.conf

This phase is REQUIRED for every new node and cannot be skipped or guessed — Slurm will mark a node with incorrect CPU/socket/core counts as invalid, even if every other step succeeded.

### 5.1 Query the node's actual hardware

```bash
xdsh cnode01 "slurmd -C"
```

**Why this step matters** `slurmd -C` asks the node itself to report its real CPU/socket/core/thread/memory layout, in the exact syntax slurm.conf expects. Never hand-type these values — a guessed value that doesn't match reality is the most common cause of a node being marked invalid.

```
# Example output:
NodeName=cnode01 CPUs=1 Boards=1 SocketsPerBoard=1 CoresPerSocket=1 ThreadsPerCore=1 RealMemory=3633
```

### 5.2 Add this exact line to the master's slurm.conf

```bash
# Remove any existing line for this node first:
sed -i '/^NodeName=cnode01/d' /etc/slurm/slurm.conf

# Add the corrected line (append State=UNKNOWN, which slurmd -C does not print):
echo "NodeName=cnode01 CPUs=1 Boards=1 SocketsPerBoard=1 CoresPerSocket=1 ThreadsPerCore=1 RealMemory=3633 State=UNKNOWN" >> /etc/slurm/slurm.conf

# Confirm the partition line includes this node, e.g.:
grep -i "^PartitionName" /etc/slurm/slurm.conf
```

**Why this step matters** `State=UNKNOWN` tells slurmctld to determine the node's actual state from its first registration, rather than assuming a state — required on every NodeName line.

**How to verify** `grep -i "^NodeName=cnode01" /etc/slurm/slurm.conf` — confirm it matches the `slurmd -C` output exactly, with `State=UNKNOWN` appended.

### 5.3 Re-sync the corrected config to both the file server AND the node directly

```bash
cp /etc/slurm/slurm.conf /install/postscripts/files/slurm.conf
xdsh cnode01 "curl -s -o /etc/slurm/slurm.conf http://192.168.245.128/install/postscripts/files/slurm.conf"
```

**Why this step matters** Two separate copies must be updated: the served copy (for any FUTURE node that fetches it) and the copy already sitting on cnode01 (which won't re-fetch on its own). Skipping either one leaves a stale, mismatched config somewhere in the cluster.

### 5.4 Restart both daemons

```bash
systemctl restart slurmctld
xdsh cnode01 "systemctl restart slurmd"
```

**Why this step matters** Both daemons must reload the corrected config from disk — neither picks up a changed slurm.conf automatically while running.

---

## Phase 6 — Verify and Test

### 6.1 Confirm Munge trust between master and node

```bash
munge -n | ssh cnode01 unmunge
```

**How to verify** Must return `STATUS: Success (0)` with a real, current DECODE_TIME — not an epoch-zero "1970" timestamp, which would indicate a key mismatch.

### 6.2 Confirm slurmd is healthy

```bash
xdsh cnode01 "systemctl status slurmd"
```

**How to verify** Must show "active (running)" with no "Domain socket directory" or Munge decode errors in the recent log lines.

### 6.3 Confirm the node's state in Slurm

```bash
sinfo
```

**Why this step matters** This is the definitive check that the controller and compute node agree on everything: authentication, config, and hardware.

> **Note:** If the state shows "inval" or "down" even after Phase 5, check the exact reason before assuming anything: `sinfo -R` and `scontrol show node cnode01` — look at the `Reason=` field. Do not guess a second time; read the actual reason.

### 6.4 If needed, manually clear a stuck state

```bash
# If sinfo shows 'inval', 'down', or 'drain' but Phase 5 was completed correctly:
scontrol update NodeName=cnode01 State=DOWN Reason="reconfig"
scontrol update NodeName=cnode01 State=RESUME
```

**Why this step matters** Slurm does not always automatically clear a node's drain/invalid state even after the underlying cause is fixed. From certain prior states (e.g. INVALID_REG), a direct jump to RESUME is rejected — forcing the node to DOWN first, then RESUME, is a reliable two-step transition.

**How to verify** `sinfo` should now show `STATE=idle` with no asterisk and no drain/inval/down.

### 6.5 Run a real test job

```bash
cd ~
srun -N1 hostname
```

**Why this step matters** This proves the full pipeline end-to-end: the controller schedules the job, authenticates it via Munge, dispatches it to slurmd on the compute node, and returns real output.

> **Note:** Run this from a directory that exists on both the master and the compute node (e.g. your home directory, especially if NFS-shared) — running it from a master-only path like `/install/postscripts` produces a harmless "couldn't chdir" warning before falling back to `/tmp`. The job still completes correctly either way; this is cosmetic, not a failure.

**How to verify** Output should be exactly: `cnode01`

### 6.6 Batch job test (closer to real usage)

```bash
cat > ~/testjob.sh << 'EOF'
#!/bin/bash
#SBATCH -J testjob
#SBATCH -N 1
#SBATCH -o ~/testjob.out
hostname
sleep 5
echo "job complete"
EOF
sbatch ~/testjob.sh
squeue
cat ~/testjob.out
```

**How to verify** `testjob.out` should contain the node's hostname and the line "job complete".

---

## 7. Quick-Reference Checklist for the Next New Node

Once Phases 2 and 3 (file preparation and postscript creation) have been done once, adding another new node only requires:

- **Phase 1** — define the new node in xCAT (mkdef, makehosts, makedns, makedhcp, nodeset, boot it).
- **Phase 4** — chdef the three postscripts onto the new node and run updatenode.
- **Phase 5** — run `slurmd -C` on the NEW node specifically, add its NodeName line, re-sync slurm.conf everywhere, restart both daemons.
- **Phase 6** — verify Munge, slurmd, sinfo, and run a test job.

Phases 2 and 3 only need to be repeated if: the Munge key is regenerated, the OpenHPC/EPEL repos or their GPG keys change, or the postscripts themselves need a bug fix.

### Keep slurm.conf in sync — recommended alias

```bash
alias sync-slurm-conf='cp /etc/slurm/slurm.conf /install/postscripts/files/slurm.conf && echo "slurm.conf synced $(date)"'
```

Add this to `/root/.bashrc` on the master. Run `sync-slurm-conf` every single time slurm.conf is hand-edited, for any reason — this single habit prevents the most common and hardest-to-diagnose failure mode in this entire procedure: the controller and a compute node silently running different configs.
