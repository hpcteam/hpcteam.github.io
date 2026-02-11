# Module Management in Clusters

## 1. What is a Cluster
A cluster is a group of computers (nodes) that work together. Many users share the same system, so software versions must be managed carefully.

## 2. What is a Module in a Cluster?
A module is an easy way to load, unload, and switch software environments. Instead of installing software separately for each user, the cluster provides:
- Multiple versions of software (Python, GCC, CUDA, MPI, etc.)
- Users choose what they need using modules

Think of a module like a switch:
- “Turn ON Python 3.10”
- “Turn OFF Python 3.8”

## 3. What Problem Do Modules Solve?
Without modules:
- Software versions conflict
- Environment variables get messy
- One version breaks another

With modules:
- Clean environments
- Multiple versions coexist
- Easy to change setups

## 4. What is a Module File?
A module file is a small configuration file that tells the system:
- Where the software is installed
- What environment variables to set

It usually:
- Updates `PATH`
- Sets `LD_LIBRARY_PATH`
- Sets variables like `PYTHONPATH`, `CUDA_HOME`, etc.

📁 Example location:
```
/usr/share/modules/modulefiles/
/opt/modulefiles/
```

## 5. What Does a Module File Do?
When you run:

```
module load python/3.10
```

The module file:

* Adds Python 3.10 binaries to your `PATH`
* Makes sure libraries are found
* Hides other Python versions

When you unload it:

```bash
module unload python/3.10
```

Everything is cleaned up.

## 6. Very Important Module Commands

```bash
module avail        # See available modules
module load xyz     # Load a module
module unload xyz   # Unload a module
module list         # See loaded modules
module purge        # Unload everything
```

## 7. Simple Analogy 🧠

* **Cluster** → shared kitchen
* **Software** → ingredients
* **Module** → choosing which ingredient set you’re allowed to use
* **Module file** → recipe that tells the kitchen where everything is

## 8. Typical Examples of Modules

* `gcc/11.2`
* `python/3.9`
* `openmpi/4.1`
* `cuda/12.0`

Each one has its own module file.

## 9. Before Creating a Module File

You need two things:

1. Software already installed somewhere
2. Permission to write module files (or do it in your own directory)

📌 Example software install:

```
/opt/apps/myapp/1.0/
├── bin/
├── lib/
└── include/
```

## 10. Decide Module Name & Location

Module files are usually stored like:

```
/opt/modulefiles/myapp/1.0
```

If you don’t have admin access, you can use:

```
$HOME/modulefiles/myapp/1.0
```

## 11. Tell the System Where Your Modulefiles Are

(Only needed for personal modules)

```bash
module use $HOME/modulefiles
```

## 12. Create the Module File

Create the file:

```bash
mkdir -p $HOME/modulefiles/myapp
vi $HOME/modulefiles/myapp/1.0
```

## 13. Basic Module File (Tcl Format)

This is the simplest working module file.

```tcl
#%Module1.0
##
## myapp 1.0 module file
##

proc ModulesHelp { } {
    puts stderr "Loads myapp version 1.0"
}

module-whatis "MyApp version 1.0"

# Set base directory
set root /opt/apps/myapp/1.0

# Update environment
prepend-path PATH $root/bin
prepend-path LD_LIBRARY_PATH $root/lib
```

## 14. Load and Test the Module

```bash
module avail myapp
module load myapp/1.0
which myapp
```

If it works 🎉 you’re done.

## 15. Unload Test

```bash
module unload myapp/1.0
```

Your `PATH` should go back to normal.

## 16. Important Common Commands Inside Module Files

| Command        | What it Does                            |
| -------------- | --------------------------------------- |
| `set`          | Define variables                        |
| `prepend-path` | Add path at the beginning of a variable |
| `append-path`  | Add path at the end of a variable       |
| `setenv`       | Create environment variable             |
| `conflict`     | Prevent version clashes                 |

Example:

```bash
conflict myapp
setenv MYAPP_HOME $root
```

## 17. Common Mistakes 🚨

* Wrong install path
* Forgetting `module use`
* Using `append-path` instead of `prepend-path`
* Not handling version conflicts

## 18. What Is Inside the Module File?

### `%Module1.0`

* **Required header**: Tells the module system: “This file is a module file.” Without this line, the module will not load.

### Comments

* For humans only. Helps document what this module is for. It has no effect on execution.

### `proc ModulesHelp { }`

* **Defines a help procedure**: `proc` = procedure (function in Tcl)
* **ModulesHelp** is a special name recognized by the module system.
* This runs when the user types:

  ```bash
  module help myapp/1.0
  ```

### `module-whatis`

* **Short description**: Displayed when the user runs:

  ```bash
  module avail
  ```

  It provides a one-line summary of the module.

### Set Base Directory

```bash
set root /opt/apps/myapp/1.0
```

* **Defines a variable**: Assigns a value to a variable (`root` is the variable name, `/opt/apps/myapp/1.0` is the installation directory).

### Update Environment

```bash
prepend-path PATH $root/bin
prepend-path LD_LIBRARY_PATH $root/lib
```

* **Modifies `PATH`**: Adds `/opt/apps/myapp/1.0/bin` to the beginning of `PATH`.
* Ensures this version is found before others.
* `$root` is the value of the variable defined earlier.

### Result of Loading the Module

When you run:

```bash
module load myapp/1.0
```

* The `myapp` command becomes available.
* Correct libraries are found.
* The environment is cleanly modified.

When you unload it:

```bash
module unload myapp/1.0
```

* All changes are reversed automatically.

