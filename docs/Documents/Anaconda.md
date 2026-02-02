# Conda Environment Management Guide with Examples


This guide explains **how to install Anaconda** and manage **Conda environments** with practical examples.

## Installing Anaconda

1. Download Anaconda from:  
   [https://repo.anaconda.com/archive/](https://repo.anaconda.com/archive/)

2. Install Anaconda:
```
   # Move the downloaded file to the installation directory

   bash Anaconda3-2024.10-1-Linux-x86_64.sh
   
   # Accept license
   # Type 'yes'

   # Exit after installation
   exit
```


3. Log out and log back in to initialize the base environment.

## Configuring Proxy (Optional)

If behind a proxy, edit `.condarc`:

```bash
vi /home/<username>/anaconda3/.condarc
```

Add:

```yaml
proxy_servers:
  http: http://user:password@proxyaddress:port
  https: https://user:password@proxyaddress:port
```

## Creating Conda Environments

### 1. Basic Environment Creation

```bash
# Activate base environment
source /opt/anaconda3/bin/activate

# Create a new environment called 'env_conda'
conda create --name env_conda
```

**Example output:**
```
Proceed ([y]/n)? y
```

### 2. Creating Environment with Specific Python Version

```bash
# Create 'py39env' with Python 3.9
conda create --name py39env python=3.9
```

**Example output:**
```
Packages to be installed: python-3.9.18, pip-23.2.1, setuptools-68.0.0
Proceed ([y]/n)? y
```

## Activating and Deactivating Environments

```bash
# Activate 'env_conda'
conda activate env_conda

# Deactivate current environment
conda deactivate
```

**Optional:** Auto-activate environment at login

```bash
vi ~/.bashrc
# Add:
conda activate env_conda
```

## Cloning an Environment

```bash
# Activate base environment first
source /opt/anaconda3/bin/activate

# Clone 'env_conda' into 'env_conda_clone'
conda create --name env_conda_clone --clone env_conda --offline
```

**Example output:**
```
Cloning environment from /home/<username>/anaconda3/envs/env_conda
```

## Deleting Environments

```bash
# Activate base environment
source /opt/anaconda3/bin/activate

# Delete environment 'env_conda'
conda remove --name env_conda --all
```

**Example output:**
```
Remove all packages in environment
/home/<username>/anaconda3/envs/env_conda? [y/N]: y
```

## Managing Packages in an Environment

### 1. Install a Package

```bash
# Activate environment
conda activate py39env

# Install numpy
conda install numpy
```

### 2. Remove a Package

```bash
# Remove numpy from 'py39env'
conda remove --name py39env numpy
```

## Quick Reference: Conda Commands

| Task | Command | Example |
|------|---------|---------|
| Activate environment | `conda activate <env>` | `conda activate py39env` |
| Deactivate environment | `conda deactivate` | `conda deactivate` |
| Create environment | `conda create --name <env>` | `conda create --name env_conda` |
| Create with Python version | `conda create --name <env> python=3.9` | `conda create --name py39env python=3.9` |
| Clone environment | `conda create --name <new> --clone <existing>` | `conda create --name cloneenv --clone py39env` |
| Delete environment | `conda remove --name <env> --all` | `conda remove --name env_conda --all` |
| Install package | `conda install -n <env> <package>` | `conda install -n py39env pandas` |
| Remove package | `conda remove -n <env> <package>` | `conda remove -n py39env pandas` |
```

***