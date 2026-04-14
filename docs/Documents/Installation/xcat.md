# Documentation Structure for xCAT

---

# 1. Introduction

xCAT (Extreme Cloud Administration Toolkit) is an open-source cluster management software designed to deploy, provision, and manage large-scale computing environments such as High Performance Computing (HPC) clusters and data centers.

In HPC infrastructures, administrators often need to manage dozens or hundreds of compute nodes. Performing operating system installation, configuration, and maintenance manually on each node would be time-consuming and inefficient. xCAT solves this problem by providing centralized management from a single management node.

xCAT automates several critical tasks required for cluster deployment and administration. These include network-based operating system installation using PXE boot, node configuration, software updates, and remote management of cluster nodes. By integrating services such as DHCP, TFTP, HTTP, and NFS, xCAT enables compute nodes to boot from the network and automatically install the required operating system and software stack.

In a typical cluster architecture, a management node runs xCAT services and controls multiple compute nodes connected through a management network. When a compute node is powered on, it performs a PXE boot, obtains network configuration from the DHCP server, downloads boot files from the TFTP server, and begins operating system installation provided by the management node.

Because of its scalability, automation capabilities, and integration with enterprise Linux environments, xCAT is widely used in HPC clusters, research laboratories, and large-scale data center infrastructures.

---

# 2. Architecture Overview

## Architecture Overview

The architecture of xCAT is designed to manage and provision large numbers of nodes in a cluster from a centralized system. It follows a management-based architecture, where one main system (management node) controls and deploys operating systems and configurations to multiple compute nodes through network services.

In a typical HPC cluster environment, the xCAT architecture consists of several core components that work together to automate node provisioning, configuration, and management.

---

## 2.1. Management Node

The Management Node (MN) is the central system where xCAT is installed and configured. It controls all cluster operations and maintains the configuration database for all nodes.

### Responsibilities of the Management Node

- Runs the xCAT management services
- Stores cluster configuration in the xCAT database
- Provides OS images for node deployment
- Controls network boot services
- Executes commands across multiple nodes
- Manages node definitions and attributes

The management node typically runs several system services such as:

| Service | Purpose |
|------|------|
| DHCP | Assigns IP addresses to compute nodes |
| TFTP | Provides boot files for PXE boot |
| HTTP | Provides OS installation files |
| NFS | Shares filesystems with nodes |
| xcatd | Main xCAT daemon that handles commands |

All cluster administration commands are executed from the management node.

---

## 2.2. Compute Nodes

Compute Nodes are the systems that perform the actual computational workloads in the cluster. These nodes are managed by the management node using xCAT.

Compute nodes usually do not require manual installation because xCAT automatically installs the operating system through network boot.

### Characteristics of Compute Nodes

- Boot from the network using PXE
- Receive IP configuration from DHCP
- Download boot loader and kernel through TFTP
- Install OS from HTTP/NFS repository
- Register themselves with the management node

Compute nodes can be categorized based on their roles:

| Node Type | Description |
|------|------|
| Compute Node | Performs computation tasks |
| Login Node | Provides user access to the cluster |
| GPU Node | Contains GPUs for accelerated computing |
| Storage Node | Provides shared storage |

---

## 2.3. Service Node (Optional)

In large clusters, a Service Node (SN) may be used to distribute management tasks and reduce load on the management node.

Service nodes act as intermediaries between the management node and compute nodes.

### Responsibilities of Service Nodes

- Provide DHCP services
- Serve OS images
- Manage subsets of nodes
- Reduce network and CPU load on the management node

This architecture improves scalability when clusters contain hundreds or thousands of nodes.

---

## 2.4. xCAT Database

xCAT maintains a cluster configuration database that stores information about all nodes and their attributes.

This database contains details such as:

- Node names
- MAC addresses
- IP addresses
- Operating system image
- Boot configuration
- Node roles

Administrators interact with the database using commands such as:

```
lsdef
mkdef
chdef
```

These commands allow administrators to define and manage nodes in the cluster.

---

## 2.5. Network Architecture

xCAT environments typically use multiple networks to separate cluster management traffic from user traffic.

### Management Network

The management network is used for cluster provisioning and internal communication.

Functions of this network include:

- PXE boot
- OS installation
- Node configuration
- Cluster management commands

### External / Public Network

The external network allows users to access the cluster and submit jobs.

---

## 2.6. PXE Boot Workflow

One of the most important features of xCAT is network-based operating system deployment using PXE boot.

The process works as follows:

1. A compute node powers on.
2. The node sends a DHCP request on the network.
3. The DHCP server running on the management node assigns an IP address.
4. The DHCP response provides the location of the boot file.
5. The compute node downloads the boot loader from the TFTP server.
6. The boot loader loads the kernel and initrd.
7. The operating system installation starts from the HTTP or NFS repository on the management node.
8. The node completes installation and becomes part of the cluster.

---

## 2.7. xCAT Command Framework

xCAT provides command-line tools that allow administrators to control multiple nodes simultaneously.

Examples include:

| Command | Purpose |
|------|------|
| nodels | Lists nodes |
| nodeset | Sets boot state for nodes |
| rpower | Controls node power |
| updatenode | Updates nodes with configuration changes |
| xdsh | Runs commands on multiple nodes |

These commands allow administrators to manage large clusters efficiently.

---

## 2.8. Boot and Image Repository

The management node stores operating system images that are deployed to compute nodes.

These images are typically stored under directories such as:

```
/install
/tftpboot
```

These directories contain:

- OS installation files
- Boot loaders
- Kernel images
- Initial RAM disks (initrd)

When nodes boot through PXE, they retrieve these files from the management node.

---

## 2.9. Scalability of xCAT Architecture

The architecture of xCAT is designed to scale from small clusters to very large HPC systems.

Small clusters may use only:

- One management node
- Several compute nodes

Large clusters may include:

- Management nodes
- Multiple service nodes
- Hundreds or thousands of compute nodes
- Dedicated storage nodes
- High-speed interconnect networks such as InfiniBand

This scalable architecture allows xCAT to be used in research institutions, enterprise HPC environments, and large supercomputing facilities.

---

# 3. Prerequisites

## Prerequisites

Before installing and configuring xCAT, several system, network, and software components must be prepared. These prerequisites ensure that the cluster provisioning process, PXE boot, and automated node deployment work properly.

The prerequisites can be grouped into the following categories:

- Hardware Requirements
- Operating System Requirements
- Network Configuration
- Required Services
- Required Software Packages
- System Configuration

---

## 3.1. Hardware Requirements

A basic xCAT cluster requires at least one management node and one or more compute nodes.

### Management Node

The management node is the central system where xCAT is installed. It manages all compute nodes and provides required services for provisioning.

Typical specifications

| Component | Recommended Specification |
|------|------|
| CPU | 4 cores or higher |
| RAM | 8 GB or more |
| Storage | 200 GB or higher |
| Network Interfaces | Minimum 2 NICs |

Purpose of network interfaces

| Interface | Purpose |
|------|------|
| External Network Interface | Used for user access and internet connectivity |
| Management Network Interface | Used for provisioning compute nodes |

The management node stores:

- OS installation images
- Boot files
- xCAT configuration database
- Cluster configuration data

---

### Compute Nodes

Compute nodes perform the actual computational workloads.

Minimum requirements include:

| Component | Description |
|------|------|
| CPU | Multi-core processor |
| RAM | Depends on workload |
| Network Card | PXE boot capable |
| Storage | Optional (diskless nodes supported) |

Important requirement:

The network interface must support PXE boot, because nodes receive their operating system through the network.

---

## 3.2. Operating System Requirements

The management node must run a supported Linux distribution.

Common operating systems used with xCAT include:

- Red Hat Enterprise Linux
- Rocky Linux
- CentOS

Recommended installation type:

- Minimal Server Installation
- Static IP configuration
- Root access enabled

After installing the operating system, repositories should be configured to install the required packages.

---

## 3.3. Network Configuration

Proper network setup is essential for xCAT.

Typically, two networks are used in HPC clusters.

### Management Network

This network is used for cluster provisioning.

Functions include:

- PXE boot
- Node installation
- Cluster management communication

Example configuration:

| Parameter | Example |
|------|------|
| Network | 192.168.1.0/24 |
| Management Node IP | 192.168.1.1 |
| Node IP Range | 192.168.1.100 – 192.168.1.200 |

All compute nodes must be connected to this network.

---

### External Network

This network allows users to access the cluster.

Typical uses:

- SSH access
- job submission
- external connectivity

Example:

| Parameter | Example |
|------|------|
| Network | 10.2.6.0/24 |
| Gateway | 10.2.6.1 |

---

## 3.4. Required Network Services

Several services must run on the management node to support node provisioning.

### DHCP Service

DHCP assigns IP addresses to compute nodes during boot.

Common package:

```
dhcp-server
```

Functions:

- Assign IP addresses
- Provide boot file location
- Specify the next boot server

Without DHCP, nodes cannot receive network configuration.

---

### TFTP Service

TFTP transfers boot files required during PXE boot.

Common package:

```
tftp-server
```

Boot files are usually stored in:

```
/tftpboot
```

Files include:

- PXE boot loader
- kernel
- initrd

---

### HTTP Server

An HTTP server is used to provide operating system installation files.

Common web server used:

- Apache HTTP Server

Package:

```
httpd
```

OS images are usually stored in:

```
/install
```

Compute nodes download installation files from this location.

---

### NFS Server (Optional)

Network File System can be used to share files between cluster nodes.

Package:

```
nfs-utils
```

Common uses:

- shared software directories
- shared user data
- shared cluster tools

---

## 3.5. Required xCAT Software Packages

After preparing the system repositories, the xCAT packages must be installed.

Main package:

```
xCAT
```

This installs several components:

| Component | Description |
|------|------|
| xcatd | Main xCAT daemon |
| xCAT database | Stores node configuration |
| provisioning tools | OS deployment utilities |
| node management tools | Cluster administration commands |

Once installed, the xCAT daemon must be started.

---

## 3.6. Required Directory Structure

The management node must contain directories used for provisioning.

Important directories include:

| Directory | Purpose |
|------|------|
| /install | Stores OS installation images |
| /tftpboot | Stores PXE boot files |
| /etc/xcat | xCAT configuration files |
| /var/lib/xcat | xCAT database |

These directories are automatically used during node provisioning.

---

## 3.7. Time Synchronization

All cluster nodes should have synchronized time.

Time synchronization prevents issues with:

- logging
- authentication
- job scheduling

Common services include:

- Chrony
- Network Time Protocol

---

## 3.8. Security Configuration

Some security features may interfere with provisioning and need adjustment.

### Firewall

The firewall must allow the following services:

- DHCP
- TFTP
- HTTP
- NFS (if used)

### SELinux

SELinux may need to be configured or temporarily disabled during installation.

Example:

```
setenforce 0
```

---

## 3.9. BIOS Configuration for Compute Nodes

Compute nodes must be configured to support network boot.

Required BIOS settings:

- Enable PXE Boot
- Set Network Boot as First Boot Device
- Configure UEFI or Legacy Boot mode

Without these settings, compute nodes cannot boot from the network.

---

# 4. Installation and Initial Configuration

---

# 4.1 Preparing the Management Node

Before installing xCAT, the management node must be prepared with a properly configured operating system and required system settings.

### 4.1.1 Install the Operating System

Install a supported Linux distribution on the management node.

Commonly used operating systems include:

* Red Hat Enterprise Linux
* Rocky Linux

Recommended installation type:

* Minimal installation
* Static IP configuration
* Root access enabled

---

### 4.1.2 Configure Network Interfaces

The management node must have a properly configured **static IP address** for the management network. This IP address will be used by services such as DHCP, TFTP, and HTTP for node provisioning in xCAT.

Two common methods can be used to configure a static IP address.

---

#### Option 1: Configure Static IP using `nmcli`

Identify the network interface:

```bash
nmcli device status
```

Configure the static IP address:

```bash
nmcli connection modify eth1 ipv4.addresses 192.168.1.1/24
nmcli connection modify eth1 ipv4.gateway 192.168.1.254
nmcli connection modify eth1 ipv4.method manual
nmcli connection modify eth1 connection.autoconnect yes
```

Restart the network connection:

```bash
nmcli connection down eth1
nmcli connection up eth1
```

Verify the configuration:

```bash
ip addr show eth1
```

---

#### Option 2: Configure Static IP by Editing Network Script

Edit the interface configuration file:

```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth1
```

Example configuration:

```bash
TYPE=Ethernet
BOOTPROTO=none
NAME=eth1
DEVICE=eth1
ONBOOT=yes
IPADDR=192.168.1.1
PREFIX=24
GATEWAY=192.168.1.254
DNS1=8.8.8.8
```

Restart the network service:

```bash
systemctl restart NetworkManager
```

Verify the IP address:

```bash
ip addr
```

---

### 4.1.3 Configure Hostname and DNS Name

Set the hostname of the management node. It is recommended to configure a **Fully Qualified Domain Name (FQDN)** for proper cluster identification.

Example hostname:

```
mgmt.cluster.com
```

Set the hostname using the following command:

```bash
hostnamectl set-hostname mgmt.cluster.com
```

Verify the hostname:

```bash
hostnamectl
```

Update the hosts file if required:

```bash
vi /etc/hosts
```

Example entry:

```bash
192.168.1.1   mgmt.cluster.com   mgmt
```

This ensures proper hostname resolution within the cluster environment.

---

### 4.1.4 Update System Packages or Configure Local Repository

Before installing xCAT, ensure that all required system libraries and dependencies are available.

If the management node has **internet connectivity**, update the system packages:

```bash
dnf update -y
```

This ensures that all system libraries and dependencies are updated to the latest version.

If the management node **does not have internet access**, configure a **local repository** using a mounted operating system ISO or internal repository server. This allows required packages to be installed without internet connectivity.

Local repositories are commonly used in **secure HPC environments and offline clusters**.

### 4.1.5 Disable SELinux 

Some provisioning services may fail if SELinux policies block them.

Temporarily disable SELinux:

```
setenforce 0
```

To disable permanently:

Edit the configuration file:

```
/etc/selinux/config
```

Change:

```
SELINUX=enforcing
```

to

```
SELINUX=disabled
```
Reboot need to reflecting this change
---

### 4.1.6 Configure Firewall Rules

The firewall configuration depends on the cluster environment requirements when deploying xCAT.

If the cluster environment **requires the firewall to remain enabled**, then necessary services must be allowed through the firewall so that node provisioning services can function properly.

Required services include:

* DHCP
* TFTP
* HTTP
* NFS (if used)

Add the required firewall rules:

```bash
firewall-cmd --permanent --add-service=dhcp
firewall-cmd --permanent --add-service=tftp
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=nfs
```

Reload the firewall configuration:

```bash
firewall-cmd --reload
```

Verify the active firewall rules:

```bash
firewall-cmd --list-all
```

If the cluster environment **does not require firewall protection**, the firewall service can be disabled.

Stop the firewall service:

```bash
systemctl stop firewalld
```

Disable it at system startup:

```bash
systemctl disable firewalld
```

---

### 4.1.7 Verify Repository Configuration

Before installing xCAT, verify that the required repositories are configured correctly on the management node.

Run the following command:

```bash
dnf repolist
```

or

```bash
yum repolist
```

The output should list the required repositories, including:

* BaseOS
* AppStream
* xcat-core
* xcat-dep

Example expected output:

```
repo id          repo name
BaseOS           BaseOS Repository
AppStream        AppStream Repository
xcat-core        xCAT Core Repository
xcat-dep         xCAT Dependencies Repository
```

If these repositories are listed, the system is ready to install xCAT packages. If not, the repositories must be configured before proceeding with the installation.

---
## 5. Installation of xCAT

This section explains the installation procedure of xCAT on the management node.

After configuring the system prerequisites and repositories, the xCAT packages can be installed using the package manager.

---

### 5.1 Install xCAT Packages

Install the xCAT software using the system package manager.

Using **dnf**:

```bash
dnf install xCAT -y
```

or using **yum**:

```bash
yum install xCAT -y
```

After executing the command, the system will begin downloading and installing the required packages.

---

### 5.2 Wait for Required Package Installation

During the installation process, several dependent packages will also be installed automatically. This process may take some time depending on system performance and repository speed.

The installation includes multiple xCAT components and utilities required for cluster provisioning and management.

---

### 5.3 Resolve Dependency Errors (If Any)

If any errors occur during installation, such as **missing dependency errors**, it usually means that the required repositories are not configured correctly.

In such cases:

* Verify repository configuration using:

```bash
dnf repolist
```

* Ensure that the following repositories are available:

| Repository | Purpose                        |
| ---------- | ------------------------------ |
| BaseOS     | Base operating system packages |
| AppStream  | Application libraries          |
| xcat-core  | Core xCAT packages             |
| xcat-dep   | xCAT dependency packages       |

If any repository is missing, configure the required repository and run the installation command again.

---

### 5.4 Verify Installed Services

After the installation completes, several required services are installed automatically along with xCAT.

Important services include:

* DHCP service
* HTTP service
* xCAT daemon

Check the status of these services:

```bash
systemctl status dhcpd
```

```bash
systemctl status httpd
```

```bash
systemctl status xcatd
```

If the services are installed correctly, they should appear in the system service list.

---

### 5.5 Verify xCAT Installation

After installation, verify that xCAT commands are available in the system environment.

Load the xCAT environment variables:

```bash
source /etc/profile.d/xcat.sh
```

Check the xCAT version using:

```bash
lsdef -v
```

or

```bash
lsdef --version
```

### Example Output

After loading the xCAT environment and running the version command, the system should display the installed version of xCAT.

```bash
[root@mgmt ~]# lsdef -v
lsdef - Version 2.17.0 (git commit 2960b0e9f948abf4af8ce6c1f459191649883f15, built Wed Nov 13 15:34:26 CET 2024)
```

This confirms that the xCAT installation has been completed successfully and the system is ready for further configuration and node provisioning.

---

## 6. Initial Cluster Service Verification and DHCP Configuration

After installing xCAT, the next step is to verify that the **management node is properly configured for cluster operations**. Before configuring DHCP and other provisioning services, it is recommended to check the health status of the xCAT management node.

---

### 6.1 Verify xCAT Management Node Status

The `xcatprobe` command is used to check whether the **cluster management node is correctly configured and ready for production use**.

Run the following command:

```bash
xcatprobe xcatmn
```

This command performs a series of validation checks on the management node, including:

* xCAT daemon status
* xCAT configuration tables
* Provisioning network configuration
* Required system services
* Directory configuration
* Security settings
* Disk space availability

---

### Example Output

```bash
[root@mgmt ~]# xcatprobe xcatmn
[mn]: Checking all xCAT daemons are running... [ OK ]
[mn]: Checking xcatd can receive command request... [ OK ]
[mn]: Checking 'site' table is configured... [ OK ]
[mn]: Checking provision network is configured... [ OK ]
[mn]: Checking HTTP service is configured... [ OK ]
[mn]: Checking TFTP service is configured... [ OK ]
[mn]: Checking DNS service is configured... [ OK ]
[mn]: Checking DHCP service is configured... [ OK ]
[mn]: Checking NTP service is configured... [ OK ]
[mn]: Checking firewall is disabled... [ OK ]
[mn]: Checking minimum disk space for xCAT... [ OK ]
=================================== SUMMARY ====================================
[MN]: Checking on MN... [ OK ]
```

---

### 6.2 Purpose of the `xcatprobe` Check

The `xcatprobe xcatmn` command verifies whether the **management node is ready for cluster provisioning**. It ensures that the required services and configurations are properly set.

Important checks performed include:

| Check             | Purpose                                  |
| ----------------- | ---------------------------------------- |
| xcatd daemon      | Confirms xCAT service is running         |
| site table        | Verifies cluster configuration           |
| Provision network | Ensures management network is configured |
| HTTP service      | Provides OS installation files           |
| TFTP service      | Provides PXE boot files                  |
| DHCP service      | Assigns IP addresses to nodes            |
| DNS service       | Resolves node hostnames                  |
| NTP service       | Synchronizes time across cluster         |
| Disk space        | Ensures enough storage for OS images     |

---

### 6.3 Handling Warning Messages

Sometimes the command may display **warning messages**, for example:

```
No interface provided by '-i' option, detected 'site' table IP attribute
```

This warning occurs when the interface is not explicitly specified during the check. It usually does not affect cluster functionality if the detected IP address is correct.

---

### 6.4 Next Configuration Steps

After verifying the management node status, the next step is to configure the **cluster provisioning services** required for node deployment.

These services will be configured step by step in the following sections:

1. DHCP configuration
2. TFTP configuration
3. HTTP configuration
4. DNS configuration
5. Time synchronization service

These services work together to enable **PXE-based operating system installation for compute nodes** in the cluster environment.

---

**6.5 DHCP Configuration for PXE Boot**

### 6.5 DHCP Configuration for PXE Boot

In an xCAT cluster environment, the **DHCP service** is required to enable **network boot (PXE boot)** for compute nodes. When a compute node starts, it sends a DHCP request to obtain network configuration such as an IP address and boot information.

The DHCP server provides the following information to the node:

- IP address  
- subnet information  
- next boot server  
- PXE boot file location  

This allows the node to download boot files from the management node and start the operating system installation using entity["software","xCAT","Extreme Cloud Administration Toolkit"].

---

### Configure DHCP Interface in xCAT

First, configure which **network interface on the management node** will provide DHCP services for provisioning.

Run the following command:

```bash
chdef -t site dhcpinterfaces="eth1"
```

**Explanation:**

- `chdef` – Used to modify xCAT configuration tables  
- `-t site` – Specifies the **site table** in the xCAT database  
- `dhcpinterfaces="eth1"` – Defines the network interface used by the DHCP server  

This configuration tells xCAT to use **interface `eth1`** to serve DHCP requests for compute nodes during PXE boot.

---

### Important Note About IPv6

In some environments, the system may automatically detect and configure **IPv6 addresses**. However, for most HPC cluster deployments it is recommended to use **IPv4 addresses** for provisioning networks.

Therefore, it is better to explicitly configure the **IPv4 management interface** to avoid conflicts with IPv6 addresses.

---

### Verify DHCP Interface Configuration

After setting the DHCP interface, verify that the configuration is correctly stored in the xCAT database.

Run the following command:

```bash
tabdump site | grep dhcpinterfaces
```

Expected output example:

```
dhcpinterfaces=eth1
```

This confirms that the DHCP service will operate on the specified interface.

---

### Regenerate DNS Configuration

If the system incorrectly detects **IPv6 entries or disabled values**, regenerate the DNS configuration to ensure correct host resolution.

Run the following commands:

```bash
makedns -n
```

```bash
makedns -a
```

**Explanation:**

- `makedns -n` – Generates DNS configuration entries  
- `makedns -a` – Applies the DNS configuration to the system  

These commands update the DNS configuration based on the current xCAT database settings.

You’re right to question it. **Yes — the previous two points are mostly the same**, because both used the `network` table and `dynamicrange`. In good documentation, they should be **separated clearly**:

* One point → **Define the provisioning network**
* Another point → **Define the DHCP IP range**

Let me correct it so your document is clean and professional.

---

## 6.6 Configure Provisioning Network in xCAT

In xCAT, the provisioning network must be defined in the **xCAT network table**. This network is used by the management node to communicate with compute nodes during PXE boot and operating system deployment.

Configure the provisioning network using the following command:

```bash
chdef -t network netname=provnet net=172.18.0.0 mask=255.255.255.0 mgtifname=eth1 gateway=172.18.0.1
```

### Explanation

| Parameter   | Description                           |
| ----------- | ------------------------------------- |
| `netname`   | Name of the provisioning network      |
| `net`       | Network address                       |
| `mask`      | Subnet mask                           |
| `mgtifname` | Interface used by the management node |
| `gateway`   | Gateway of the provisioning network   |

Verify the configuration:

```bash
lsdef -t network
```

This confirms that the provisioning network is registered in the xCAT database.

---

## 6.7 Configure DHCP IP Range for Compute Nodes

After defining the provisioning network, configure the **dynamic IP address range** that will be assigned to compute nodes during PXE boot.

Use the following command:

```bash
chdef -t network provnet dynamicrange=172.18.0.100-172.18.0.200
```

### Explanation

| Parameter      | Description                         |
| -------------- | ----------------------------------- |
| `provnet`      | Name of the network created earlier |
| `dynamicrange` | IP address range used by DHCP       |

Example allocation:

| Device          | Example IP                  |
| --------------- | --------------------------- |
| Management Node | 172.18.0.2                  |
| Compute Nodes   | 172.18.0.100 – 172.18.0.200 |

Verify the range:

```bash
lsdef -t network provnet
```

## 6.8 Generate and Apply DHCP Configuration

After defining the provisioning network and IP range in xCAT, the next step is to generate the **DHCP server configuration file**.
xCAT automatically creates the DHCP configuration based on the values stored in its database tables.

This is done using the `makedhcp` command.

---

### Generate DHCP Configuration

Run the following command to generate the DHCP configuration:

```bash
makedhcp -n
```

**Explanation**

* `makedhcp` → xCAT command used to generate DHCP configuration
* `-n` → Generates the DHCP configuration file but does not apply it yet

This command reads the **network table, node definitions, and site configuration** from the xCAT database and generates the required DHCP configuration.

---

### Apply DHCP Configuration

After generating the configuration, apply it using:

```bash
makedhcp -a
```

**Explanation**

* `-a` → Applies the generated DHCP configuration to the system

This step updates the DHCP server configuration file typically located at:

```
/etc/dhcp/dhcpd.conf
```

---

### Restart the DHCP Service

After applying the configuration, restart the DHCP service:

```bash
systemctl restart dhcpd
```

Enable the service to start automatically during system boot:

```bash
systemctl enable dhcpd
```

---

### Verify DHCP Service Status

Check whether the DHCP service is running correctly:

```bash
systemctl status dhcpd
```

If the service is running successfully, the management node is now ready to assign **IP addresses to compute nodes during PXE boot**.

---

### Purpose of DHCP in xCAT

The DHCP service plays an important role in cluster provisioning:

| Function            | Description                                      |
| ------------------- | ------------------------------------------------ |
| Assign IP Address   | Provides IP addresses to compute nodes           |
| Provide Boot Server | Specifies the management node as the boot server |
| Provide Boot File   | Provides the PXE boot loader                     |
| Enable PXE Boot     | Allows nodes to boot from the network            |

With DHCP configured, compute nodes can now receive network configuration and start the **PXE-based operating system installation process**.

After configuring the DHCP interface and verifying the site table, the next step is to configure the **DHCP server settings such as IP ranges, next-server, and boot files** for compute node provisioning.
---
Good catch. In a properly configured **xCAT management node**, you usually **do not manually install a separate TFTP server** because xCAT already handles the **TFTP service internally through xcatd**. Your documentation should reflect that clearly.

Here is the corrected section you can place in your document.

---

# 7. TFTP Service in xCAT

In an xCAT cluster environment, the **TFTP service is used during the PXE boot process** to transfer boot files from the management node to compute nodes.

Unlike a traditional setup, there is **no need to manually install or configure a separate TFTP server**, because xCAT automatically manages the required TFTP functionality.

During xCAT installation, the necessary boot infrastructure and services are configured automatically.

---

### Verify TFTP Port Usage

To verify that the TFTP service is already active, check the listening ports on the system:

```bash
lsof -i :69
```

or

```bash
netstat -tulnp | grep 69
```

Example output may show that the port is being used by the xCAT service:

```
xcatd   1234 root   UDP   *:69
```

This indicates that the **TFTP service is already handled by xCAT**.

---

### Verify TFTP Directory

The PXE boot files used by xCAT are stored in the following directory:

```
/tftpboot
```

You can verify the directory contents using:

```bash
ls /tftpboot
```

Typical directories include:

| Directory    | Purpose                             |
| ------------ | ----------------------------------- |
| pxelinux.cfg | PXE configuration files             |
| xcat         | xCAT boot configuration             |
| kernels      | Kernel images used during node boot |

---

### Role of TFTP in Node Provisioning

During node provisioning the workflow is:

1. Compute node sends a **DHCP request**
2. DHCP server returns **boot server and boot file**
3. Node contacts the **TFTP service on the management node**
4. Boot loader is downloaded
5. Kernel and initrd are loaded
6. Operating system installation starts


Add this as another subsection in your documentation.

---

## 7.1 Required Boot Packages for PXE/UEFI Provisioning

For successful node provisioning in xCAT, some **boot-related packages must be available on the management node**. These packages provide the necessary boot loaders and images required for both **BIOS and UEFI based systems**.

The important packages include:

| Package               | Purpose                                              |
| --------------------- | ---------------------------------------------------- |
| `grub2-efi-x64`       | Provides the GRUB2 bootloader for UEFI-based systems |
| `shim`                | Secure boot loader used in UEFI environments         |
| `ipxe-bootimages-x86` | Provides iPXE boot images used for network booting   |

---

### Install Required Boot Packages

If these packages are not already installed, install them using:

```bash
dnf install grub2-efi-x64 shim ipxe-bootimages-x86 -y
```

or

```bash
yum install grub2-efi-x64 shim ipxe-bootimages-x86 -y
```

---

### Why These Packages Are Important

During PXE boot, compute nodes require a **boot loader compatible with their firmware type**.

| System Type  | Boot Loader Used |
| ------------ | ---------------- |
| Legacy BIOS  | pxelinux / iPXE  |
| UEFI Systems | GRUB2 EFI + shim |

The `ipxe-bootimages-x86` package provides the **network boot images**, while `grub2-efi-x64` and `shim` enable **UEFI-based node booting**.

These components ensure that both **legacy BIOS nodes and modern UEFI nodes** can boot successfully during cluster provisioning.

## 7.2 Purpose of Boot Packages

These packages enable compute nodes to boot depending on the system firmware type.

System Type	Boot Method
Legacy BIOS Systems	PXE / iPXE boot
UEFI Systems	GRUB2 EFI with shim

The ipxe-bootimages-x86 package provides the network boot images, while grub2-efi-x64 and shim support UEFI-based node booting.

These components ensure that both BIOS and UEFI based compute nodes can boot successfully during the provisioning process.

# 8. OS Image Import and Node Provisioning

After completing the configuration of services such as DHCP, TFTP, and required boot packages, the next step in the cluster setup is to import the operating system image and provision compute nodes.

In xCAT, node provisioning is performed by importing the OS installation media into the management node and configuring compute nodes to boot through PXE.

The overall provisioning workflow includes the following steps:

| Step                 | Description                                           |
| -------------------- | ----------------------------------------------------- |
| Import OS Image      | Copy the OS installation media to the management node |
| Define Compute Nodes | Add node definitions in the xCAT database             |
| Configure Boot State | Set nodes to boot in installation mode                |
| Start Installation   | Begin OS installation through PXE boot                |

---

# 8.1 Importing the Operating System Image

The operating system installation files must be copied to the management node so that compute nodes can access them during installation.

Insert the OS installation media (ISO or mounted image) and use the following command:

```bash
copycds <path-to-mounted-iso>
```

Example:

```bash
copycds /mnt
```

This command imports the OS image into the xCAT installation directory, typically:

```
/install
```

During this process, xCAT automatically extracts the installation files and prepares them for network-based deployment.

---

# 8.2 Verify Imported OS Images

After importing the operating system, verify that the image has been successfully added.

Use the following command:

```bash
lsdef -t osimage
```

Example output:

```
rhels9.2-x86_64-install-compute
```

This confirms that the operating system image is available for provisioning compute nodes.

---

# 8.3 Define Compute Nodes

Before provisioning nodes, each compute node must be defined in the xCAT database.

Use the following command:

```bash
mkdef -t node cn001 groups=compute ip=172.18.0.101 mac=XX:XX:XX:XX:XX:XX
```

Explanation:

| Parameter | Description                               |
| --------- | ----------------------------------------- |
| `cn001`   | Name of the compute node                  |
| `groups`  | Node group classification                 |
| `ip`      | IP address assigned to the node           |
| `mac`     | MAC address of the compute node interface |

Verify node definition:

```bash
lsdef cn001
```

---

# 8.4 Set Node Boot State

To prepare the compute node for installation, configure the boot state using the `nodeset` command.

```bash
nodeset cn001 osimage=<osimage-name>
```

Example:

```bash
nodeset cn001 osimage=rhels9.2-x86_64-install-compute
```

This command configures the compute node to boot into installation mode using the specified OS image.

---

# 8.5 Start Node Provisioning

After setting the boot state, start the provisioning process using:

```bash
rinstall cn001
```

This command initiates the remote installation of the operating system on the compute node.

During this process:

1. The node sends a **DHCP request**
2. DHCP provides **network configuration and boot information**
3. The node downloads boot files using **TFTP**
4. Installation files are accessed from the **HTTP server**
5. The operating system installation begins automatically

---

# 8.6 Monitor Provisioning Process

You can monitor the provisioning process using the following commands:

Check node status:

```bash
nodestat cn001
```

Check xCAT logs:

```bash
tail -f /var/log/xcat/cluster.log
```

These logs help identify provisioning progress or troubleshoot installation issues.

---

# 8.7 Verify Node After Installation

Once the installation is complete, verify that the compute node is accessible.

Test connectivity:

```bash
ping cn001
```

Check remote command execution:

```bash
rpower cn001 stat
```

If the node responds successfully, the provisioning process has completed correctly.

# 9. xCAT Command Cheat Sheet

This section provides commonly used commands in xCAT for managing cluster configuration, defining nodes, and verifying settings.

---

# 9.1 `chdef` Command (Change Definition)

The `chdef` command is used to **modify existing objects** in the xCAT database such as nodes, networks, or site configurations.

### Syntax

```bash
chdef <object-name> <attribute>=<value>
```

### Common Examples

**Set DHCP interface**

```bash
chdef -t site dhcpinterfaces="eth1"
```

**Define provisioning network**

```bash
chdef -t network netname=provnet net=172.18.0.0 mask=255.255.255.0 gateway=172.18.0.1 mgtifname=eth1
```

**Configure DHCP IP range**

```bash
chdef -t network provnet dynamicrange=172.18.0.100-172.18.0.200
```

**Modify node IP**

```bash
chdef cn001 ip=172.18.0.101
```

**Assign node to group**

```bash
chdef cn001 groups=compute
```

---

# 9.2 `mkdef` Command (Make Definition)

The `mkdef` command is used to **create new objects** in the xCAT database such as compute nodes or node groups.

### Syntax

```bash
mkdef -t node <node-name> <attribute>=<value>
```

### Common Examples

**Create a compute node**

```bash
mkdef -t node cn001 ip=172.18.0.101 mac=AA:BB:CC:DD:EE:FF groups=compute
```

**Create multiple nodes**

```bash
mkdef -t node cn[001-010] groups=compute
```

**Create a node group**

```bash
mkdef -t group compute
```

---

# 9.3 `lsdef` Command (List Definition)

The `lsdef` command is used to **display object definitions** stored in the xCAT database.

### Syntax

```bash
lsdef <object-name>
```

### Common Examples

**View specific node configuration**

```bash
lsdef cn001
```

**List all defined nodes**

```bash
lsdef -t node
```

**List network configuration**

```bash
lsdef -t network
```

**List OS images**

```bash
lsdef -t osimage
```

**Check xCAT version**

```bash
lsdef -v
```

Example output:

```
lsdef - Version 2.17.0
```

---

# 9.4 Useful Supporting Commands

| Command            | Purpose                              |
| ------------------ | ------------------------------------ |
| `tabdump site`     | View site configuration              |
| `makedhcp -n`      | Generate DHCP configuration          |
| `makedhcp -a`      | Apply DHCP configuration             |
| `nodeset`          | Set node boot state                  |
| `rinstall`         | Start remote installation            |
| `nodestat`         | Check node provisioning status       |
| `xcatprobe xcatmn` | Verify management node configuration |


---

# 10. Node Provisioning

Explain PXE boot installation.

Steps:

1. Node boots
2. DHCP gives IP
3. TFTP loads boot file
4. OS installation starts

---

# 11. Useful xCAT Commands

Example table:

| Command | Purpose |
|------|------|
| nodels | list nodes |
| lsdef | show node definitions |
| rpower | power control |
| nodeset | set boot method |
| updatenode | update nodes |

---

# 12. Troubleshooting

Include real problems you faced.

Example:

### PXE Boot Failure

Check:

```
systemctl status dhcpd
systemctl status tftp
systemctl status httpd
```

### Boot Files Missing

Check:

```
/tftpboot
```

---

# 13. GUI Access (Optional)

Explain xCAT Web UI if you installed it.

Explain:

- how to access
- port
- login

---

# 14. Conclusion

Explain what you achieved.

Example:

- Successfully configured xCAT environment
- Enabled PXE boot
- Deployed compute nodes automatically

---

# Pro Tip (Important)

Do NOT just write commands.

Write like this:

### Example

Step: Start TFTP Service

```
systemctl enable tftp
systemctl start tftp
```

Explanation:

TFTP service is used to provide boot files to compute nodes during PXE boot.

---

# Optional Advanced Sections (Very Good for HPC)

You can add later:

- Diskless nodes
- GPU node provisioning
- InfiniBand configuration
- Integration with Mellanox Unified Fabric Manager

If you want, I can also help you create a **complete professional xCAT documentation template (MkDocs ready)** so you can directly paste it into your docs.

It will look like **real HPC deployment documentation**.
