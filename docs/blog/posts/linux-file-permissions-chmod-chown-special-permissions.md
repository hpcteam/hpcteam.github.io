---
title: "Linux File Permissions: chmod, chown, umask, and Special Permissions Explained"
date: 2026-09-16
authors:
  - ayyappa
categories:
  - Linux
tags:
  - permissions
  - chmod
  - chown
  - umask
  - suid
  - sgid
  - sticky-bit
---

# Linux File Permissions 

This post covers Linux file and directory permissions from the ground up — what read/write/execute actually mean for files vs. directories, how to use `chmod` and `chown` with real examples, how default permissions are calculated with `umask`, and how the three special permissions (`SUID`, `SGID`, sticky bit) work with practical scenarios.

## 1. The Three Basic Permissions

Every file and directory carries three permission types, each with a symbol and a numeric value:

| Permission | Symbol | Value |
|---|---|---|
| Read | `r` | 4 |
| Write | `w` | 2 |
| Execute | `x` | 1 |

These apply separately to three categories of users:

| Category | Symbol |
|---|---|
| User / Owner | `u` |
| Group | `g` |
| Other | `o` |

And they can be changed using three operators:

| Operator | Meaning |
|---|---|
| `+` | Add a permission, keep existing ones |
| `-` | Remove a permission, keep the rest |
| `=` | Set the **exact** permission, discarding anything not listed |

## 2. What Permissions Mean on a Directory

| Permission | Meaning | Example |
|---|---|---|
| Read (`r`) | Lets you **list** the contents of the directory | `ls /project` works only if you have read on `/project` |
| Write (`w`) | Lets you **create or delete** files/sub-directories inside it | `touch /project/newfile.txt` needs write on `/project` |
| Execute (`x`) | Lets you **enter** the directory (`cd` into it) and access its contents | `cd /project` needs execute on `/project` |

**Key rule:** without execute (`x`) on a directory, write (`w`) is useless — you can't get inside to create or remove anything. For example, a directory with permissions `rw-------` lets the owner list files but **not** enter or create anything in it, because execute is missing.

## 3. What Permissions Mean on a File

| Permission | Meaning | Example |
|---|---|---|
| Read (`r`) | Lets you **view** the file's contents | `cat notes.txt` needs read on `notes.txt` |
| Write (`w`) | Lets you **modify** the file's contents | `nano notes.txt` and saving needs write |
| Execute (`x`) | Lets you **run** the file as a program/script | `./backup.sh` needs execute on `backup.sh` |

Files are created **without** execute by default. You add it manually depending on the file type — e.g., a Python script or shell script:

```bash
chmod u+x backup.sh
./backup.sh
```

## 4. Reading a Permission String

Permission strings like `rwxr-xr--` are read as three groups of three, left to right — user, group, other — but when you're reasoning about which category "wins" or applying privilege changes with numeric shorthand, always work through them **right to left**: other, then group, then user.

Example: `rwxr-xr--`

| Segment | Category | Permissions |
|---|---|---|
| `rwx` | User | read, write, execute |
| `r-x` | Group | read, execute |
| `r--` | Other | read only |

## 5. Modifying Permissions with `chmod`

### Symbolic method

**Example 1 — remove group's rwx, add other's rwx, remove user's write, and show what changed:**

```bash
chmod g-rwx,o+rwx,u-w report.txt -c
```
Output (via `-c`, which prints only what actually changed):
```
mode of 'report.txt' changed from 0644 (rw-r--r--) to 0087 (----w-rwx)
```

**Example 2 — set an exact permission**, discarding whatever `other` had before:

```bash
chmod o=x script.sh
```
If `script.sh` was `rwxrwxrwx`, it becomes `rwxrwx--x` — other now has **only** execute, nothing else, regardless of what it had previously.

### Numeric (octal) method

Add up the values (4 = read, 2 = write, 1 = execute) for each category:

```bash
chmod 754 script.sh
```
This means:
- User (`7` = 4+2+1) → read, write, execute
- Group (`5` = 4+1) → read, execute
- Other (`4`) → read only

A `0` removes all permissions for that category:

```bash
chmod 700 private.txt
```
Only the owner can read/write/execute; group and other get nothing.

### Recursive changes

```bash
chmod -R 777 /var/www/project
```
This applies `rwxrwxrwx` to the `project` directory **and every file and sub-directory inside it**, recursively.

### Changing permissions for everyone at once

```bash
chmod a+x deploy.sh
```
The `a` (all) flag applies the change to user, group, **and** other in one command — here, giving everyone execute permission on `deploy.sh`.

## 6. Changing Ownership: `chown` and `chgrp`

| Command | Effect |
|---|---|
| `chown ravi:devteam report.txt` | Sets owner to `ravi` **and** group to `devteam` in one step |
| `chown ravi report.txt` | Changes only the owner to `ravi` |
| `chgrp devteam report.txt` | Changes only the group to `devteam` |

**Example:** you have a file owned by `apple:apple` and want it owned by `mango` with group `devteam`:

```bash
chown mango:devteam report.txt
```

You only need `chown` for changing **both** owner and group together — `chgrp` is only needed when changing the group alone.

## 7. Default Permissions and `umask`

Linux's maximum default permissions are:

- **Directories:** `777`
- **Files:** `666` (files never get execute by default, even at maximum)

The actual permission applied at creation time is:

```
Default permission = Maximum permission − umask value
```

With the standard `umask` of `022`:

| Type | Calculation | Result |
|---|---|---|
| Directory | `777 − 022` | `755` (`rwxr-xr-x`) |
| File | `666 − 022` | `644` (`rw-r--r--`) |

**Example — check it yourself:**

```bash
umask          # shows current value, e.g. 0022
mkdir testdir
touch testfile
ls -ld testdir   # drwxr-xr-x
ls -l testfile   # -rw-r--r--
```

**Example — a stricter umask:**

```bash
umask 077
touch secret.txt
ls -l secret.txt   # -rw-------  (only the owner gets any access)
```

### Temporary vs. permanent umask

- Running `umask 077` directly in the terminal only lasts for that **session** — it resets to `022` once you close the terminal or log out.
- To make it **permanent**, add the `umask` line to your `.bashrc`:

```bash
echo "umask 027" >> ~/.bashrc
source ~/.bashrc
```

## 8. Special Permissions

Beyond the standard `rwx` set, three special permissions handle privilege escalation and inheritance:

| Special Permission | Numeric Value | Symbolic Form |
|---|---|---|
| SUID (Set User ID) | 4 | `u+s` |
| SGID (Set Group ID) | 2 | `g+s` |
| Sticky Bit | 1 | `o+t` |

### 8.1 SUID — Set User ID

Normally, running a command uses **your own** privileges. SUID makes a command run with the **file owner's** privileges instead — so a normal user can execute it with elevated rights, without being given full `sudo` access.

**Example:** `useradd` normally requires `sudo` because it needs root privileges to modify `/etc/passwd`. If you set SUID on the `useradd` binary:

```bash
chmod u+s /usr/sbin/useradd
```

...then a normal user can run `useradd newuser` and it will execute **as root** for that one command — without granting that user broad `sudo` access to everything else. (In practice this is rarely done for `useradd` specifically since it's a security-sensitive shortcut, but it illustrates exactly what SUID does — the classic real-world example is `/usr/bin/passwd`, which already ships with SUID so any user can update their own password, which is stored in root-owned `/etc/shadow`.)

You can spot SUID in `ls -l` output as an `s` in place of the owner's execute bit:

```bash
ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 27832 ... /usr/bin/passwd
```

### 8.2 SGID — Set Group ID

**Without SGID:** if directory `/project` is owned by user `apple`, group `mango`, and user `apple` creates a new file inside it, that file gets owner `apple` and group `apple` — the creator's **own** group, not the parent directory's group.

**With SGID applied to the directory:**

```bash
chmod g+s /project
```

Now, **any** new file or sub-directory created inside `/project` — by any user — automatically inherits the **group of the parent directory** (`mango`), instead of the creator's personal group.

**Example scenario:** a shared team directory where multiple users (`apple`, `banana`, `cherry`) each have their own primary group, but all need their files to belong to `devteam` so teammates can access them:

```bash
mkdir /shared/devteam-project
chown :devteam /shared/devteam-project
chmod g+s /shared/devteam-project
```

Now no matter who creates a file inside `devteam-project`, it's automatically group-owned by `devteam` — no manual `chgrp` needed afterward.

### 8.3 Sticky Bit

**The problem it solves:** in a shared directory (like `/tmp`) where many users have write access, any user could delete or rename **any other user's** files, even ones they don't own — because directory write permission alone controls the ability to remove files inside it, regardless of who owns the file itself. This risks permanent, accidental data loss for other users.

**The fix:** applying the sticky bit to a directory means users can still create files inside it, but they can **only delete or rename their own files** — not other users'. Only the file's owner, the directory's owner, or root can remove it.

```bash
sudo chmod +t /home/apple
```
or, using the numeric form (`1` prefix for sticky bit, combined with full `777`):
```bash
sudo chmod 1777 /home/apple
```

You can confirm it's set by checking for a `t` at the end of the permission string:

```bash
ls -ld /home/apple
drwxrwxrwt. 2 apple apple 4096 ... /home/apple
```

**Real-world example:** `/tmp` on almost every Linux system already has the sticky bit set for exactly this reason — many users and processes write temporary files there, and the sticky bit stops one user from deleting another's temp files.

