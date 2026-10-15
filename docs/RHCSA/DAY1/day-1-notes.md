# Day 1: Introduction to Linux, Red Hat Enterprise Linux, and the RHCSA Path

**Course:** RHCSA
**Section:** Foundations
**Topic:** What Linux is, how it is structured, why Red Hat Enterprise Linux is used, and how to reach your practice lab
**Audience:** Linux beginners through RHCSA exam candidates

---

## 1. Topic Introduction

Linux is a free, open-source operating system used on servers, cloud platforms, network devices, phones, and supercomputers. Red Hat Enterprise Linux (RHEL) is a commercially supported distribution of Linux, and it is the platform used for the Red Hat Certified System Administrator (RHCSA) certification.

Before you can administer a system, you need a clear mental model of what the system is made of. A Linux administrator who understands the difference between the kernel, the shell, and applications can reason about problems instead of memorizing fixes. This section builds that model, explains why Linux is so widely deployed, introduces the Red Hat ecosystem, and shows you how to reach a practice lab.

**Where this knowledge is used in practice**

- Choosing an operating system for a new server and justifying that choice.
- Explaining to a colleague why a command typed in a terminal ends up controlling hardware.
- Deciding which Red Hat certification fits a career path.
- Setting up a safe environment in which to practice administration tasks.

---

## 2. Learning Objectives

After completing this section, you will be able to:

- Define an operating system and explain its role between users and hardware.
- Explain what "open source" means and what it allows.
- Distinguish the Linux **kernel** from a Linux **distribution**.
- Describe the layers of a Linux system and the role of each one.
- Explain the difference between a graphical interface (GUI) and a command-line interface (CLI).
- List major Linux distributions and explain what makes RHEL an enterprise distribution.
- Describe the Red Hat certification path (RHCSA, RHCE, RHCA).
- Run basic read-only commands to identify the operating system, kernel, shell, and user.
- Open the course practice lab and start and stop it.

---

## 3. Core Concepts

### 3.1 Operating system

**Definition:** An operating system (OS) is the software that manages hardware resources and provides services to programs and users.

**How it works:** Hardware (processor, memory, disk, network card) cannot be used directly by an ordinary person. The OS sits between the user and the hardware, accepts requests, and translates them into hardware operations.

```
User  <-->  Operating system  <-->  Hardware
```

**Why it matters:** Every task you perform, such as saving a file, starting a service, or opening a network connection, passes through the operating system.

### 3.2 Open source

**Definition:** Open-source software is distributed with its source code, so anyone may read, use, modify, and redistribute it under the terms of its license.

**What it allows**

- Use the software without paying a license fee for the software itself.
- Study how it works.
- Modify it for your own needs.
- Share improvements.

**Common misunderstanding:** "Open source" does not mean "unsupported". Companies such as Red Hat sell support, tested updates, and certifications built on open-source software.

### 3.3 The kernel

**Definition:** The kernel is the core of the operating system. It controls the processor, memory, devices, and processes, and it is the only part that talks directly to hardware.

**How it works:** Programs ask the kernel to perform work through controlled interfaces. The kernel schedules processes, manages memory, and passes data to and from devices.

**Why it matters:** If the kernel cannot load during boot, nothing else on the system can run.

**Terminology: Linux versus distribution**

| Term | Meaning |
|---|---|
| **Linux** (strict meaning) | The kernel, first created by Linus Torvalds in 1991 |
| **Linux** (everyday meaning) | A complete operating system built around the Linux kernel |
| **Distribution (distro, flavor)** | A packaged operating system that combines the kernel with tools, libraries, a package manager, and applications |

**Common misunderstanding:** "Linux" and "Ubuntu" are not the same thing. Ubuntu is one distribution that uses the Linux kernel.

### 3.4 Linux and Unix

Linux was designed in the tradition of Unix, an earlier family of operating systems. Classic commercial Unix systems were distributed under proprietary licenses, while the Linux kernel is open source. This licensing difference is the most important practical distinction between the two.

> **Note:** Some Unix-like systems, such as the BSD family, are also open source. The statement above applies to classic commercial Unix.

### 3.5 Layers of a Linux system

A Linux system works as a stack of layers. Each layer has one responsibility, and the layers depend on each other.

```
+------------------------------------+
| User                               |
+------------------------------------+
| Applications                       |  terminal, browser, editor
+------------------------------------+
| Shell                              |  command interpreter
+------------------------------------+
| Kernel                             |  core of the OS
+------------------------------------+
| Operating system (whole system)    |
+------------------------------------+
| Hardware                           |  laptop, server, VM
+------------------------------------+
```

| Layer | Role |
|---|---|
| **Hardware** | The physical (or virtual) machine. Without it, nothing runs. |
| **Operating system** | The complete set of system software built around the kernel. |
| **Kernel** | Manages hardware, memory, and processes. |
| **Shell** | Reads the commands you type, asks the kernel to perform them, and shows the result. |
| **Applications** | Programs such as a terminal, web browser, or text editor. |
| **User** | The person or account that works through applications or the shell. |

**Compiler analogy:** In Java, you run `javac` to compile source code and `java` to run it. You give an instruction, another program translates it, and you get a result. The shell plays a similar translating role for Linux commands.

> **Verify:** Different textbooks draw this stack slightly differently, for example placing the kernel directly above the hardware and treating the shell and applications as user space. The idea to remember is that commands flow from the user, through the shell, to the kernel, to the hardware, and results flow back.

### 3.6 The shell

**Definition:** The shell is a program that interprets commands. On RHEL the default shell is **Bash** (the Bourne Again Shell).

**How it works:** You type a command, the shell parses it, starts the requested program (or runs a built-in function), and the kernel does the underlying work. The shell then prints the result.

**Why it matters:** Nearly all system administration on RHEL is done through the shell, and the RHCSA exam is performed at the command line.

### 3.7 GUI and CLI

| Interface | How you work | Typical use |
|---|---|---|
| **GUI** (graphical user interface) | Mouse, windows, menus | Desktops, first steps |
| **CLI** (command-line interface) | Typed commands | Server administration, automation, remote work |

A GUI is convenient while you are learning. The CLI is faster for repeated tasks, works over slow network connections, and can be scripted, so professional administration relies on it.

### 3.8 Why Linux is widely used

| Reason | Explanation |
|---|---|
| **Security** | Multiple independent controls protect resources (see below). |
| **Stability** | Linux systems are known for long uptimes. |
| **Performance** | Efficient use of CPU and memory. |
| **Flexibility** | Runs on a wide range of hardware and can be customized. |
| **Cost** | The software itself is free to obtain. Support and subscriptions are optional services. |

**Layered security controls:** To access a protected resource, a request must pass several independent checks. If any control denies it, access fails.

1. **File permissions:** who can read, write, or execute a file.
2. **Special permissions:** additional modes such as set-user-ID, set-group-ID, and the sticky bit.
3. **Access control lists (ACLs):** fine-grained permissions for specific users or groups.
4. **SELinux (Security-Enhanced Linux):** a mandatory access control system that limits what processes can do.
5. **Firewall:** controls network traffic.

These topics are covered in later sections.

### 3.9 Where Linux is used

- **Servers and cloud platforms:** virtual machines on AWS, Google Cloud, and Microsoft Azure commonly run Linux.
- **Web infrastructure:** a large share of websites are served from Linux machines.
- **Mobile devices:** Android is built on the Linux kernel.
- **Network equipment:** routers, switches, firewalls, and home Wi-Fi routers often run embedded Linux.
- **Embedded systems and supercomputers.**
- **Enterprise IT, including banking back-end systems.**

### 3.10 Linux distributions

Because the source code is open, many organizations build their own distributions.

| Distribution | Notes |
|---|---|
| **Red Hat Enterprise Linux (RHEL)** | Commercially supported enterprise distribution. The focus of this course. |
| **Fedora** | Community project where newer features appear first. |
| **CentOS Stream** | A community distribution that tracks the development of upcoming RHEL releases. |
| **Rocky Linux, AlmaLinux** | Community rebuilds designed to be compatible with RHEL. |
| **Ubuntu, Debian** | Popular Debian-family distributions that use a different package system. |
| **openSUSE** | Another major community distribution. |
| **Kali Linux** | Specialized for security testing. |

> **Note:** The older "CentOS Linux" releases have reached end of life. If you encounter older material that refers to them, treat it as legacy.

Different distributions have different package formats and tools. RHEL and its relatives use RPM packages and the DNF package manager, while Debian-family systems use DEB packages.

### 3.11 What Red Hat provides

Red Hat is an open-source software company. It sells subscriptions that include tested software, updates, and support.

| Product or service | Purpose |
|---|---|
| **Red Hat Enterprise Linux (RHEL)** | The enterprise operating system |
| **Ansible Automation Platform** | Automation of administration tasks across many systems |
| **OpenShift** | A platform for running containerized applications, built on Kubernetes |
| **Clustering and storage products** | High availability and scalable storage |
| **Training and certification** | Official courses and exams |

**Support analogy:** A paid streaming service gives you a support team when something fails, while a free third-party site does not. A Red Hat subscription is similar. It gives you official updates, access to Red Hat repositories, and a support channel.

> **Important:** On a RHEL system, installing and updating packages from Red Hat repositories requires the system to be registered with a valid subscription. Registration is covered in a later section.

### 3.12 Certification path

| Step | Certification | Exam code | Focus |
|---|---|---|---|
| 1 | **RHCSA** (Red Hat Certified System Administrator) | EX200 | Core system administration at the command line |
| 2 | **RHCE** (Red Hat Certified Engineer) | EX294 | Automation with Ansible |
| 3 | Additional specialist exams | Varies | Areas such as OpenShift or virtualization |

**RHCA (Red Hat Certified Architect)** is awarded after earning multiple required certifications. The count and eligible exams can change, so confirm current requirements on Red Hat's website.

**Why automation follows administration:** Creating five users on each of fifty servers by hand is slow and error-prone. Ansible performs the same task on all servers in one run, but you must understand the manual task first.

> **Verify:** Exam codes, required courses, and exam objectives change between RHEL releases. Check the official Red Hat certification pages before you plan your preparation.

---

## 4. Commands and Syntax

The commands in this section only read information. They are safe to run on any lab system.

### Command: `whoami`

**Purpose:** Print the name of the user you are currently logged in as.

**Syntax:**

```bash
whoami
```

**Example:**

```bash
whoami
```

**Explanation:** The command prints one user name. Use it before running administrative commands to confirm whether you are a normal user or `root`.

**Important points:** No privileges are required.

---

### Command: `cat /etc/os-release`

**Purpose:** Display information about the installed operating system.

**Syntax:**

```bash
cat /etc/os-release
```

**Example:**

```bash
cat /etc/os-release
```

**Explanation:** `/etc/os-release` is a text file with `KEY=value` lines. Useful keys include `NAME`, `VERSION`, `ID`, `VERSION_ID`, and `PRETTY_NAME`. The output tells you which distribution and release you are using.

**Important points:**

- The values depend on your installation. Read your own output rather than expecting specific text.
- This command works on most modern Linux distributions.

---

### Command: `uname`

**Purpose:** Print information about the running kernel and system.

**Syntax:**

```bash
uname [options]
```

**Important options:**

| Option | Meaning |
|---|---|
| `-r` | Kernel release |
| `-s` | Kernel name |
| `-m` | Machine hardware architecture |
| `-a` | All available information |

**Example:**

```bash
uname -r
uname -a
```

**Explanation:** `uname -r` reports the version of the kernel that is running now. Administrators check this after kernel updates and when diagnosing hardware or driver problems.

---

### Command: `hostname` and `hostnamectl`

**Purpose:** Show the system's name (and, with `hostnamectl`, more system details).

**Syntax:**

```bash
hostname
hostnamectl
```

**Explanation:** `hostname` prints the machine name. `hostnamectl` prints the hostname together with details such as the operating system and kernel, which is useful as a quick system summary.

---

### Command: `pwd`

**Purpose:** Print the current working directory.

**Syntax:**

```bash
pwd
```

**Explanation:** `pwd` shows where you are in the file system. A normal user's login directory is usually under `/home`, while the `root` user's is `/root`.

---

### Command: `ls`

**Purpose:** List files and directories.

**Syntax:**

```bash
ls [options] [path]
```

**Example:**

```bash
ls /
```

**Explanation:** This lists the top-level directories of the file system. You will study their purposes in a later section.

---

### Command: `echo $SHELL` and `cat /etc/shells`

**Purpose:** Identify your login shell and the shells installed on the system.

**Example:**

```bash
echo $SHELL
cat /etc/shells
```

**Explanation:** `$SHELL` is a variable holding the path of your login shell. `/etc/shells` lists valid login shells. On RHEL, your login shell is normally `/bin/bash`.

---

### Command: `mkdir`

**Purpose:** Create a directory (used here to compare GUI and CLI workflows).

**Syntax:**

```bash
mkdir [options] directory-name
```

**Example:**

```bash
mkdir practice
ls
```

**Explanation:** `mkdir practice` creates a directory named `practice` in the current directory. `ls` lists the current directory so you can confirm it exists.

**Common errors**

| Error message (approximate) | Cause | Fix |
|---|---|---|
| File exists | A file or directory with that name already exists | Choose a different name |
| Permission denied | You lack write permission in the current directory | Move to your home directory with `cd ~` |

---

### Hardware views (optional)

These commands show information about the lowest layer.

```bash
lscpu        # processor details
free -h      # memory in human-readable units
lsblk        # block devices (disks)
```

They require no special privileges and make no changes.

---

## 5. Configuration Files and System Paths

| Path | Purpose |
|---|---|
| `/etc/os-release` | Operating system identification |
| `/etc/shells` | List of valid login shells |
| `/etc` | System-wide configuration files |
| `/home` | Home directories of normal users |
| `/root` | Home directory of the `root` user |

**Inspecting:** Use `cat`, `less`, or `ls -l` to read files. These files are not meant to be edited as part of this section.

**Back up before modifying:** In later sections you will edit configuration files. A good habit is to copy a file before changing it:

```bash
sudo cp /etc/example.conf /etc/example.conf.bak
```

(`example.conf` is a placeholder.)

---

## 6. Step-by-Step Administration Procedures

### Procedure 1: Identify your system

**Goal:** Determine the user, operating system, kernel, and shell of a Linux system.

**Prerequisites:** A terminal on a RHEL (or compatible) system.

**Steps**

1. Open a terminal.
2. Identify the user:
   ```bash
   whoami
   ```
3. Identify the operating system:
   ```bash
   cat /etc/os-release
   ```
4. Identify the kernel:
   ```bash
   uname -r
   ```
5. Identify the shell:
   ```bash
   echo $SHELL
   ```

**Verification:** You can state the user name, distribution name and version, kernel release, and shell path.

**Cleanup:** None. These commands do not change the system.

---

### Procedure 2: Open the course practice lab

**Goal:** Create, start, and stop the hosted practice lab included with your course.

**Prerequisites:**

- A Red Hat account (the RHN ID is the account's email address).
- Access to the course through your subscription.

**Steps**

1. Open `https://rol.redhat.com` in a web browser.
2. Log in with your Red Hat account credentials.
3. Open **My Training** and choose your course (shown in the portal as **RH199**, version 10, in this course).
4. Check the **lab extension date**, which shows how long your lab access lasts. Do not rely on the displayed class end date.
5. Click **Create** to build the lab environment. Allow a few minutes. In this course, creation took about three to four minutes.
6. Use the lab controls to **start** and **stop** the environment as needed.

**Verification:** The lab shows a running state and you can open a terminal in it.

**Rollback:** Stop the lab when you finish a session.

**If the lab does not appear:** Note the instructor or account name displayed in the portal and contact your course provider so access can be arranged.

> **Verify:** Portal layout, labels, and durations can change. Follow what your own portal displays. Course lab access is time-limited (45 days in this course), so plan your practice time. A later section explains how to build your own VM to keep practicing afterward.

---

### Procedure 3: Compare GUI and CLI

**Goal:** Perform the same task in both interfaces.

**Steps**

1. In a graphical desktop, create a folder using the file manager.
2. In a terminal, create another folder:
   ```bash
   mkdir practice
   ```
3. Confirm both exist:
   ```bash
   ls
   ```

**Verification:** Both folders appear. Consider which method scales better to 50 folders.

---

## 7. Practical Examples

### Example 1: Preparing a system inventory

A colleague asks which operating system and kernel a server is running.

```bash
whoami
cat /etc/os-release
uname -r
hostnamectl
```

**Why each step is needed:** `whoami` confirms your identity so you know what privileges you hold. `/etc/os-release` identifies the distribution. `uname -r` reports the running kernel, which can differ from the newest installed kernel until the next reboot. `hostnamectl` collects the same facts in one summary.

### Example 2: Mapping commands to layers

| Command | What it reveals | Layer |
|---|---|---|
| `lscpu` | Processor details | Hardware |
| `uname -r` | Kernel release | Kernel |
| `echo $SHELL` | Login shell | Shell |
| `whoami` | Current account | User |
| `cat /etc/os-release` | Distribution information | Operating system |

---

## 8. Verification and Troubleshooting

| Symptom | Likely cause | Resolution |
|---|---|---|
| `command not found` for a correct command | Misspelled command or missing package | Check spelling; Linux commands are case-sensitive |
| `No such file or directory` | Wrong path or typo | Check the path with `ls`; use `Tab` completion |
| `Permission denied` | Insufficient privileges | Check `whoami`; use `sudo` only for tasks that need it |
| Lab does not appear in the portal | Access not provisioned yet | Contact your course provider with the displayed instructor name |
| Lab is slow to start | Normal build time or network speed | Wait a few minutes before retrying |

**Case sensitivity:** `Cat` and `cat` are different. Commands and file names are case-sensitive.

**Persistent versus temporary:** Commands in this section change nothing permanent, except `mkdir`, which creates a directory that stays until deleted.

---

## 9. Best Practices and Important Notes

- Learn the CLI early, because administration and the RHCSA exam rely on it.
- Run commands as a normal user and use `sudo` or `root` only when needed.
- Read command output carefully rather than assuming results.
- Never practice on production systems. Use a disposable lab.
- Use the official lab for course exercises and your own VM for extra experiments.
- Keep notes of commands that surprise you.

**Common mistakes**

- Treating "Linux" and "Linux distribution" as identical.
- Assuming a free distribution includes enterprise support.
- Practicing only in the GUI.

---

## 10. RHCSA Exam Preparation

**General administration knowledge covered here**

- Navigating a Linux system from a terminal.
- Identifying the operating system, kernel, user, and shell.
- Understanding what RHEL and a subscription provide.

**Preparation habits that help in any hands-on exam**

- Practice daily in a lab, not only reading.
- Use the manual pages and tab completion, and verify each result with a command.

> **Verify:** The RHCSA exam (EX200) is described by Red Hat as a hands-on practical exam. This section's topics are foundation knowledge, and no specific exam objective is claimed here. Always compare your study plan with the current official EX200 objectives on Red Hat's website.

---

## 11. Section Summary

- Linux is an open-source operating system. Strictly, "Linux" is the kernel, and a distribution packages it into a complete system.
- A Linux system is a stack of layers: hardware, operating system, kernel, shell, applications, and user.
- The shell (Bash on RHEL) turns typed commands into kernel requests.
- The CLI is the main tool for administration, while the GUI helps beginners.
- Linux is valued for security, stability, performance, flexibility, and cost.
- RHEL is a supported enterprise distribution. The path begins with RHCSA, then RHCE, and builds toward RHCA.
- A hosted lab is available for a limited time, so plan to build your own VM.

---

## 12. Quick Revision

| Command | Purpose | Notes |
|---|---|---|
| `whoami` | Show current user | Check before using privileges |
| `cat /etc/os-release` | Show OS identity | Values vary by system |
| `uname -r` | Show kernel release | Running kernel |
| `uname -a` | Show all system info | One line |
| `hostname` | Show machine name | |
| `hostnamectl` | System summary | Includes OS and kernel |
| `pwd` | Show current directory | |
| `ls /` | List top-level directories | |
| `echo $SHELL` | Show login shell | Normally `/bin/bash` on RHEL |
| `cat /etc/shells` | List valid shells | |
| `mkdir name` | Create a directory | |
| `lscpu`, `free -h`, `lsblk` | Hardware views | Read-only |

---

## 13. Knowledge Check

1. In one sentence, what does an operating system do?
2. What does "open source" allow you to do?
3. What is the strict meaning of "Linux", and how does a distribution differ?
4. List the six layers of a Linux system from hardware to user.
5. What does the shell do with a command you type?
6. Why do administrators prefer the CLI over the GUI?
7. Name the default shell on RHEL.
8. Which command shows the running kernel release?
9. Name two things a Red Hat subscription provides.
10. Which certification comes first on the path, and what does it cover?

---

## Answer Key

1. It sits between users and hardware, accepting requests and translating them into hardware operations.
2. Read, use, modify, and share the software and its source code under its license.
3. Strictly, Linux is the kernel. A distribution combines the kernel with tools, libraries, a package manager, and applications into a complete system.
4. Hardware, operating system, kernel, shell, applications, user.
5. It interprets the command, asks the kernel (through the programs it starts) to carry it out, and displays the result.
6. It is faster for repeated tasks, works well remotely, and can be scripted.
7. Bash.
8. `uname -r`.
9. Any two of: official updates, access to Red Hat repositories, technical support, certified content.
10. RHCSA (exam EX200), covering core system administration at the command line.