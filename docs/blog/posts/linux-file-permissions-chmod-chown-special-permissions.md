---
title: "Linux File Permissions: chmod, chown, and Special Permissions Explained"
date: 2026-09-11
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
---

# Linux File Permissions 
This post covers Linux file and directory permissions from the ground up — read/write/execute for files vs. directories, how to use `chmod` and `chown`, default permissions via `umask`, and the special permissions `SUID`, `SGID`, and the sticky bit.

## The Three Basic Permissions

Every file and directory in Linux carries three permission types, each with a symbol and a numeric value:

| Permission | Symbol | Value |
|---|---|---|
| Read | `r` | 4 |
| Write | `w` | 2 |
| Execute | `x` | 1 |

These permissions apply to three categories of users:

| Category | Symbol |
|---|---|
| User / Owner | `u` |
| Group | `g` |
| Other | `o` |

## What Permissions Actually Mean

Permissions behave differently depending on whether they're applied to a **file** or a **directory**.

### On a Directory

| Permission | Meaning |
|---|---|
| Read (`r`) | Lets you **list** the files inside the directory |
| Write (`w`) | Lets you **create or remove** files and sub-directories inside it |
| Execute (`x`) | Lets you **enter** the directory and access its contents (files and sub-directories) |

**Important:** without execute permission on a directory, write permission is effectively useless — you can't actually get inside the directory to create or remove anything.

### On a File

| Permission | Meaning |
|---|---|
| Read (`r`) | Lets you **view** the contents of the file |
| Write (`w`) | Lets you **add or remove** content in the file |
| Execute (`x`) | Lets you **run** the file as a program or script |

By default, files are created **without** execute permission. It needs to be added manually depending on what the file is meant to do — for example, Python scripts or shell scripts.

## Reading Permission Strings

Permissions are always read from **right to left** when interpreting privilege levels — other, then group, then user — and each triplet (`rwx`) maps to user, group, and other respectively.

## Modifying Permissions with `chmod`

### Symbolic Method (`+`, `-`, `=`)

| Operator | Effect |
|---|---|
| `+` | Adds the specified permission, keeping existing ones intact |
| `-` | Removes the specified permission, keeping the rest intact |
| `=` | Sets the **exact** permission, removing any others not listed |

Example — remove read/write/execute from group, add all to other, remove write from user, and show what changed:

```bash
chmod g-rwx,o+rwx,u-w file -c
```

The `-c` flag prints only the changes that were actually made — showing the previous and new permissions.

Example — set the **exact** permission on other to execute-only, discarding whatever it had before:

```bash
chmod o=x filename
```

### Numeric (Octal) Method

Using numbers `4` (read), `2` (write), and `1` (execute), you can combine them to set permissions directly. A value of `0` removes all permissions for that category, overriding whatever was previously set.

### Recursive Changes

```bash
chmod -R 777 directoryname
```

This applies the permission change **recursively** — to the directory itself and every file and sub-directory inside it.

### Changing Permissions for All User Types

```bash
chmod a+x filename
```

The `a` flag targets **all** categories at once — user, group, and other — granting execute permission to everyone in one command.

## Changing Ownership: `chown` and `chgrp`

| Command | What it does |
|---|---|
| `chown user1:group2 filename` | Changes both the owner and group of a file/directory |
| `chown user2 filename` | Changes only the owner |
| `chgrp group3 filename` | Changes only the group |

`chown` is the only command needed when you want to change **both** the user and group ownership at once — you don't need `chgrp` separately in that case.

## Default Permissions and `umask`

Linux defines maximum default permissions as:

- **Directories:** `777`
- **Files:** `666`

The actual default permission applied when a file or directory is created is calculated as:

```
Default permission = Maximum permission − umask value
```

With the standard `umask` of `022`:

- Directories: `777 − 022 = 755`
- Files: `666 − 022 = 644`

### Temporary vs. Permanent umask Changes

- Changing `umask` directly in the terminal only affects the **current session** — it resets to the default `022` once the terminal is closed or the user logs out.
- To make the change **permanent**, edit the user's `.bashrc` file, set the desired `umask` value there, and then source the file (`source ~/.bashrc`) to apply it.

## Special Permissions

Beyond the standard `rwx` set, Linux has three special permissions:

| Special Permission | Numeric Value | Symbolic Form |
|---|---|---|
| SUID (Set User ID) | 4 | `u+s` |
| SGID (Set Group ID) | 2 | `g+s` |
| Sticky Bit | 1 | `o+t` |

### SUID — Set User ID

When SUID is applied to a command/executable, it lets a normal user run that command **as if they were the file's owner** — without needing `sudo`.

**Example:** the `useradd` command normally requires `sudo` privileges to run. If SUID is set on the `useradd` binary, normal users can create new users **without** being granted full `sudo` access — instead of handing out broad admin rights, you selectively elevate just that one command.

### SGID — Set Group ID

Normally, when a file or directory is created inside another directory, it inherits ownership from the **user who created it** — for example, if the parent directory's owner is `apple` and group is `mango`, a new file created inside it by user `apple` would typically get user `apple` and group `apple`.

When **SGID** is applied to the parent directory, this changes: any new file or directory created inside it inherits the **group ownership of the parent directory** instead of the creating user's own group. So in the example above, new files would get group `mango` (inherited from the parent), rather than the creator's personal group.

This is set with:

```bash
chmod g+s directoryname
```

SGID is especially useful for shared team directories, where everyone's files should automatically belong to the same project group, regardless of who created them.

## Quick Recap

> Linux permissions revolve around three core rights — read, write, execute — applied across user, group, and other, with directories and files interpreting each right differently. `chmod` (symbolic or numeric) controls these permissions, `chown`/`chgrp` control ownership, `umask` sets the defaults for newly created files and directories, and the special permissions `SUID`, `SGID`, and the sticky bit provide fine-grained control over privilege escalation and group inheritance. Together, these form the foundation of Linux file security at the RHCSA level.
