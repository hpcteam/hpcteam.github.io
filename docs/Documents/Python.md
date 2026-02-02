---

# **Python Setup & Offline Library Installation Guide for RHEL**

---

## 1. Python Installation on RHEL (Supported Versions)

### RHEL Python Policy (Important)

| RHEL Version | System Python          | Notes                              |
| ------------ | ---------------------- | ---------------------------------- |
| RHEL 7       | Python 2.7 (legacy)    | Use Software Collections or source |
| RHEL 8       | Python 3.6 / 3.8 / 3.9 | `dnf module` based                 |
| RHEL 9       | Python 3.9 / 3.11      | Recommended                        |

⚠️ **Never replace system Python** (used by yum/dnf).
Use **python3**, **venv**, or **pyenv** instead.

---

### Step 1: Update system

```bash
sudo dnf clean all
sudo dnf update -y
```

---

### Step 2: Install Python (Recommended way)

#### RHEL 8 / 9

```bash
sudo dnf install -y python3 python3-pip python3-devel
```

Verify:

```bash
python3 --version
pip3 --version
```

---

### Step 3: Install development tools (required for many packages)

```bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y gcc gcc-c++ make \
                    openssl-devel bzip2-devel libffi-devel \
                    zlib-devel readline-devel sqlite-devel
```

---

### Step 4: Create a user environment (best practice)

```bash
python3 -m venv ~/pyenv
source ~/pyenv/bin/activate
```

Verify:

```bash
which python
python --version
```

---

## 2. Install Python Libraries (Online & Offline)

---

## A. Online Installation (Internet Available)

### Upgrade pip

```bash
pip install --upgrade pip setuptools wheel
```

### Install packages

```bash
pip install numpy pandas matplotlib
```

### Freeze for later offline use

```bash
pip freeze > requirements.txt
```

---

## B. Offline Installation (Air-gapped Systems)

There are **3 enterprise-grade methods** (use depending on scale).

---

# 3. Detailed Offline Library Installation Workflow

---

## Method 1: Download Wheels on Internet Machine (Best Practice)

### Step 1: On internet-connected system

```bash
mkdir ~/offline_pkgs
pip download -r requirements.txt -d ~/offline_pkgs
```

This downloads `.whl` files and dependencies.

---

### Step 2: Copy to offline system

```bash
scp -r offline_pkgs user@offline-server:/tmp/
```

---

### Step 3: Install offline

```bash
pip install --no-index --find-links=/tmp/offline_pkgs -r requirements.txt
```

---

## Method 2: Offline Install from Tarballs (source build)

Use when wheels are not available (HPC, older glibc).

### Download sources

```bash
pip download --no-binary=:all: numpy
```

On offline system:

```bash
pip install numpy-*.tar.gz
```

⚠️ Requires compilers + devel libs.

---

## Method 3: Mirror PyPI Repository (Enterprise / HPC clusters)

### On online system:

```bash
pip install bandersnatch
bandersnatch mirror
```

Copy mirror:

```bash
rsync -av pypi_mirror/ offline:/opt/pypi/
```

Configure pip:

```bash
cat <<EOF > ~/.pip/pip.conf
[global]
index-url = file:///opt/pypi/simple
EOF
```

Install normally:

```bash
pip install numpy pandas
```

---

## 4. Offline OS-Level Dependencies (VERY IMPORTANT)

Many Python libs require OS packages.

Download RPMs online:

```bash
dnf download --resolve gcc gcc-c++ python3-devel
```

Install offline:

```bash
sudo dnf install *.rpm
```

---

## 5. Typical Developer Workstation Example

```
# Install Python
sudo dnf install -y python3 python3-pip
```
```
# Create environment
python3 -m venv ~/dev
source ~/dev/bin/activate
```
```
# Install libs
pip install numpy pandas jupyterlab
```
```
# Test
python - <<EOF
import numpy as np
print(np.arange(5))
EOF

```
---

## 6. Verification Commands

```bash
python -c "import sys; print(sys.version)"
pip list
pip check
```

---

## 7. Common Pitfalls & Fixes

| Problem               | Cause                    | Fix                          |
| --------------------- | ------------------------ | ---------------------------- |
| pip fails building    | Missing gcc/devel libs   | Install Development Tools    |
| SSL errors            | Missing openssl-devel    | Install openssl-devel        |
| NumPy fails           | BLAS/LAPACK missing      | `dnf install openblas-devel` |
| System break          | Replaced /usr/bin/python | NEVER do this                |
| Offline install fails | Missing deps             | Download with `--resolve`    |

---

## 8. Recommended Enterprise Setup (Best Practice)

For **servers, HPC, production**:

* Keep system Python untouched
* Use `venv` or `pyenv`
* Mirror PyPI
* Freeze requirements
* Use wheels only
* Maintain internal repo

---

## 9. Optional: Install pyenv (user-managed Python versions)

```bash
curl https://pyenv.run | bash
pyenv install 3.11.6
pyenv global 3.11.6
```

---

## Summary (What You Should Use)

| Scenario             | Method                    |
| -------------------- | ------------------------- |
| Laptop / workstation | venv + pip                |
| Server               | venv + offline wheels     |
| Air-gapped           | wheel download            |
| HPC / Enterprise     | PyPI mirror               |
| CI/CD                | requirements.txt + wheels |

---

