# Day 1 Practice Tasks: Introduction to Linux and Red Hat

**Goal:** check that you understand today's ideas, get your lab running, and take your first steps at the Linux command line.

**How to use this page:** try each task on your own first. Then compare with the answer. Tick each box when you are confident.

---

## Part A: Understand the ideas (no computer needed)

Write each answer in your own words, then check against the answers at the bottom.

-  **A1.** In one sentence, what does an operating system do?
-  **A2.** What does "open source" mean? Name two things you are allowed to do with open-source software.
-  **A3.** What is the kernel, and why is it called the "heart" of Linux?
-  **A4.** List the six layers of Linux from the bottom (hardware) to the top (user).
-  **A5.** What does the shell do when you type a command?
-  **A6.** What is the difference between a GUI and a CLI? Which one is used in real-world administration?
-  **A7.** What is a Linux distribution (flavor)? Give three examples.
-  **A8.** Why do companies pay for Red Hat Enterprise Linux when many Linux versions are free?
-  **A9.** Name three products or services included in the Red Hat package besides RHEL.
-  **A10.** Put these certifications in order, and say what each covers: RHCE, RHCSA, RHCA.

---

## Part B: Set up your lab

-  **B1.** Open **rol.redhat.com** and log in with your RHN ID (your email address).
-  **B2.** Go to **My Training** and open the **RH199** course (version 10).
-  **B3.** Find your **lab extension date**. Write it down. (Ignore the class end date.)
-  **B4.** Click **Create** and wait about 3 to 4 minutes for the lab to build.
-  **B5.** Start the lab, then stop it, then start it again, so you know how to control it.
-  **B6.** If you cannot see the lab, write down the instructor name shown in the portal and send it to your instructor.

**Self-check:** you are done when you can start and stop the lab without help.

---

## Part C: Your first commands

Open a terminal in your lab (or in your own RHEL virtual machine) and try these. They only read information, so they are safe.

-  **C1.** Show who you are logged in as:
  ```bash
  whoami
  ```
-  **C2.** Show which operating system you are running:
  ```bash
  cat /etc/os-release
  ```
  Find the line that tells you the name and version (for example, Red Hat Enterprise Linux 10).
-  **C3.** Show the kernel version:
  ```bash
  uname -r
  ```
  This is the "heart" of the system you learned about today.
-  **C4.** Show the machine name:
  ```bash
  hostname
  ```
-  **C5.** Show your current location, then list what is in the top-level directory:
  ```bash
  pwd
  ls /
  ```
-  **C6.** Check the shell you are using:
  ```bash
  echo $SHELL
  ```
  This connects to the "shell" layer from today's architecture.

**Reflection:** for each command above, name the layer it talks to (user, shell, kernel, or hardware). For example, `uname -r` reports on the kernel.

---

## Part D: Explore the two interfaces

-  **D1.** If your lab has a graphical desktop, create a folder by right-clicking (GUI).
-  **D2.** Create another folder from the terminal (CLI):
  ```bash
  mkdir practice
  ls
  ```
-  **D3.** Write two sentences on which method felt faster, and why professionals prefer the CLI for repeated tasks.

---

## Part E: Reading and research

-  **E1.** Visit the Red Hat website and find the page for the **RHCSA (EX200)** exam objectives. List five topics you will need to learn.
-  **E2.** Pick one Linux distribution not in today's notes and write one sentence about what it is used for.
-  **E3.** Draw the six-layer diagram on paper from memory and label each layer with one example.

---

## Self-check answers

**A1.** It acts as the go-between for the user and the hardware, so you can use the machine without controlling it directly.

**A2.** The source code is public. You can use it, modify it, and share your improvements.

**A3.** The kernel is the core of the system. It manages the processor, memory, and devices. If it doesn't load, nothing can run, which is why it is the "heart".

**A4.** Hardware, operating system, kernel, shell, applications, user.

**A5.** It reads your command, passes it to the kernel, and shows you the output.

**A6.** A GUI uses a mouse and windows. A CLI uses typed commands. Real-world administration uses the CLI.

**A7.** A complete system built on the Linux kernel. Examples: Ubuntu, Fedora, Debian, Kali, CentOS, RHEL.

**A8.** Enterprise support, official updates, access to Red Hat repositories, and certified content.

**A9.** Any three of: Ansible (automation), OpenShift (containers), clustering and storage, training and certification.

**A10.** RHCSA (EX200) first, covering manual system administration; RHCE (EX294) second, covering Ansible automation; RHCA last, awarded after five certifications in total.

**C reflection:** `whoami` reports the user; `echo $SHELL` reports the shell; `uname -r` reports the kernel; `cat /etc/os-release` reads a file about the operating system; `hostname` and `ls /` read system information through the kernel.

---

## Quick quiz (answer yes or no)

1. Is Linux closed source?
2. Does the shell talk directly to the hardware?
3. Is RHEL a Linux distribution?
4. Is RHCSA the first Red Hat certification in this path?
5. Does your lab last forever?

**Answers:** 1. No. 2. No, it passes commands to the kernel, which handles the hardware. 3. Yes. 4. Yes. 5. No, it lasts 45 days.

---

## Before the next class

- Make sure your lab starts and stops without errors.
- Reread the Day 1 notes and redraw the layers diagram.
- Think about which virtual machine software you will install on your own computer (VMware Workstation or VirtualBox). Day 2 covers setting up your own RHEL machine.
