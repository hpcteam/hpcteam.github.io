---
title: Understanding Linux Users - UIDs, /etc/passwd, /etc/shadow, and /etc/group
date: 2026-08-07
authors:
  - ayyappa
categories:
  - Linux
tags:
  - users
  - uid
  - passwd
  - shadow
  - linux-administration
---

# Understanding Linux Users: UIDs, /etc/passwd, /etc/shadow, and /etc/group

Today I learned how Linux organizes and manages users under the hood — the different types of users, what happens automatically when a new user is created, and where all that information actually gets stored.

## Types of Users and Their UID Ranges

Every user in Linux has a unique **UID (User ID)** — a number that identifies them to the system. Usernames are just a friendly label; internally, Linux tracks everything using the UID.

| User Type | UID Range | Purpose |
|-----------|-----------|---------|
| **root** | `0` | The administrator account — full control over the system |
| **System users (static)** | `1 – 199` | Reserved for core system services that come pre-installed with the OS (e.g. daemons) |
| **System users (dynamic)** | `200 – 999` | Created automatically when certain packages/services are installed, as needed |
| **Normal/regular users** | `1000 – 60000` | Everyday human user accounts — the people who log in and use the system |

**Why does this matter?** System users exist to run background services securely (so a service doesn't have to run as `root`), while normal users are meant for actual people. Keeping separate ranges makes it easy to tell at a glance what kind of account a UID belongs to.

## What Happens When a New Normal User is Created?

When you create a new regular user (for example using `useradd username`), several things happen **automatically**, without you having to set them up manually:

1. **A default group is created** — usually with the same name as the username, and given a default GID (Group ID) that matches the user's UID.
2. **Password information** is stored securely in `/etc/shadow`.
3. **User account details** are stored in `/etc/passwd`.
4. **Group information** is stored in `/etc/group`.
5. **A home directory is created** — typically at `/home/username`.
6. **A default login shell** is assigned to the user (commonly `/bin/bash`).

## Inside `/etc/passwd`

This file holds basic information about every user account on the system. Each user gets **one line**, split into **7 fields**, separated by colons (`:`).

```
username:x:1001:1001:Ayyappa:/home/username:/bin/bash
```

| Field # | Field Name | Meaning |
|---------|------------|---------|
| 1 | Username | The login name of the user |
| 2 | Password placeholder | Usually shows `x` — the real (encrypted) password is kept in `/etc/shadow`, not here |
| 3 | UID | The user's unique ID number |
| 4 | GID | The user's **primary group** ID |
| 5 | Comment (GECOS) | Extra info — usually the user's full name or a description |
| 6 | Home directory | The path to the user's home folder (e.g. `/home/username`) |
| 7 | Login shell | The shell assigned to the user when they log in (e.g. `/bin/bash`) |

## Inside `/etc/shadow`

This file stores the **actual (encrypted) password** and password-related security settings — and unlike `/etc/passwd`, it can only be read by `root`, which keeps password hashes protected.

Some of the key fields here include:
- Username
- Encrypted password hash
- Last password change date
- Minimum/maximum number of days before the password can/must be changed
- Password warning period and account expiry date

## Inside `/etc/group`

This file stores information about **groups** on the system, one line per group:

```
groupname:x:1001:member1,member2
```

- **Group name**
- **Password placeholder** (rarely used today)
- **GID** (Group ID)
- **List of additional users** who belong to this group (besides those who have it as their primary group)

## Useful Commands to Work with Users

| Command | What it does |
|---------|----------------|
| `useradd username` | Create a new user |
| `passwd username` | Set or change a user's password |
| `usermod` | Modify an existing user's settings |
| `userdel username` | Delete a user |
| `id username` | Show a user's UID, GID, and group memberships |
| `whoami` | Show the current logged-in username |
| `cat /etc/passwd` | View all user account entries |

## Quick Recap

> Linux organizes users using UID ranges: `root` is `0`, system users fall between `1–999`, and normal human users start from `1000`. When a regular user is created, Linux automatically sets up their group, home directory, shell, and stores their info across three key files — `/etc/passwd` (account details), `/etc/shadow` (password security), and `/etc/group` (group membership). Understanding these files is one of the fundamentals of Linux system administration.
