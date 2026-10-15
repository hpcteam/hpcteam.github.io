# Day 2: Build Your Own RHEL Practice Lab

**Course:** RHCSA (RH124 + RH134)
**Level:** Beginner
**What you will do today:** install a hypervisor, create a virtual machine, install Red Hat Enterprise Linux (RHEL) 10, and connect to it from a terminal over SSH.

**Why this day matters:** your official Red Hat lab lasts only 45 days. A virtual machine on your own computer lets you practice for as long as you need, at any hour, with no time limit.

---

## 1. The big picture

You will build this stack on your computer:

```
Your laptop (Windows)
  └─ VMware Workstation or VirtualBox   ← the "hypervisor"
       └─ Virtual machine (VM)
            └─ RHEL 10                   ← what you practice on
```

| Term | Meaning |
|---|---|
| **Hypervisor** | Software that lets you run a second "computer" inside your computer |
| **Virtual machine (VM)** | The pretend computer, with its own disk, memory, and processors |
| **ISO file** | A disk image of the RHEL installer, used instead of a physical DVD |
| **SSH** | A secure way to log in to another machine over the network |

**Before you start, you need:**

- A laptop with enough free memory (at least 8 GB total RAM is comfortable, since the VM will use 2 to 4 GB) and free disk space (about 20 GB for the VM)
- A stable internet connection
- Your Red Hat account (for downloading RHEL)

---

## 2. What to download

1. **A hypervisor.** Choose one:
   - **VMware Workstation**, or
   - **Oracle VirtualBox**
2. **The RHEL ISO.** Download it from your Red Hat account. The demo used **RHEL 10.1**.

Your instructor can share the VMware download link if you post your email in the class chat.

---

## 3. Install VMware Workstation

1. Run the installer you downloaded.
2. Click **Next** through the wizard and let it finish.

**If an older version of VMware is already installed:**

- The installer offers **Change**, **Repair**, or **Remove**.
- In the demo, Repair did not work because the installed version was old.
- Fix: **uninstall the old version**, then install the newer one.

---

## 4. Create the virtual machine

Open VMware Workstation and follow these steps.

1. Click **Create a New Virtual Machine**.
2. Choose **Installer disc image file (ISO)** and browse to your RHEL ISO.
3. Choose the guest operating system:

   | If you are installing | Choose "Linux" or "Other" | Version |
   |---|---|---|
   | **RHEL 10** | Other | Other 64-bit |
   | **RHEL 9** | Linux | Red Hat Enterprise Linux 9 64-bit |

   (VMware's list may not name RHEL 10 yet, which is why RHEL 10 uses "Other".)
4. **Name** the VM, for example `yourname-vm`.
5. Create a dedicated **folder** for it, for example a `VMware` folder under Documents, and choose it as the location. Everything about the VM is stored there.
6. Choose **Store virtual disk as a single file**.
7. **Disk size: set it to 20 GB.** A smaller disk (around 8 GB) can make the installer fail at the "installation destination" step.
8. Click **Customize Hardware**:

   | Setting | Value |
   |---|---|
   | Memory | 2 GB (4 GB if your laptop has room) |
   | Processors | 2 |

9. Click **Close**, then **Finish**.
10. Click **Play virtual machine**.

---

## 5. Install RHEL

The installation takes about **30 to 45 minutes**, depending on your laptop and network speed. Don't rush it.

1. At the boot menu, choose **Install Red Hat Enterprise Linux 10.1** and press **Enter**.
2. Choose the language: **English (India)**, then **Continue**.
3. **Software selection:**
   - Choose **Server with GUI** for now, because it is friendlier while you are learning.
   - Later, **Server** (no GUI) is the usual real-world choice.
4. **Installation destination:** select the virtual disk and click **Done**. If asked again, click **Done** once more.
5. **Root account:** enable it and set a password. The class used `redhat`. Click **Done** twice, because the installer warns that the password is weak.
6. **User creation:** create a regular user with a password. The class used `redhat` here too. Click **Done** twice for the same reason.
7. Click **Begin Installation**, wait for it to finish, then **Reboot**.

**About the simple password:** the instructor used the same password for everyone so he can troubleshoot your problems more easily. That is fine for a private practice VM that nobody else can reach. **Never use a password like this on a real server.**

---

## 6. First boot

- A first-time configuration screen may appear. You can skip it.
- Open the **Terminal**.
- A message may say the system is **not registered**. That is normal for now. Registration connects your system to your Red Hat subscription (needed to install packages). The next session covers it.

---

## 7. Official lab vs your own VM

| | Official Red Hat lab | Your own VM |
|---|---|---|
| Lasts | 45 days | As long as you keep it |
| Matches the exam environment | Yes | Close, but not identical |
| Good for | Course exercises | Extra practice, experiments |
| Extra tools | Limited to what the lab provides | Docker, single-node Kubernetes, Terraform, and so on (your subscription allows this) |

Use both: practice in the official lab for exam-style tasks, and use your own VM for anything extra.

The instructor will not demo OpenShift setup, because it takes too long.

---

## 8. Connect to your VM with SSH

The VM console and the official lab can feel slow. Logging in over **SSH** from PowerShell or PuTTY is faster, and it lets you copy and paste easily.

### 8.1 Find the VM's IP address

Inside the VM terminal:

```bash
hostname -I
```

Note the first address shown, for example `192.168.xxx.xxx`.

### 8.2 Connect

From PowerShell (or PuTTY) on your laptop:

```bash
ssh root@<IP-address>
```

Enter the root password when asked.

### 8.3 If login fails

The error may mention **password authentication** or **port 22**. By default, RHEL does not let `root` log in with a password over SSH. For a **practice VM only**, you can change that:

1. Open the file as root:
   ```bash
   vim /etc/ssh/sshd_config
   ```
2. Find `PermitRootLogin`, remove the `#` in front if it is there, and set it to `yes`:
   ```
   PermitRootLogin yes
   ```
3. Find `PasswordAuthentication` and make sure it is `yes` (remove any `#` in front):
   ```
   PasswordAuthentication yes
   ```
4. Save and exit (`:wq` in Vim).
5. Restart the SSH service:
   ```bash
   systemctl restart sshd.service
   ```
6. Try `ssh root@<IP-address>` again.

**Tips:**

- The instructor mentioned specific line numbers (about 14, 40, and 65). They differ between versions, so search for the option names instead.
- RHEL may also read extra settings from files in `/etc/ssh/sshd_config.d/`, and those can override the main file. To see what is actually in effect:
  ```bash
  sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'
  ```
- If your login still fails, check that you typed the password correctly. In the demo, the first attempt failed only because of a mistyped password.

### 8.4 Confirm you are on RHEL

```bash
cat /etc/os-release
```

You can now run all your practice commands in this SSH window. You can also power the VM off from VMware.

**Security note:** allowing root login with a password is convenient for practice but unsafe on a real or internet-facing server. Real systems use a normal user, SSH keys, and `sudo`.

---

## 9. Exam preparation advice from the instructor

The session included advice for a participant planning to take the exam in about three weeks.

- **Practice about 2 hours a day.** That is enough to be exam-ready in about 20 days.
- Watch the shared exam-preparation video (about 2 hours) while you practice.
- Success depends on two things:
  1. **The environment:** practice in the lab until the setup feels familiar.
  2. **Typical errors:** learn what usually goes wrong and how to fix it.
- If you get stuck, take a **screenshot** and message the instructor for a solution.

---

## 10. Common problems and fixes

| Problem | Likely cause | Fix |
|---|---|---|
| Installer fails at "installation destination" | Virtual disk too small | Delete the VM and recreate it with a **20 GB** disk |
| VMware installer shows Change / Repair / Remove | An older version is installed | Uninstall it, then install the new version |
| `ssh` says password or port 22 authentication failed | Root password login is off | Edit `sshd_config` (section 8.3) and restart `sshd` |
| `ssh` says "permission denied" even after the fix | Mistyped password | Retype carefully; the demo's first failure was a typo |
| Installation seems stuck | It is just slow | Wait; 30 to 45 minutes is normal |

---

## 11. Key takeaways

1. A **hypervisor** runs a virtual machine inside your computer.
2. Your own VM gives you unlimited practice after the 45-day lab ends.
3. Create the VM with **2 CPUs, 2 to 4 GB of RAM, and a 20 GB disk**.
4. Install **Server with GUI** while learning.
5. Use **SSH** from PowerShell or PuTTY for a faster, easier terminal.
6. Simple passwords and root SSH login are acceptable **only** on a private practice VM.

**Next up:** registering your system with Red Hat so you can install and update packages.
