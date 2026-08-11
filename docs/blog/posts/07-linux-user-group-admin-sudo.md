---
title: "Linux User & Group Administration: gpasswd, chage, and sudo"
date: 2026-08-07
authors:
  - ayyappa
categories:
  - Linux
tags:
  - users
  - groups
  - sudo
  - password-aging
---

# Linux User & Group Administration (RHCSA Level)

This post goes deeper into user and group management — covering `useradd` options, `/etc/skel`, `gpasswd`, primary vs. secondary groups, password aging with `chage`, and `sudo`. This is core RHCSA material, explained step by step.

## Why root Doesn't Need a Password to Change Passwords

When a **normal user** changes their own password using `passwd`, the system first asks for their **current password**, to confirm it's really them.

When **root** changes another user's password (`passwd username`), it does **not** ask for that user's current password. This is because root already has full administrative privileges over the whole system — root doesn't need to "prove" it has permission, since it already has permission to everything.

## `useradd` — Creating Users with Options

Running `useradd --help` shows many options that let you control exactly how a user is created:

| Option | What it does |
|--------|----------------|
| `-u UID` | Set a specific User ID |
| `-g GROUP` | Set the user's **primary** group |
| `-G GROUP1,GROUP2` | Add the user to one or more **secondary** groups |
| `-s /path/to/shell` | Set the user's login shell |
| `-d /path` | Set a custom home directory path |
| `-c "comment"` | Add a comment (usually the person's full name) |
| `-m` | Create the user's home directory (and copy default files into it) |
| `-M` | Do **not** create a home directory at all |
| `-p "password"` | Set the password at creation time |

## What is `/etc/skel`?

`/etc/skel` is a **template directory**. Whenever a new user is created **and their home directory is created automatically** (using `-m`, or by default on most systems), Linux copies every file inside `/etc/skel` into the new user's home directory. This is how new users automatically get default config files like `.bashrc` and `.bash_profile`.

**Important exception:** if a user is created with a home directory that already exists (for example, using `-d` to point to an existing folder), the `/etc/skel` files are **not** copied automatically. In that case, you'd need to copy them in manually:

```bash
cp -r /etc/skel/. /home/username/
```

## `groupadd` — Creating Groups

```bash
groupadd groupname
```

This creates a new group. You can also assign a specific GID:

```bash
groupadd -g 1050 groupname
```

## `gpasswd` — Managing Group Membership

`gpasswd` is used to administer groups — mainly to **add or remove users** from a group, without editing `/etc/group` by hand.

| Option | What it does |
|--------|----------------|
| `gpasswd -a username groupname` | Add a **single** user to a group |
| `gpasswd -d username groupname` | Remove a **single** user from a group |
| `gpasswd -M user1,user2,user3 groupname` | **Replace** the entire member list of a group at once |
| `gpasswd -A username groupname` | Set a user as a group **administrator** |
| `gpasswd -r groupname` | Remove a group's password |

### Important detail from testing this out

`-a` and `-d` only accept **one username at a time** — they don't accept a comma-separated list. Trying `gpasswd -a user1,user2,user3 apple` fails, because Linux tries to treat `"user1,user2,user3"` as a single (nonexistent) username.

If you want to set **multiple members at once**, use `-M` instead, which is specifically designed to accept a comma-separated list:

```bash
gpasswd -M user1,user2,user3 apple
```

This was confirmed directly:
```
# gpasswd -a user1,user2,user3 apple
gpasswd: user 'user1,user2,user3' does not exist

# gpasswd -M user1,user2,user3 apple
(works successfully)
```

## Primary Group vs. Secondary (Supplementary) Group

Every user has exactly **one primary group** and can belong to **any number of secondary groups**.

- **Primary group**: Listed in the user's entry in `/etc/passwd`. Any file the user creates is, by default, **owned by this group**.
- **Secondary group**: Extra groups the user also belongs to, listed in `/etc/group`. These are used to grant **additional permissions** — for example, access to shared project folders — without changing what group owns the files the user personally creates.

## `userdel` — Deleting a User

| Command | What it does |
|---------|----------------|
| `userdel username` | Deletes the user account, but **keeps** their home directory and files |
| `userdel -r username` | Deletes the user account **and** removes their home directory and mail spool |
| `userdel -f username` | Force-deletes the user, even if they're currently logged in |

## `/etc/login.defs` — Default Rules for New Users

This file defines the **default policies** that `useradd` follows when it creates a new user — so you don't have to specify every setting manually each time. It includes things like:

- `UID_MIN` / `UID_MAX` — the allowed range for new user UIDs
- `GID_MIN` / `GID_MAX` — the allowed range for new group GIDs
- `PASS_MAX_DAYS` — default maximum password age
- `PASS_MIN_DAYS` — default minimum days before a password can be changed again
- `PASS_WARN_AGE` — how many days before expiry the user gets warned

## Inside `/etc/shadow`

`/etc/shadow` stores the **real, encrypted password** and all password-aging details. Unlike `/etc/passwd`, it can only be read by root. Each line has **9 fields**, separated by colons:

```
username:$6$saltvalue$hashvalue:19500:0:90:7:::
```

| Field # | Meaning |
|---------|---------|
| 1 | Username |
| 2 | Encrypted password hash (not a timezone!) |
| 3 | Date of last password change (days since Jan 1, 1970) |
| 4 | Minimum days required before the password can be changed again |
| 5 | Maximum days the password is valid before it must be changed |
| 6 | Warning period (days before expiry the user is warned) |
| 7 | Inactivity period after expiry, before the account is disabled |
| 8 | Account expiration date |
| 9 | Reserved (currently unused) |

**Note:** If you set a password at creation time using `useradd -p "password"`, it gets stored (encrypted) directly into this field in `/etc/shadow` — it isn't shown in plain text, but it is set immediately without the interactive prompt.

### Understanding the password hash pattern

The encrypted password normally looks like:

```
$6$randomsalt$longhashvalue
```

- The number right after the first `$` tells you **which hashing algorithm** was used:
  - `$1$` = MD5
  - `$5$` = SHA-256
  - `$6$` = SHA-512 (the current default on RHEL-based systems)
- The next part is the **salt** (random data added to make the hash unique, even for identical passwords).
- The final part is the actual **hash** of the password.

## `chage` — Managing Password Aging

`chage` lets you view or change all the password-aging settings for a user (the same fields stored in `/etc/shadow`).

| Command | What it does |
|---------|----------------|
| `chage -l username` | **List** current password aging info for the user |
| `chage -m 5 username` | Set **minimum** days before password can be changed again |
| `chage -M 90 username` | Set **maximum** days before the password must be changed |
| `chage -W 7 username` | Set the **warning** period (days before expiry) |
| `chage -I 10 username` | Set the **inactivity** period after expiry before the account locks |
| `chage -E 2026-12-31 username` | Set an **account expiration date** |
| `chage -d 2026-01-01 username` | Manually set the **last password change** date |
| `chage username` | Interactive mode — prompts you through setting all values one by one |

## Sudo — Giving Controlled Admin Access

### What is `sudo`?

`sudo` ("superuser do") lets an approved normal user run specific commands **as root** (or another user), without ever logging in as root directly. It uses the user's **own password** to confirm identity, and every command run through sudo is logged — which is much safer than sharing the root password.

### Normal User vs. Sudo User

- A **normal user** can only do things allowed for their own account — they can't install software, manage other users, or change system-wide settings.
- A **sudo user** is a normal user who has been given permission (through configuration) to run some or all administrative commands, by putting `sudo` in front of the command.

### Making a User a Sudo User (Quick Method)

On RHEL-based systems, the `wheel` group is already configured to have full sudo access. So the easiest way to grant sudo access is:

```bash
usermod -aG wheel username
```

(`-aG` **appends** the user to the group, without removing them from their other groups.)

### Creating a Custom Sudo Group

1. Create the group:
   ```bash
   groupadd sudogroup
   ```
2. Open the sudoers file **safely** using `visudo` (this checks for syntax errors before saving, so you don't accidentally lock yourself out of admin access):
   ```bash
   visudo
   ```
3. Add this line to grant the group full sudo access:
   ```
   %sudogroup ALL=(ALL) ALL
   ```
4. Add users to the group:
   ```bash
   usermod -aG sudogroup username
   ```

### Giving Access to Only Specific Commands

Instead of full access, you can restrict a user to just one or two commands:

```
username ALL=(ALL) /usr/bin/systemctl restart httpd
```

This user can now only run that exact `systemctl` command with `sudo`, and nothing else.

### Blocking Specific Commands While Allowing Everything Else

You can allow all commands **except** a specific one, using `!` in front of the command you want to block:

```
username ALL=(ALL) ALL, !/usr/bin/passwd
```

### Finding a Command's Full Path

Since sudoers rules need the **full path** to a command (not just its name), use:

```bash
which command_name
```
or
```bash
whereis command_name
```

For example, `which systemctl` might return `/usr/bin/systemctl`.

### Locking and Unlocking a User Account

| Command | What it does |
|---------|----------------|
| `passwd -l username` | **Lock** the account (disables password login) |
| `passwd -u username` | **Unlock** the account |
| `usermod -L username` | Same as `passwd -l` — locks the account |
| `usermod -U username` | Same as `passwd -u` — unlocks the account |

Behind the scenes, locking a user simply adds a `!` in front of their encrypted password in `/etc/shadow`, which makes the stored hash invalid for login. Unlocking removes that `!`.

## Quick Recap

> User and group management in Linux revolves around a handful of key files and commands: `useradd`/`groupadd` to create accounts, `/etc/skel` to auto-populate new home directories, `gpasswd` to manage group membership, `/etc/shadow` and `chage` to control password security and aging, and `sudo`/`visudo` to safely grant controlled administrative access without ever sharing the root password. Together, these form the core of everyday Linux user administration at the RHCSA level.
