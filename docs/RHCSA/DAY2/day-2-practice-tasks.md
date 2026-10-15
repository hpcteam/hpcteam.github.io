# Day 2 Practice Tasks: Build and Connect to Your RHEL VM

**Goal:** finish today with a working RHEL virtual machine that you can log in to over SSH.

**How to use this page:** work through the tasks in order. Tick each box when it is done and verified. Use the "Verify" lines to prove it worked.

---

## Part A: Prepare

- [ ] **A1.** Check your laptop has enough room: at least 20 GB of free disk space and at least 8 GB of total RAM.
- [ ] **A2.** Choose your hypervisor: VMware Workstation or Oracle VirtualBox.
- [ ] **A3.** Download the hypervisor installer.
- [ ] **A4.** Download the RHEL ISO from your Red Hat account. Note the exact version (for example, 10.1).
- [ ] **A5.** Write down where the ISO is saved.

**Verify:** you can open the folder and see the ISO file with a size of several gigabytes.

---

## Part B: Install the hypervisor

- [ ] **B1.** Run the installer and finish the wizard.
- [ ] **B2.** If an older version exists and the installer offers Change / Repair / Remove, uninstall the old one first, then install again.
- [ ] **B3.** Open the hypervisor and confirm it starts without errors.

**Verify:** the main window opens and shows the option to create a new virtual machine.

---

## Part C: Create the VM

Use these settings and tick each one as you set it.

- [ ] **C1.** Create a new virtual machine from an **ISO** and select your RHEL ISO.
- [ ] **C2.** Guest OS: **Other / Other 64-bit** for RHEL 10 (or **Linux / RHEL 9 64-bit** for RHEL 9).
- [ ] **C3.** Name it `yourname-vm`.
- [ ] **C4.** Store it in a dedicated folder (for example `Documents/VMware`).
- [ ] **C5.** Disk size **20 GB**, stored as a **single file**.
- [ ] **C6.** Memory **2 GB** (or 4 GB), processors **2**.
- [ ] **C7.** Finish and power on the VM.

**Verify:** the RHEL boot menu appears with "Install Red Hat Enterprise Linux".

---

## Part D: Install RHEL

- [ ] **D1.** Start the installer and choose **English (India)** as the language.
- [ ] **D2.** Software selection: **Server with GUI**.
- [ ] **D3.** Installation destination: select the virtual disk and click Done.
- [ ] **D4.** Enable the root account and set a password (click Done twice if the installer warns it is weak).
- [ ] **D5.** Create a regular user and set a password.
- [ ] **D6.** Click Begin Installation and wait (30 to 45 minutes).
- [ ] **D7.** Reboot and log in.

**Verify:** you see the Linux desktop or login screen and can open a terminal.

**If the installer fails at the destination step:** check the disk size. If it is too small, delete the VM and recreate it with 20 GB.

---

## Part E: First commands in the VM

Open a terminal in the VM and run each command. Write down what you see.

- [ ] **E1.** Confirm the operating system:
  ```bash
  cat /etc/os-release
  ```
  Expected: the name shows Red Hat Enterprise Linux and your version.
- [ ] **E2.** Find your user name:
  ```bash
  whoami
  ```
- [ ] **E3.** Switch to root (enter the root password):
  ```bash
  su -
  whoami
  ```
  Expected: the second command prints `root`.
- [ ] **E4.** Find your VM's IP address:
  ```bash
  hostname -I
  ```
  Write the address down.
- [ ] **E5.** Notice the "not registered" message if it appears. You do not need to fix it yet.

---

## Part F: Connect over SSH

On your **laptop**, open PowerShell (or PuTTY).

- [ ] **F1.** Try to connect:
  ```bash
  ssh root@<IP-address>
  ```
- [ ] **F2.** If it works, run `cat /etc/os-release` to confirm you are on the VM. Then skip to Part G.
- [ ] **F3.** If it fails with an authentication error, go back to the VM and edit the SSH settings:
  ```bash
  vim /etc/ssh/sshd_config
  ```
  Set these two options (remove any `#` in front):
  ```
  PermitRootLogin yes
  PasswordAuthentication yes
  ```
- [ ] **F4.** Check what is actually in effect:
  ```bash
  sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'
  ```
  Expected: both show `yes`.
- [ ] **F5.** Restart the service:
  ```bash
  systemctl restart sshd.service
  ```
- [ ] **F6.** Try the SSH connection again.

**Verify:** you get a root prompt inside PowerShell or PuTTY, and `cat /etc/os-release` shows RHEL.

---

## Part G: Use the connection

- [ ] **G1.** In the SSH window, create a folder and a file:
  ```bash
  mkdir practice
  echo "hello from ssh" > practice/test.txt
  cat practice/test.txt
  ```
- [ ] **G2.** Copy a command from this page and paste it into the SSH window (right-click pastes in PowerShell and PuTTY).
- [ ] **G3.** Power the VM off from the hypervisor, then start it again, and reconnect over SSH. Note whether the IP address changed. If it did, you will need the new one for SSH.

---

## Part H: Safety habits

- [ ] **H1.** Write down why a simple password such as `redhat` is acceptable only on a private practice VM.
- [ ] **H2.** Write down why root login over SSH with a password is risky on a real server, and what real systems use instead (a regular user, SSH keys, and `sudo`).
- [ ] **H3.** Make sure your VM is not exposed to the internet.

---

## Self-check questions

Answer from memory, then check.

1. What is a hypervisor?
2. Why build your own VM when you already have an official lab?
3. Which guest OS option do you pick for RHEL 10 in VMware?
4. What disk size avoids the installer failure?
5. How do you find the VM's IP address?
6. Which two SSH options may need to be `yes` for root password login?
7. Which command restarts the SSH service?
8. Why should you not use root password login on a real server?

**Answers**

1. Software that runs a virtual computer inside your real computer.
2. The official lab lasts only 45 days; your own VM has no time limit and lets you try extra tools.
3. Other, with Other 64-bit.
4. 20 GB.
5. `hostname -I` inside the VM.
6. `PermitRootLogin` and `PasswordAuthentication`.
7. `systemctl restart sshd.service`.
8. A guessable password plus a powerful account is an easy target; real systems use SSH keys and `sudo` with normal users.

---

## Troubleshooting checklist

If something is not working, check in this order:

1. Is the VM powered on and fully booted?
2. Does `hostname -I` show an address, and are you using the right one?
3. Did you restart `sshd` after changing its settings?
4. Did you type the password correctly?
5. Does `sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'` show `yes` for both?

Still stuck? Take a screenshot and send it to your instructor with a one-line description of what you tried.

---

## Before the next class

- Keep the VM working and take a snapshot of it if your hypervisor supports snapshots (so you can go back if something breaks).
- Read through your shared notes and the presentation.
- Next session: registering your system with Red Hat and installing packages.
