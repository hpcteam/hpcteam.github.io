
# Installation of Anaconda and Creating Conda Environments

This document describes the step-by-step process to install Anaconda, configure Conda, and manage Conda environments on a Linux system.


## Prerequisites
- Linux OS
- User login access (example user: `user1`)
- Internet access (or offline packages if required)
- Proxy details (if applicable)

---

## Step 1: Login and Download Anaconda

Login to the required user account 

login root or user --user

Download the Anaconda installer from the official source:

* [https://repo.anaconda.com/archive/](https://repo.anaconda.com/archive/)

Choose the appropriate Linux installer, for example:
`Anaconda3-2024.10-1-Linux-x86_64.sh`

---

## Step 2: Install Anaconda

Run the installer:

**bash Anaconda3-2024.10-1-Linux-x86_64.sh**

### Output:

Welcome to Anaconda3 2024.10-1

Do you accept the license terms? [yes|no]
** yes**

Anaconda3 will now be installed into this location:
/home/user1/anaconda3

installation finished.
Do you wish to initialize Anaconda3? [yes|no]
yes

---

## Step 3: Re-login to Activate Base Environment

Logout and login again.

### Verification:

conda info --envs

### Output:

# conda environments:
#
base                  *  /home/user1/anaconda3

The `*` indicates the active **base** environment.

---

## Step 4: Configure Conda and Create Environment

Edit the `.condarc` file:

**vi /home/user1/anaconda3/.condarc**

Example `.condarc` (proxy configuration):

proxy_servers:
  http: http://proxy.example.com:8080
  https: http://proxy.example.com:8080

Create a new Conda environment:

**conda create --name user1env**

### Output:

Collecting package metadata (current_repodata.json): done
Solving environment: done

## Package Plan ##
  environment location: /home/user2/anaconda3/envs/user2env

Proceed ([y]/n)? y

Preparing transaction: done
Executing transaction: done

---

## Step 5: Auto-Activate Conda Environment on Login

Edit `.bashrc`:

vi ~/.bashrc

Add:

**conda activate user2env**

After re-login, verify:

echo $CONDA_DEFAULT_ENV

### Output:

user2env

---

## Step 6: Activate and Deactivate Conda Environment

**Activate environment:**

**conda activate user2env**

### Sample Output:

(user2env) [user2@server ~]$

Deactivate environment:

**conda deactivate**

### Output:

(base) [user2@server ~]$

---

## Creating Conda Environments

### 1. Creating a New Environment (Default Python)

Activate base environment:

**source /home/user1/annaconda3/bin/activate**

Create environment:

**conda create --name <new-env-name>**

### Sample Output:

## Package Plan ##
  environment location: /home/softwares/AI/annaconda3/envs/new-env-name
Proceed ([y]/n)? y
---

### 2. Creating an Environment with a Specific Python Version

**conda create --name <env_name> python=3.11**

#### Explanation:

* Creates a new Conda environment
* Installs **Python 3.11** explicitly
* Avoids compatibility issues with libraries

Example:

**conda create --name py311env python=3.11**

### Sample Output:

## Package Plan ##
  python: 3.11.7
  pip: 23.3

Proceed ([y]/n)? y

Verify Python version:

conda activate py311env
python --version

Python 3.11.7
---
### 3. Cloning an Existing Environment

Activate base environment:

**source /home/user1/annaconda3/bin/activate**

Clone environment:

**conda create --name <new-env-name> --clone <available_env> --offline**

## Output:

Source:      /home/user1/annaconda3/envs/available_env
Destination: /home/user1/annaconda3/envs/new-env-name

Executing transaction: done

> **Note:**
> A **fresh login** is required before creating or cloning environments to ensure correct setup.

---

### 4. Deleting a Conda Environment

Activate base environment:

**source /home/user1/annaconda3/bin/activate**

Delete environment:

**conda remove --name <env-name> --all**

### Sample Output:

Remove all packages in environment /home/.../envs/env-name? [y/N] y
Environment removed.

---

### 5. Removing a Package from an Environment

**conda remove --name <env-name> <lib-name>**

### Sample Output:

The following packages will be REMOVED:
  numpy

Proceed ([y]/n)? y
---

* Install Anaconda from the official source
* Configure `.condarc` for proxy settings
* Create environments with default or specific Python versions
* Clone, activate, deactivate, and delete environments
* Use command outputs to verify each step
---


