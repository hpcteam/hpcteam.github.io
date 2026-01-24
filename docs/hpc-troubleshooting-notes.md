# HPC Operational Issues & Troubleshooting Scenarios

This page documents common operational issues, failure scenarios, and troubleshooting
areas encountered while supporting large-scale Linux and High-Performance Computing
(HPC) environments in production. The content is based on real-world cluster operations
and incident handling.

---

## 1. User Access & Connectivity Issues

- Users unable to access cluster nodes due to routing or network configuration issues
- IP address changes in mixed or static network environments causing connectivity loss
- Users unable to ping compute or management node IP addresses
- SSH, SCP, and file copy failures between cluster nodes
- Access issues caused by incorrect firewall rules or network policies

---

## 2. Job Scheduling & Execution Issues

- Jobs remaining in queue due to compute node unavailability
- Jobs failing due to incorrect software paths or environment configuration
- Jobs failing when assigned nodes become unresponsive
- Jobs entering held or stalled state due to time drift across compute nodes
- Scheduler-related issues affecting job start or completion

---

## 3. Benchmarking & Performance Issues

- Benchmark tests failing on specific nodes due to mismatched firmware versions
- HPL benchmark failures on single nodes caused by:
  - Slurm errors
  - Munge service failures
  - Munge key mismatches
- HPL benchmark failures due to time differences across nodes
  (date or hardware clock mismatch)
- HPL calculation errors during benchmark execution
- Communication errors during multi-node benchmark runs
- Script-related issues in PBS or HPL execution scripts

---

## 4. GPU & NVIDIA Software Issues

- NVIDIA driver installation failures
- Incorrect or incompatible driver versions installed
- Partial GPU visibility on GPU nodes (e.g., fewer GPUs detected than expected)
- Application failures due to CUDA, driver, or library mismatches
- GPU-related performance degradation due to configuration issues

---

## 5. Cluster Management & Licensing Issues

- Inability to access cluster management interfaces
- Issues while syncing or pushing cluster configuration changes
- UFM license-related issues
- UFM web portal access problems
- Bright Cluster Manager license issues
- Cluster lock conditions caused by license problems
- Unlocking and restoring cluster functionality after license resolution

---

## 6. Network & Fabric Issues

- InfiniBand fabric communication failures
- Nodes unable to communicate with license servers
- Network configuration issues impacting application communication
- Fabric-level issues affecting parallel workload performance

---

## 7. Hardware-Level Issues

- Disk and root filesystem failures
- NIC card issues causing application or connectivity problems
- InfiniBand cable faults and physical connectivity issues
- DIMM errors detected during system operation
- CPU-related hardware errors impacting node stability

---

## 8. Software Installation & Compilation Issues

- Compilation errors during application installation
- Installation failures caused by missing dependencies
- High-availability (HA) related errors leading to node issues
- Runtime failures caused by incorrect software paths
- Version mismatches between application components

---

## 9. Documentation & Operational Practices

- Preparing dependency and prerequisite documentation before installations
- Capturing screenshots and logs during critical cluster changes
- Tracking configuration changes across cluster nodes
- Maintaining operational records for troubleshooting and audits

---

## 10. Parallel Computing Concepts (Reference Notes)

### MPI (Message Passing Interface)
- Distributed memory programming model
- Scales across multiple nodes
- Used for inter-node communication in HPC workloads

### OpenMP
- Shared memory programming model
- Used within a single compute node
- Suitable for multi-threaded computation

MPI is commonly used for inter-node parallelism, while OpenMP is used for intra-node
parallelism.

---

## 11. CPU Affinity & Scheduler Concepts

- CPU affinity configuration in Slurm
- Binding processes to specific CPU cores
- Benefits include improved cache locality and execution efficiency

### Common Slurm Options
- `--cpu-bind`
- `--threads-per-core`
- `--ntasks`
- `--cpus-per-task`

These options help control task placement and CPU utilization for performance-sensitive
workloads.

