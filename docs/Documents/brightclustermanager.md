# HPC Cluster Management Experience
## Using Bright Cluster Manager

I have **2+ years of hands-on experience** working with **Bright Cluster Manager** in a large-scale **HPC production environment**.  
My role involved **installation, configuration, provisioning, and operational management** of a **1300+ node HPC cluster**, including **CPU, GPU, diskless, and login nodes**.

---

## Bright Cluster Manager Installation & Initial Setup
- Installed Bright Cluster Manager on the **management (head) node**
- Performed initial cluster setup and base configurations
- Configured:
    - Management node
    - Core cluster services
    - Authentication and basic policies

---

## Node Category Design & Management
Designed and configured **multiple node categories** based on workload and usage.

### Diskless Compute Nodes
- PXE-based provisioning
- Diskless boot configuration
- Optimized for high-performance compute workloads

### GPU Nodes
- Dedicated GPU node category
- Installed:
    - NVIDIA drivers
    - CUDA libraries
- Created and maintained GPU-specific OS images

### Login Nodes
- Separate login node category
- Optimized for:
    - User access
    - Job submission
    - Development activities

Each node category was managed independently using **custom configurations and images**.

---

## Image Management & Provisioning
- Created **separate OS images** for each node category
- Customized images based on:
    - Node role
    - Hardware type
    - Software requirements
- Pushed images to nodes using Bright Cluster Manager provisioning
- Verified provisioning status and resolved image-related issues

---

## Network Configuration
- Configured **management network** for all cluster nodes
- Set up **InfiniBand network** for high-speed interconnect
- Managed:
    - Network interfaces
    - Node-to-network mapping
    - Category-based network configuration
- Applied network settings consistently across node groups

---

## Bulk Node Addition & Automation
- Added **all compute nodes at once** using shell scripting
- Automated:
    - Node definitions
    - MAC address mapping
    - Category assignment
- Reduced manual effort and provisioning errors

---

## Node-Specific Configuration
- Applied category-specific configurations:
    - NVIDIA drivers and CUDA for GPU images
    - Performance tuning for compute nodes
- Validated:
    - Provisioning success
    - Driver installation
    - Network connectivity

---

## Rack & Data Center Layout Management
- Created **rack configurations** in Bright Cluster Manager
- Added:
     - Racks
    - Servers
    - Switches
- Defined:
    - Server positions in racks
    - Switch positions
    - Physical layout mapping
- Maintained accurate **data center visualization**

---

## Switch Configuration
- Added and configured **network switches**
- Associated switches with racks and nodes
- Maintained logical and physical network topology

---

## Monitoring & Hardware Troubleshooting
- Continuously monitored cluster and node health
- Tracked:
    - Node status
    - Hardware alerts
    - Sensor data (fans, power, temperature)
- Identified and reported:
    - Hardware failures
    - Disk issues
    - Network and node boot problems
- Assisted in proactive issue resolution to reduce downtime

---

## Key Skills & Technologies
- Bright Cluster Manager (Installation & Administration)
- Large-scale cluster provisioning (1300+ nodes)
- Diskless node configuration
- GPU cluster management
- NVIDIA drivers and CUDA
- InfiniBand and management network configuration
- Image-based OS management
- Shell scripting and automation
- Rack and data center layout management
- Hardware monitoring and troubleshooting

---
