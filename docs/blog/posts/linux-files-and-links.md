---
title: Understanding Linux Files, Links, and Folders
date: 2026-08-07
authors:
  - ayyappa
categories:
  - Linux
tags:
  - files
  - links
  - filesystem
  - operating-systems
---

# Understanding Linux Files, Links, and Folders 

Today I learned about the different types of files in Linux, what hard links and soft links are, and what all those folders under `/` actually mean. Here's the simple version.

## Types of Files in Linux

In Linux, almost everything is treated as a "file." There are a few types:

**Regular files (`-`)**
Normal files — text files, images, programs, documents. Most files you work with fall here.

**Directory files (`d`)**
These are folders. They hold other files and folders inside them.

### Special Files

**Block files (`b`)**
Represent storage devices like hard disks. Data is read/written in chunks called "blocks."

**Character files (`c`)**
Represent devices that send data one character at a time, like a keyboard or a mouse.

**Link files (`l`)**
Shortcuts that point to another file (more on this below).

**PIPE / FIFO files (`p`)**
Let one program send data directly to another program, like a pipeline.

**Socket files (`s`)**
Let two programs talk to each other, usually used for communication between processes.

## Hard Links vs Soft Links

Links are like different ways of "pointing" to a file.

### Hard Link

- Creates **another name** for the same file — it's not a copy, no new data is created on disk.
- Both the original and the hard link share the **same inode number**.
- An inode is like the file's ID number — it stores the file's real information on disk (like permissions, size, and where the actual data blocks are).
- Since both names point to the exact same data, editing one shows the change in the other too.
- The file's actual data is only removed once **all** hard links to it are deleted (not just the original).
- You can check inode usage with:
  ```
  df -Thi
  ```
  This shows how many inodes are used and how many are still available.
- **Hard links cannot be created for directories** — only for regular files.

### Soft Link (Symbolic Link)

- Creates a **pointer/shortcut** to the original file, kind of like a shortcut icon on a desktop.
- It has its **own, different inode number** from the original file.
- If the original file is edited, the soft link reflects that too, since it's just pointing to the original.
- If the original file is deleted, the soft link breaks (it points to nothing).

### Commands to Create Links

```bash
ln target_file link_name       # creates a hard link
ln -s target_file link_name    # creates a soft (symbolic) link
```

## The Linux Folder Structure

Everything in Linux lives under one main folder: `/` (called "root"). Here's what the important folders under it are for:

| Folder | What it's for |
|--------|----------------|
| `/bin` | Regular user commands and basic programs |
| `/boot` | Files needed to start up (boot) the system |
| `/dev` | Device files — hard disks and other storage/hardware devices |
| `/etc` | Configuration files for applications, so the system knows how to run them |
| `/home` | Personal folders for regular users |
| `/root` | Home folder for the **root** user (the administrator) |
| `/run` | Temporary data created while the system is running (cleared on reboot) |
| `/tmp` | Temporary files — usually auto-deleted after some time |
| `/usr` | Files and data related to installed applications |
| `/var` | "Variable" data — data that keeps changing, like logs, that needs to stay saved |

## The Simple Takeaway

> Linux treats almost everything as a file — even devices and folders. Hard links are like sharing the exact same file under two names, while soft links are just shortcuts pointing to the original. And the whole system is organized under `/`, with each folder having its own clear job.
