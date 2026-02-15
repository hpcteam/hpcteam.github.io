# OpenPBS 23.06.06 

**Source Installation on Rocky Linux 9**

This document describes how **I installed OpenPBS 23.06.06 from source**, configured all required services, enabled compute functionality, and validated the setup by running a sample batch job.
This document was **created by me as part of my self-learning to improve my skills in HPC cluster administration**.

---

## 1. Download and Extract OpenPBS Source

```bash
tar xf openpbs-23.06.06.tar.gz
cd openpbs-23.06.06
ls
```

![autogen.sh output](/assets/OpenPbs23/6.png)

### Why this step is required

* Extracts the OpenPBS source archive
* Moves into the source directory
* Prepares the system for building OpenPBS from source

---

## 2. Install Required Dependencies

```bash
sudo dnf install -y gcc make rpm-build libtool hwloc-devel \
libX11-devel libXt-devel libedit-devel libical-devel \
ncurses-devel perl postgresql-devel postgresql-contrib \
python3-devel tcl-devel tk-devel swig expat-devel \
openssl-devel libXext libXft autoconf automake gcc-c++ cjson-devel
```

### Why are these packages required?

* Compiler tools (`gcc`, `make`, `gcc-c++`) are required to build OpenPBS binaries
* Development libraries are needed for scheduler, server, and MOM components
* PostgreSQL development packages are required for database integration
* Missing packages will cause `configure` or `make` to fail

![Source directory](/assets/OpenPbs23/1.png)

---

## 3. Install and Configure PostgreSQL (Required)

OpenPBS uses **PostgreSQL** internally to store:

* Job information
* Scheduler data
* Accounting and history records

---

### 3.1 Install PostgreSQL Packages

```bash
sudo dnf install -y postgresql-server postgresql-contrib
```

![Dependencies](/assets/OpenPbs23/2.png)

---

### 3.2 Initialize PostgreSQL Database

```bash
sudo postgresql-setup --initdb
```

![PostgreSQL install](/assets/OpenPbs23/3.png)

### Why this step is required

* Creates the PostgreSQL data directory
* Mandatory before starting the PostgreSQL service

---

### 3.3 Start and Enable PostgreSQL Service

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
systemctl status postgresql
```

![PostgreSQL initdb](/assets/OpenPbs23/4.png)

![PostgreSQL status](/assets/OpenPbs23/5.png)

### Why this step is required

* Starts PostgreSQL immediately
* Enables PostgreSQL on system boot
* OpenPBS server will not function if PostgreSQL is not running

---

## 4. Generate Build Files

```bash
./autogen.sh
```

![Configure output](/assets/OpenPbs23/7.png)

### Why this step is required

* Generates the `configure` script
* Prepares Makefiles needed to compile OpenPBS
* Warnings during this step are normal

---

## 5. Configure OpenPBS Installation Path

```bash
./configure --prefix=/opt/pbs
```

![Make build](/assets/OpenPbs23/8.png)

### Why this step is required

* Sets `/opt/pbs` as the OpenPBS installation directory
* Keeps OpenPBS separate from system binaries
* Recommended for production and HPC environments

---

## 6. Compile OpenPBS Source Code

```bash
make
```

![Make install](/assets/OpenPbs23/9.png)

### Why this step is required

* Compiles all OpenPBS components:

  * PBS Server
  * PBS Scheduler
  * PBS MOM
* This step may take several minutes

---

## 7. Install OpenPBS

```bash
make install
```

![Postinstall](/assets/OpenPbs23/10.png)

### Why this step is required

* Installs compiled binaries into `/opt/pbs`
* Copies libraries, commands, and supporting files

---

## 8. Run OpenPBS Post-Installation Script

```bash
sudo /opt/pbs/libexec/pbs_postinstall
```

![pbs.conf](/assets/OpenPbs23/11.png)

### Why this step is required

* Creates `/etc/pbs.conf`
* Registers OpenPBS services
* Sets PBS home directory (`/var/spool/pbs`)

---

## 9. Verify PBS Configuration File

```bash
cat /etc/pbs.conf
```

![Permissions](/assets/OpenPbs23/12.png)

### What this configuration means

```text
PBS_SERVER=client
PBS_START_SERVER=1
PBS_START_SCHED=1
PBS_START_COMM=1
PBS_START_MOM=0
PBS_EXEC=/opt/pbs
PBS_HOME=/var/spool/pbs
```

* Node is configured as:

  * PBS Server
  * PBS Scheduler
  * PBS Communication daemon
* MOM is disabled initially

---

## 10. Set Required Permissions

```bash
chmod 4755 /opt/pbs/sbin/pbs_rcp
chmod 4755 /opt/pbs/sbin/pbs_iff
```

![pbs.sh](/assets/OpenPbs23/13.png)
![PBS restart](/assets/OpenPbs23/14.png)

### Why this step is required

* Required for secure job execution
* Prevents permission-related job launch failures

---

## 11. Load PBS Environment

```bash
source /etc/profile.d/pbs.sh
```

![PBS restart](/assets/OpenPbs23/18.png)

### Why this step is required

* Loads PBS commands (`qsub`, `qstat`, `pbsnodes`)
* Required for users to interact with OpenPBS

---

## 12. Start and Restart PBS Services

```bash
/etc/init.d/pbs start
/etc/init.d/pbs restart
```

![PBS restart](/assets/OpenPbs23/17.png)

### Why this step is required

* Starts all PBS components
* Restart is required after configuration changes

---

## 13. Enable PBS MOM on the Node

Edit `/etc/pbs.conf`:

```text
PBS_START_MOM=1
```

![PBS restart](/assets/OpenPbs23/16.png)

Restart PBS:

```bash
/etc/init.d/pbs restart
```

![Enable MOM](/assets/OpenPbs23/19 enable mom.png)

### Why this step is required

* Enables compute (execution) functionality
* Without MOM, jobs cannot run

---

## 14. Add Node to the PBS Pro Configuration:

First, you need to modify the PBS Pro configuration to recognize the new nodes.

Edit the PBS Pro configuration file to include the new node:

```
qmgr -c "create node <node_name>"
```

Replace <node_name> with the hostname of the new node.

![Node details](/assets/OpenPbs23/addingnode01.png)



```bash
pbsnodes -a
```

![Node summary](/assets/OpenPbs23/addingnode02.png)

### What this shows

* Displays summarized node information
* Useful for quick health checks

---

## 15. Create Python Test Program

### File: `print_numbers.py`

```python
#!/usr/bin/env python3
import time

for i in range(1000, 100001):
    print(i, flush=True)
    time.sleep(0.001)
```

![Python script](/assets/OpenPbs23/input.png)

### Why this program is used

* Simple workload to validate job execution
* `flush=True` ensures real-time output

---

## 16. Create PBS Job Script

### File: `run.pbs`

```bash
#!/bin/bash
#PBS -N python_numbers
#PBS -l select=1:ncpus=1
#PBS -l walltime=01:00:00
#PBS -j oe
#PBS -q workq

cd $PBS_O_WORKDIR
python3 print_numbers.py >> python_numbers.log
```

![PBS job script](/assets/OpenPbs23/run.pbs.png)

---

## 17. Submit the Job

```bash
qsub run.pbs
```

![qsub](/assets/OpenPbs23/qsub.png)

### What happens here

* Job is submitted to PBS scheduler
* Scheduler assigns a unique Job ID

---

## 18. Monitor Job Output

```bash
tail -f python_numbers.log
```

![Job output](/assets/OpenPbs23/tail-flog_monitor .png)

### What this confirms

* Job is running correctly
* Output is generated in real time

---

## 19. Enable Job History

```bash
qmgr -c "set server job_history_enable = True"
```

![Enable history](/assets/OpenPbs23/enableing histrory.png)

### Why this step is required

* Keeps completed job records
* Useful for auditing and troubleshooting

Restart PBS:

```bash
/etc/init.d/pbs restart
```

![Restart after history](/assets/OpenPbs23/afterenabling histrory restar the service.png)

---

## 20. Submit Job Again and Verify History

```bash
qsub run.pbs
qstat -x
```

![Job history](/assets/OpenPbs23/histrory achnges submoyed job again and check.png)

### What this confirms

* Completed jobs remain visible
* Job history feature is working correctly

---

##  Final Result

* OpenPBS 23.06 installed from source
* PostgreSQL configured and running
* PBS Server, Scheduler, and MOM active
* Node registered and available
* Python batch job executed successfully
* Live output verified
* Job history enabled

---

### One-Line Summary

> Installed and configured OpenPBS 23.06 from source on Rocky Linux, integrated PostgreSQL, enabled MOM, submitted and monitored batch jobs, and validated job history as part of my self-learning in HPC administration.

---
