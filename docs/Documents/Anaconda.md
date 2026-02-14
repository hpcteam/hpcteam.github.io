# Anaconda: A Comprehensive Guide

Anaconda is an open-source distribution for Python and R, primarily used in data science, machine learning, and scientific computing. It simplifies package management, environment management, and provides a suite of tools for working on data science projects.

---

## 1. **Anaconda Installation**

To get started with Anaconda, you need to first install it on your system.

### Steps to Install Anaconda:

1. **Download Anaconda**  
   Visit [Anaconda's download page](https://repo.anaconda.com/archive/) and download the appropriate version for your operating system.

2. **Run the Installer**  
   Move the downloaded file to your directory and run the installer. For Linux, the command is:
```
bash Anaconda3-2024.10-1-Linux-x86_64.sh
```
3. **Accept the License**
   Type `yes` to accept the license agreement.

4. **Complete the Installation**
   Once installed, exit the terminal and log out to initialize the base environment.

   ![shcommnad](/assets/anaconda/sh-Anaconda.png)
   ![license-accept](/assets/anaconda/license-accpect.png)
   ![location](/assets/anaconda/Location.png)
   ![installation-finished](/assets/anaconda/installationfinished.png)

---

## 2. **Activating and Deactivating Environments**

### Activating an Environment:

To work within a specific environment, you need to activate it:

```bash
conda activate <env_name>
```

Example:

```bash
conda activate base
```

![conda-activate](/assets/anaconda/conda%20activate.png)

### Deactivating an Environment:

Once done, you can deactivate the environment:

```bash
conda deactivate
```

![conda-deactivate](/assets/anaconda/conda%20deactivate%20-new-env.png)

---

## 3. **Checking Existing Environments**

To see a list of all existing environments on your system:

```bash
conda env list
```

Or:

```bash
conda info --envs
```
![conda-deactivate](/assets/anaconda/envlist.png)
This will show all environments along with their paths.

---

## 4. **Creating New Environments**

### Creating an Environment (With Internet Access):

To create a new environment called `env_conda`, use the following command:

```bash
conda create --name env_conda
```

![creating-env](/assets/anaconda/creating-env-with-internet.png)

### Creating an Environment (Without Internet Access):

If you're working in an offline environment and need to install packages from previously downloaded files, you can create an environment without needing internet access:

```bash
conda create --name env_conda --offline
```
![creating-env-with-pyhton-version](/assets/anaconda/new-env.png)
---

## 5. **Creating an Environment with a Specific Python Version** Using internet

To create an environment with a specific version of Python (for example, Python 3.9):

```bash
conda create --name compute-env python=3.9 
```

![creating-env-with-pyhton-version](/assets/anaconda/creating-new-env-with-pysthonversin-with-internet.png)

For conformation check the Python version in newly creating env

![creating-env-with-pyhton-version](/assets/anaconda/python3-versioncheck.png)

To create an environment with a specific version of Python (for example, Python 3.11):

```bash
conda create --name ai-env python=3.11 --offline
```

![creating-env-with-pyhton-version](/assets/anaconda/creating-env-with-pyhton-version.png)
![creating-env-with-pyhton-version](/assets/anaconda/creating-env-with-pyhton-version-completed.png)
For conformation check the Python version in newly creating env

![creating-env-with-pyhton-version](/assets/anaconda/creating-env-with-pyhton-version-check.png)
---

## 6. **Cloning an Environment**

If you want to clone an existing environment into a new one:

```bash
conda create --name <new_env_name> --clone <existing_env_name>
```

Example:

```bash
conda create --name web-env --clone ai-env
```

![clone-env](/assets/anaconda/cloneing-env.png)

This will create an exact copy of the environment, including all installed packages.

---

## 7. **Removing an Environment and Its Packages**

### Removing Specific Packages from an Environment:

To remove a specific package from an environment, use:

Activate the env 

```bash
pip uninstall <package_name>
```

Example:

```bash
pip uninstall pandas
```
![removing-lib](/assets/anaconda/removing-library.png)
### Removing the Entire Environment:

To remove an environment along with all its data and packages:

```bash
conda remove --name <env_name> --all
```

This will delete the environment `env_conda` and all installed packages within it.
![removing-env](/assets/anaconda/removing%20the%20env%20witll%20all%20packges.png)

---

## 8. **Quick Reference: Conda Commands**

| **Task**                       | **Command**                                    | **Example**                                    |
| ------------------------------ | ---------------------------------------------- | ---------------------------------------------- |
| **Activate environment**       | **`conda activate <env>`**                         | `conda activate py39env`                       |
| **Deactivate environment**     | **`conda deactivate`**                             | `conda deactivate`                             |
| **Create environment**         | **`conda create --name <env>`**                    | `conda create --name env_conda`                |
| **Create with Python version** | **`conda create --name <env> python=3.x`**        | `conda create --name py39env python=3.9`       |
| **Clone environment**          | **`conda create --name <new> --clone <existing>`** | `conda create --name cloneenv --clone py39env` |
| **Delete environment**         | **`conda remove --name <env> --all`**              | `conda remove --name env_conda --all`          |
| **Install package**            | **`conda install -n <env> <package>`**             | `conda install -n py39env pandas`              |
| **Remove package**             | **`conda remove -n <env> <package>`**              | `conda remove -n py39env pandas`               |

