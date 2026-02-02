---

### 📁 File & Directory Commands

```
ls        -- list the files and directories
pwd       -- present working directory
cd        -- change directory
mkdir     -- create a directory
rmdir     -- remove empty directory
cp        -- copy files or directories
mv        -- move or rename files
rm        -- remove files or directories
tree      -- display directory structure
find      -- search files and directories
locate    -- find files by name (fast search)
```

---

### 📄 File Viewing & Text Commands

```
cat       -- display file content
less      -- view file page by page
more      -- view file (basic pager)
head      -- show first lines of a file
tail      -- show last lines of a file
tail -f   -- monitor file in real time
wc        -- count lines, words, characters
grep      -- search text in files
awk       -- text processing by columns
sed       -- stream editor for text replace
```

---

### 🔐 Permissions & Ownership

```
chmod     -- change file permissions
chown     -- change file owner
chgrp     -- change group ownership
umask     -- default permission settings
```

---

### ⚙️ System Information

```
uname     -- system information
hostname  -- show or set system hostname
uptime    -- system running time
whoami    -- current user
id        -- user and group info
free      -- memory usage
top       -- real-time process view
htop      -- enhanced process viewer
df        -- disk free space
du        -- disk usage
lsblk     -- block devices
lscpu     -- CPU information
```

---

### 🔄 Process Management

```
ps        -- process status
top       -- live process monitor
kill      -- terminate process
kill -9   -- force kill process
pkill     -- kill process by name
pgrep     -- find process id
nice      -- start process with priority
renice    -- change process priority
```

---

### 🌐 Networking Commands

```
ip        -- network configuration
ifconfig  -- network interface info (legacy)
ping      -- check connectivity
traceroute-- trace network path
ss        -- socket statistics
netstat   -- network connections (legacy)
curl      -- download data from URL
wget      -- download files
scp       -- secure file copy
rsync     -- fast file sync
```

---

### 💽 Disk & Filesystem

```
mount     -- mount filesystem
umount    -- unmount filesystem
blkid     -- block device info
fdisk     -- disk partition tool
mkfs      -- create filesystem
fsck      -- filesystem check
```

---

### 📦 Archiving & Compression

```
tar       -- archive files
gzip      -- compress files
gunzip    -- decompress files
zip       -- zip archive
unzip     -- extract zip file
```

---

### 📥 Package Management

**RHEL / Rocky / Alma**

```
dnf install  -- install package
dnf remove   -- remove package
dnf search   -- search package
dnf update   -- update system
```

**Ubuntu / Debian**

```
apt install  -- install package
apt remove   -- remove package
apt update   -- update repo
apt upgrade  -- upgrade system
```

---

### 🔑 Admin & Services

```
sudo       -- run command as root
su         -- switch user
systemctl  -- manage services
journalctl -- view logs
crontab    -- schedule jobs
```

---

### 🧪 Environment & Shell

```
env        -- show environment variables
export     -- set environment variable
printenv   -- print env variables
which      -- command location
whereis    -- binary location
history    -- command history
clear      -- clear screen
```

---

### ⚡ Useful One-liners

```
watch      -- run command repeatedly
time       -- measure execution time
xargs      -- build arguments from input
alias      -- create shortcut command
```

---

## 🚀  **HPC / Linux Admin Specific Commands**

---

### 🖥️ CPU, NUMA & Performance

```
lscpu        -- show CPU architecture and cores
numactl      -- run program with NUMA control
numastat     -- NUMA memory statistics
taskset      -- set CPU affinity for a process
mpstat       -- CPU usage per core
iostat       -- CPU and disk I/O statistics
vmstat       -- memory, process, IO statistics
perf         -- performance profiling tool
sar          -- system activity report
```

---

### 🧠 Memory & Limits (Very Important for HPC)

```
free -h      -- memory usage
ulimit -a    -- show user limits
ulimit -u    -- max processes
ulimit -n    -- max open files
sysctl -a    -- kernel parameters
sysctl -p    -- reload sysctl config
```

---

### ⚙️ Process & Job Control (HPC scale)

```
ps -eLf      -- show threads
pstree       -- process tree
htop         -- interactive process monitor
top -H       -- thread-level view
killall      -- kill all matching processes
strace       -- system call tracing
ltrace       -- library call tracing
```

---

### 💾 Storage & Parallel Filesystems

```
df -Th       -- filesystem type
mount        -- mount filesystem
umount       -- unmount filesystem
lsblk        -- block devices
blkid        -- UUID and device info
lfs df -h    -- Lustre filesystem usage
lfs quota    -- Lustre quota
lctl list_nids -- Lustre network info
```

---

### 🌐 Networking (Cluster & MPI)

```
ip a         -- show IP addresses
ip r         -- routing table
ss -lntup    -- listening ports
ping         -- test connectivity
ibstat       -- InfiniBand status
ibv_devinfo  -- IB device info
rdma link    -- RDMA device status
ethtool eth0 -- NIC details
```

---

### 🧩 MPI & Parallel Runtime

```
mpirun       -- run MPI program
mpiexec      -- execute MPI job
mpicc        -- MPI C compiler
mpicxx       -- MPI C++ compiler
mpif90       -- MPI Fortran compiler
ompi_info    -- OpenMPI configuration info
```

---

### 📊 GPU / Accelerator (NVIDIA)

```
nvidia-smi   -- GPU status and usage
nvidia-smi -l 1 -- live GPU monitoring
nvcc         -- CUDA compiler
cuda-gdb     -- CUDA debugger
dcgmi        -- datacenter GPU manager
```

---

### 📦 Modules & Software Management

```
module avail -- list modules
module load  -- load software
module unload-- unload software
module list  -- loaded modules
module purge -- unload all modules
```

---

### 🗂️ Scheduler (Slurm / PBS / LSF)

#### Slurm

```
sinfo        -- cluster status
squeue       -- job queue
srun         -- run job
sbatch       -- submit job
scancel      -- cancel job
scontrol     -- job control
```

#### PBS / Torque

```
qstat        -- job status
qsub         -- submit job
qdel         -- delete job
qhost        -- node info
pbsnodes     -- node details
```

#### LSF

```
bjobs        -- job status
bsub         -- submit job
bkill        -- kill job
bhosts       -- node info
```

---

### 🔐 System Services & Admin (Cluster nodes)

```
systemctl status -- service status
systemctl restart -- restart service
journalctl -xe   -- service logs
timedatectl      -- time sync
chronyc sources  -- NTP status
```

---

### 🛠️ Debugging & Troubleshooting

```
dmesg       -- kernel messages
journalctl  -- system logs
ldd         -- library dependencies
ldconfig    -- update linker cache
strace      -- trace syscalls
lsof        -- open files by process
```

---

### ⚡ HPC Best Practice Tools

```
pdsh        -- parallel ssh
pssh        -- parallel shell
clush       -- cluster shell
ansible     -- cluster automation
```

---
